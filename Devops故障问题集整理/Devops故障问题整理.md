# 泰华DevOps故障问题解决方案记录

 ## Jenkins异常
 ### jenkins out of memory

 内存溢出，查看相关节点，在jenkins部署时，首先需要创建相关pod，分配指定的内存与cpu。linux内存机制是如果可利用内存不足需求分配的内存大小则pod创建不成功，导致本次jenkin构建失败。
 查看内存：  
`free -g`


 前端无法访问数据库或单点登录

 目前策略在外网只能访问58，所有相关跳转页面都需要在58的nginx上做映射，然后由58转发到目标机器进行请求。首先在58进行访问，出现500-504异常，应该是nginx没有代理上，访问61或62，如果可以访问到，应用应该部署成功，但是需要配置nginx；如果应用没有访问到，在kubernetes主节点上查找相应的pod，查看对应的应用日志定位问题。

 在镜像进行构建时，应该在每一个步骤进行日志埋点，方便进行错误定位。

 

### spinnaker节点出现connection refused，节点不断重启

 重启该节点机器，如果没有恢复就重启docker服务。

`reboot`

`systemctl restart docker`



 ## kubernetes节点故障

 ### node notready
 节点网络故障，重启该节点机器，有时候会有重启机器后仍然不能恢复的状态，或机器在reboot命令后不能登录，但是能ping通的情况，这是该机器在重启过程中被某进程卡住阻塞关机进程，这时如果是虚拟机的话可以登录vmware控制台直接拔掉电源级别重启，此时如果恢复即可。







## pod故障

 ### pod异常，前端不能访问
 pod中的镜像运行异常，可以删除pod，kubernetes会自动出发重新分配pod重新运行镜像。此方法也可以出发重新将该pod调度到其他节点。

 ### pod pending
 pod在等待状态，可以在k8s控制台使用describe或者log进行故障排查，看是因为镜像启动出错还是因为节点故障。

 ### crashloopbackoff
 kubernetes正在尽力启动这个项目，但是这个pod不能正常启动，一般是由于pod中的项目不能启动，此时不能进入长时间pod查看，因为失败后会重启，只能查看日志，请尽量在任务的启动脚本中使用echo进行日志埋点输出


## docker image push error
`received unexpected HTTP status: 500 Internal Server Error`
 在jenkins的最后一个步骤中无法推送打好的镜像到dockerhub。docker错误，镜像无法推送到镜像仓库中，登录到相关镜像仓库的机器上查看docker进程信息，进入docker镜像中使用df -Th命令查看挂卷，挂载盘达到百分之百，docker容器挂载镜像达到百分之百时docker进程假死，需要进行挂载卷扩容。

 清除镜像后只是清除了该镜像的并且标记，然后进入docker容器内（因为registry使用docker镜像构建），执行docker的垃圾回收命令。相关命令如下，
`docker ps`
`docker exec -it 'docker-id' sh或者/bin/bash`
 注意命令根据相应的docker镜像而使用，有的docker镜像的基础镜像中没有bash命令行。
`registry garbage-collect /etc/docker/registry/config.yml`



 