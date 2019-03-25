#### ansible介绍

ansible是一款自动化运维工具，支持多节点发布、远程任务执行。适合管理多节点的安装部署软件环境，只需要在运行节点安装本工具，被管理节点不需要安装，只需要配置节点互信即可，ansible基于python开发。

ansible把命令抽象成剧本，通过事先写好的剧本进行在管理节点软件的分发安装，并且检查相应安装的状态是否成功。使用ansible可以轻松的在数十或成百上千个配置了节点互信的机器上部署需要的集群。

现在主流的`kubernetes`集群方案大多数由`ansible`项目进行部署，用户只需要在`github`中`clone`项目到本地，在准备相应的镜像包或者配置可达的镜像仓库，在`ansible`项目中对hosts文件进行配置，再执行`ansible`的命令进行安装。

以下是两个基于ansible搭建kubernetes项目：

https://github.com/gjmzj/kubeasz

https://github.com/kubernetes-sigs/kubespray