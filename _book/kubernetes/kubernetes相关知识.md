kubernetes节点调度：

隔离节点，使节点不能在分配pod

kubectl cordon node

解除隔离节点

kubectl uncordon node

一般用来保护master节点，防止master节点被pod压垮。





集群对外暴露服务的方式：

1.nodeport 

2.loadbalance

3.ingress（nginx，traefik）





kubernetes资源类型：

namespace：命名空间，做隔离方案

deployment：replicaset，pod控制器，用于描述rs，pod资源，完成扩容更新等。

replicases/replicacontroller：pod控制器，用于描述pod资源。

pod：kubernetes最小调度但单元，docker／rkt等容器运行的容器组（内中可能有一个或多个docker容器）。

daemonset：节点上的pod控制器。

configmap：配置键值对，pod运行的配置。