# 泰华DevOps故障问题解决方案记录

 ## Jenkins异常
 问题：jenkins构建时出现jenkins out of memory。

 原因：jenkins分配到的node的内存不足pod的request内存。

 解决方案：内存溢出，查看相关节点，在jenkins部署时，首先需要创建相关pod，分配指定的内存与cpu。linux内存机制是如果可利用内存不足需求分配的内存大小则pod创建不成功，导致本次jenkin构建失败。
 查看内存：  
`free -g`




 问题：前端无法访问数据库或单点登录

 原因：目前在办公网只能访问58网段

 解决方案：所有相关跳转页面都需要在58的nginx上做映射，然后由58转发到目标机器进行请求。首先在58进行访问，出现500-504异常，应该是nginx没有代理上，访问61或62，如果可以访问到，应用应该部署成功，但是需要配置nginx；如果应用没有访问到，在kubernetes主节点上查找相应的pod，查看对应的应用日志定位问题。

 注意：在镜像进行构建时，应该在每一个步骤进行日志埋点，方便进行错误定位。

 

### spinnaker节点出现connection refused，节点不断重启

问题:spinnaker组件无法通信，组件不断重启

解决方案: 重启该节点机器，如果没有恢复就重启docker服务。

`reboot`

`systemctl restart docker`



 ## kubernetes node

 问题：kubectl get no/node出现节点not ready现象

 解决方案：节点网络故障，首先重启docker服务，重启该节点机器，有时候会有重启机器后仍然不能恢复的状态，或机器在reboot命令后不能登录，但是能ping通的情况，这是该机器在重启过程中被某进程卡住阻塞关机进程，这时如果是虚拟机的话可以登录vmware控制台直接拔掉电源级别重启，此时如果恢复即可。







## kubernetes pod

 问题：pod异常，前端不能访问

 解决方案：pod中的镜像运行异常

 解决方案：可以删除pod，kubernetes会自动出发重新分配pod重新运行镜像。此方法也可以出发重新将该pod调度到其他节点。



 问题：pod pending

 原因：pod在等待状态

 解决方案：可以在k8s控制台使用describe或者log进行故障排查，看是因为镜像启动出错还是因为节点故障。

相关命令：

```shell
kubectl describe podname -n namespace
kubectl log -f podname -n namespace
```



 问题：pod crashloopbackoff

 原因：kubernetes正在尽力启动这个项目，但是这个pod不能正常启动，一般是由于pod中的项目不能启动，此时不能进入长时间pod查看，因为失败后会重启

 解决方案：只能查看日志，请尽量在任务的启动脚本中使用echo进行日志埋点输出，并且分析问题实在启动容器时还是环境原因，建议在docker环境下运行排除配置中心原因



问题：pod莫名重启

定位问题：先从主节点使用命令进行排查，使用`kubectl get pod -n namespace -o wide`定位该pod位于那个node上,使用`kubectl describe pod-n namespace `，定位问题的具体原因，ssh到相应node机器，使用`journalctl -u kubelet`进行故障的查看。

原因：网络插件cni与docker接口冲突，无法找到命名空间，终结pod。

https://www.colabug.com/3973167.html

搜索：Flannel and docker Interface Conflict



问题：日志与机器时间不一致

原因：

- 容器内事件与当前node节点事件不一致
- 容器内的应用实践与当前容器时间不一致 

一般这种情况都是差八个小时，时区不对，kubernetes的节点时间是同步的，使用chrony同步。

解决方法

- 把当前容器的时间挂载机器的时间，挂载位置：/etc/localtime

- 在java启动时增加参数例如命令：

  ```java
  java -jar -Duser.timeone=GMT+8 jarpackagename.jar
  ```

  




## docker
 问题：docker镜像推送失败

`received unexpected HTTP status: 500 Internal Server Error`
 原因：在jenkins的最后一个步骤中无法推送打好的镜像到dockerhub。docker错误，镜像无法推送到镜像仓库中，登录到相关镜像仓库的机器上查看docker进程信息，进入docker镜像中使用df -Th命令查看挂卷，挂载盘达到百分之百，docker容器挂载镜像达到百分之百时docker进程假死，需要进行挂载卷扩容。

 解决方案：清除镜像后只是清除了该镜像的并且标记，然后进入docker容器内（因为registry使用docker镜像构建），执行docker的垃圾回收命令。相关命令如下，
`docker ps`
`docker exec -it 'docker-id' sh或者/bin/bash`
 注意命令根据相应的docker镜像而使用，有的docker镜像的基础镜像中没有bash命令行。
`registry garbage-collect /etc/docker/registry/config.yml`



 