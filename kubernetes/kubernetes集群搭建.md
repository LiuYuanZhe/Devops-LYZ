kubernetes环境搭建

1.准备linux环境，准备节点环境。

2.准备docker镜像。

3.准备kubespray项目，编写相关ansible

集群架构图

![](/Users/rqw1991/Devops-LYZ/kubernetes/img/kubernetes测试集群.jpg)

#### kubernetes主要组件：

**kube-controller-manager** 是集群内部的管理控制中心，负责集群内的Node、Pod副本、服务端点（Endpoint）、命名空间（Namespace）、服务账号（ServiceAccount）、资源定额（ResourceQuota）的管理，当某个Node意外宕机时，Controller Manager会及时发现并执行自动化修复流程，确保集群始终处于预期的工作状态。 

**kube-scheduler** 集群的调度器，负责根据特定的调度算法将pod调度到指定的工作节点上。调度器需要考虑节点和集体的资源要求、服务质量要求、硬件/软件/政策约束、亲和力和反亲和力规范、数据局部性、负载间干扰、完成期限等。

**kube-apiserver** 是集群的管理者，负责对外提供接口（restful），配置资源对象，提供功能组件管理集群等功能。

**kubelet** 是集群每个节点最重要的组件，负责管理和维护kubernetes集群上运行的容器，主要是保证集群上的pod的状态与目标状态一致。

**kube-proxy/coredns** 是集群的服务发现与反向代理的组件，保证了集群中的网络通信，监控了集群中service和endpoint的动态变化，主要保证了用户可以通过ip和端口号访问到后端的pod。

**etcd** 是kv数据库，用于配制共享和服务发现，保证了kubernetes中的数据一致性。较于zk操作简单。

**nfs** 网络文件系统，共享挂载的存储目录。

#### ha方案相关：

##### Haproxy简介(负载均衡)

HAProxy 提供高可用性、负载均衡以及基于TCP和HTTP应用的代理,支持虚拟主机,它是开源、快速并且可靠的一种解决方案。HAProxy 特别适用于那些负载特大的 web 站点, 这些站点通常又需要会话保持或七层处理（和Nginx比较有优势的地方）。HAProxy 运行在当前的硬件上,完全可以支持数以万计的并发连接。并且它的运行模式使得它可以很简单安全的整 合进您当前的架构中, 同时可以保护你的 web 服务器不被暴露到网络上。

 

##### Keepalived简介(故障转移)

Keepalived是基于vrrp协议的一款高可用软件，它是作用在主机上，而不是路由器上。Keepailived把多台主机虚拟在一起，提供一个虚拟IP对外提供服务，它拥有一台master服务器和多台backup服务器，当主服务器出现故障时，虚拟IP地址会自动漂移到备份服务器，实现故障转移的高可用可能，即双机热备。注意：服务器的时间一定要一致。

VRRP（Virtual Router Redundancy Protocol）协议，即虚拟器路由冗余协议，是为了解决局域网内默认网关单点失效的问题。

VRRP 将局域网内的一组路由器组成一个虚拟路由器组,每个路由器都有自己的局域网地址, 根据设置的优先级最高决定那个是master路由器。然后网关地址赋给该主路由器, 该主路由器定时发送VRRP报文向虚拟路由器组公布健康状况, 备份的路由器根据柏爱文判断Master路由器是否工作正常,从而决定是否要接替它. VRRP说白了就是实现IP地址漂移的，是一种容错协议。在下图中，Router A(10.100.10.1)、Router B(10.100.10.2)和Router C(10.100.10.3) 组成一个虚拟路由器。各虚拟路由器都有自己的IP地址。局域网内的主机将虚拟路由器设置为缺省网关。 Router A、Router B和Router C中优先级最高的那台路由器作为Master路由器,比如A，承担网关的功能。局域网内的服务 只知道这台主master路由器A的存在,将自己缺省路由下一跳地址设置为该路由的ip地址10.100.10.1, 其余两台路由器作为Backup路由器。当master路由器出故障后， backup路由器会根据优先级重新选举出新的master路由器承担网关功能。Master路由器周期性地发送VRRP报文， 在虚拟路由器中公布其配置信息（优先级等）和工作状况。Backup路由器通过接收到VRRP报文的情况来判断Master路由器是否工作工常。

![](/Users/rqw1991/Devops-LYZ/kubernetes/img/vrrp图片.png)

 

 

 

 

 