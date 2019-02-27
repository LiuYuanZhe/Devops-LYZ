# Pods

在Kubernetes中，最小的管理元素不是一个个独立的容器，而是Pod,Pod是最小的，管理，创建，计划的最小单元.



 

## **什么是Pod**

一个**Pod**(就像一群鲸鱼，或者一个豌豆夹)相当于一个共享context的配置组，在同一个context下，应用可能还会有独立的cgroup隔离机制，一个Pod是一个容器环境下的“逻辑主机”，它可能包含一个或者多个紧密相连的应用，这些应用可能是在同一个物理主机或虚拟机上。

Pod 的context可以理解成多个linux命名空间的联合

- PID 命名空间（同一个Pod中应用可以看到其它进程）
- 网络 命名空间（同一个Pod的中的应用对相同的IP地址和端口有权限）
- IPC 命名空间（同一个Pod中的应用可以通过VPC或者POSIX进行通信）
- UTS 命名空间（同一个Pod中的应用共享一个主机名称）

同一个Pod中的应用可以共享磁盘，磁盘是Pod级的，应用可以通过文件系统调用，额外的，一个Pod可能会定义顶级的cgroup隔离，这样的话绑定到任何一个应用（好吧，这句是在没怎么看懂，就是说Pod，应用，隔离）

由于docker的架构，一个Pod是由多个相关的并且共享磁盘的容器组成，Pid的命名空间共享还没有应用到Docker中

和相互独立的容器一样，Pod是一种相对短暂的存在，而不是持久存在的，正如我们在Pod的生命周期中提到的，Pod被安排到结点上，并且保持在这个节点上直到被终止（根据重启的设定）或者被删除，当一个节点死掉之后，上面的所有Pod均会被删除。特殊的Pod永远不会被转移到的其他的节点，作为替代，他们必须被replace.

## **Pod的发展**

### **资源的共享及通信**

Pod使Pod内的数据共享及通信变得容易

Pod的中的应用均使用相同的网络命名空间及端口，并且可以通过localhost发现并沟通其他应用，每个Pod都有一个扁平化的网络命名空间下IP地址，它是Pod可以和其他的物理机及其他的容器进行无障碍通信，（The hostname is set to the pod’s Name for the application containers within the pod）主机名被设置为Pod的名称（这个没翻译出来…）

除了定义了在Pod中运行的应用之外，Pod还定义了一系列的共享的磁盘，磁盘让这些数据在容器重启的时候不回丢失并且可以将这些数据在Pod中的应用进行共享

### **管理**

Pod通过提供一个高层次抽象而不是底层的接口简化了应用的部署及管理，Pod 作为最小的部署及管理单位，位置管理，拷贝复制，资源共享，依赖关系都是自动处理的。（fate sharing估计就说什么时候该死了，什么时候该新增一个了…）

## **Pod的使用**

Pod可以作为垂直应用整合的载体，但是它的主要特点是支持同地协作，同地管理程序，例如：

- 内容管理系统，文件和数据加载，本地缓存等等
- 日志和检查点备份，压缩，循环，快照等等
- 数据交换监控，日志追踪，日志记录和监控适配器，以及事件发布等等
- 代理，网桥，适配器
- 控制，管理，配置，更新

总体来说，独立的Pod不会去加载多个相同的应用实例

## 考虑过的其他方案

**为什么不直接在一个容器上运行所有的应用？**

1. 透明，**Pod**中的容器对基础设施可见使的基础设施可以给容器提供服务，例如线程管理和资源监控，这为用户提供很多便利
2. 解耦软件依赖关系,独立的容器可以独立的进行重建和重新发布，Kubernetes 甚至会在将来支持独立容器的实时更新
3. 易用，用户不需要运行自己的线程管理器，也不需要关心程序的信号以及异常结束码等
4. 高效，因为基础设施承载了更多的责任，所以容器可以更加高效

**为什么不支持容器的协同调度**

容器的协同调度可以提供，但是它不具备Pod的大多数优点，比如资源共享，IPC，选择机制，简单管理等

## **Pod的持久性**

Pod并不是被设计成一个持久化的资源，它不会在调度失败，节点崩溃，或者其他回收中（比如因为资源的缺乏，或者其他的维护中）幸存下来

总体来说，用户因该直接去创建Pod，并且一直使用controller(replication controller),即使是一个节点的情况，这是因为controller提供了集群范围内的自我修复，以及复制还有展示管理

集群API的使用是用户的主要使用方式，这是相对普遍的在如下云管理平台中（ Borg, Marathon, Aurora, and Tupperware.）

**Pod的直接暴露是如下操作变得更容器**

1. 调度和管理的易用性
2. 在没有代理的情况下通过API可以对Pod进行操作
3. Pod的生命周期与管理器的生命周期的分离
4. 解偶控制器和服务，后段管理器仅仅监控Pod
5. 划分清楚了Kubelet级别的功能与云平台级别的功能，kubelet 实际上是一个Pod管理器
6. 高可用，当发生一些删除或者维护的过程时，Pod会自动的在他们被终止之前创建新的替代

目前对于宠物的最佳实践是，创建一个副本等于1和有对应service的一个replication控制器。如果你觉得这太麻烦，请在这里留下你的意见。

## 容器的终止

因为pod代表着一个集群中节点上运行的进程，让这些进程不再被需要，优雅的退出是很重要的（与粗暴的用一个KILL信号去结束，让应用没有机会进行清理操作）。用户应该能请求删除，并且在室进程终止的情况下能知道，而且也能保证删除最终完成。当一个用户请求删除pod，系统记录想要的优雅退出时间段，在这之前Pod不允许被强制的杀死，TERM信号会发送给容器主要的进程。一旦优雅退出的期限过了，KILL信号会送到这些进程，pod会从API服务器其中被删除。如果在等待进程结束的时候，Kubelet或者容器管理器重启了，结束的过程会带着完整的优雅退出时间段进行重试。

一个示例流程：

\1. 用户发送一个命令来删除Pod，默认的优雅退出时间是30秒

\2. API服务器中的Pod更新时间，超过该时间Pod被认为死亡

\3. 在客户端命令的的里面，Pod显示为”Terminating（退出中）”的状态

\4. （与第3同时）当Kubelet看到Pod标记为退出中的时候，因为第2步中时间已经设置了，它开始pod关闭的流程

i. 如果该Pod定义了一个停止前的钩子，其会在pod内部被调用。如果钩子在优雅退出时间段超时仍然在运行，第二步会意一个很小的优雅时间断被调用

ii. 进程被发送TERM的信号

\5. （与第三步同时进行）Pod从service的列表中被删除，不在被认为是运行着的pod的一部分。缓慢关闭的pod可以继续对外服务，当负载均衡器将他们轮流移除。

\6. 当优雅退出时间超时了，任何pod中正在运行的进程会被发送SIGKILL信号被杀死。

\7. Kubelet会完成pod的删除，将优雅退出的时间设置为0（表示立即删除）。pod从API中删除，不在对客户端可见。

默认情况下，所有的删除操作的优雅退出时间都在30秒以内。kubectl delete命令支持–graceperiod=的选项，以运行用户来修改默认值。0表示删除立即执行，并且立即从API中删除pod这样一个新的pod会在同时被创建。在节点上，被设置了立即结束的的pod，仍然会给一个很短的优雅退出时间段，才会开始被强制杀死。

## 使用Volume

Volume可以为容器提供持久化存储，比如

```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

更多挂载存储卷的方法参考[Volume](https://feisky.gitbooks.io/kubernetes/concepts/volume.html)。

## 私有镜像

在使用私有镜像时，需要创建一个docker registry secret，并在容器中引用。

创建docker registry secret：

```
kubectl create secret docker-registry regsecret --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

容器中引用该secret：

```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
    - name: private-reg-container
      image: <your-private-image>
  imagePullSecrets:
    - name: regsecret
```

## RestartPoliy

支持三种RestartPolicy

- Always：只要退出就重启
- OnFailure：失败退出（exit code不等于0）时重启
- Never：只要退出就不再重启

注意，这里的重启是指在Pod所在Node上面本地重启，并不会调度到其他Node上去。

## 环境变量

环境变量为容器提供了一些重要的资源，包括容器和Pod的基本信息以及集群中服务的信息等：

(1) hostname

`HOSTNAME`环境变量保存了该Pod的hostname。

（2）容器和Pod的基本信息

Pod的名字、命名空间、IP以及容器的计算资源限制等可以以Downward API的方式获取并存储到环境变量中。

```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "sh", "-c"]
      args:
      - env
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
        - name: MY_CPU_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.cpu
        - name: MY_CPU_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.cpu
        - name: MY_MEM_REQUEST
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: requests.memory
        - name: MY_MEM_LIMIT
          valueFrom:
            resourceFieldRef:
              containerName: test-container
              resource: limits.memory
  restartPolicy: Never
```

(3) 集群中服务的信息

容器的环境变量中还包括了容器运行前创建的所有服务的信息，比如默认的kubernetes服务对应了环境变量

```
KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
```

由于环境变量存在创建顺序的局限性（环境变量中不包含后来创建的服务），推荐使用DNS来解析服务。

## ImagePullPolicy

支持三种ImagePullPolicy

- Always：不管镜像是否存在都会进行一次拉取。
- Never：不管镜像是否存在都不会进行拉取
- IfNotPresent：只有镜像不存在时，才会进行镜像拉取。

注意：

- 默认为`IfNotPresent`，但`:latest`标签的镜像默认为`Always`。
- 拉取镜像时docker会进行校验，如果镜像中的MD5码没有变，则不会拉取镜像数据。
- 生产环境中应该尽量避免使用`:latest`标签，而开发环境中可以借助`:latest`标签自动拉取最新的镜像。

## 资源限制

Kubernetes通过cgroups限制容器的CPU和内存等计算资源，包括requests（请求，调度器保证调度到资源充足的Node上）和limits（上限）等：

- `spec.containers[].resources.limits.cpu`：CPU上限，可以短暂超过，容器也不会被停止
- `spec.containers[].resources.limits.memory`：内存上限，不可以超过；如果超过，容器可能会被停止或调度到其他资源充足的机器上
- `spec.containers[].resources.requests.cpu`：CPU请求，可以超过
- `spec.containers[].resources.requests.memory`：内存请求，可以超过；但如果超过，容器可能会在Node内存不足时清理

比如nginx容器请求30%的CPU和56MB的内存，但限制最多只用50%的CPU和128MB的内存：

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          cpu: "300m"
          memory: "56Mi"
        limits:
          cpu: "500m"
          memory: "128Mi"
```

注意，CPU的单位是milicpu，500mcpu=0.5cpu；而内存的单位则包括E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki等。

## 健康检查

为了确保容器在部署后确实处在正常运行状态，Kubernetes提供了两种探针（Probe，支持exec、tcp和httpGet方式）来探测容器的状态：

- LivenessProbe：探测应用是否处于健康状态，如果不健康则删除重建改容器
- ReadinessProbe：探测应用是否启动完成并且处于正常服务状态，如果不正常则更新容器的状态

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx
  name: nginx
spec:
    containers:
    - image: nginx
      imagePullPolicy: Always
      name: http
      livenessProbe:
        httpGet:
        path: /
        port: 80
        initialDelaySeconds: 15
        timeoutSeconds: 1
      readinessProbe:
        httpGet:
        path: /ping
        port: 80
        initialDelaySeconds: 5
        timeoutSeconds: 1
```

## Init Container

Init Container在所有容器运行之前执行（run-to-completion），常用来初始化配置。

```
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://kubernetes.io
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

## 容器生命周期钩子

容器生命周期钩子（Container Lifecycle Hooks）监听容器生命周期的特定事件，并在事件发生时执行已注册的回调函数。支持两种钩子：

- postStart： 容器启动后执行，注意由于是异步执行，它无法保证一定在ENTRYPOINT之后运行。如果失败，容器会被杀死，并根据RestartPolicy决定是否重启
- preStop：容器停止前执行，常用于资源清理。如果失败，容器同样也会被杀死

而钩子的回调函数支持两种方式：

- exec：在容器内执行命令
- httpGet：向指定URL发起GET请求

postStart和preStop钩子示例：

```
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:
        exec:
          command: ["/usr/sbin/nginx","-s","quit"]
```

## 指定Node

通过nodeSelector，一个Pod可以指定它所想要运行的Node节点。

首先给Node加上标签：

```
kubectl label nodes <your-node-name> disktype=ssd
```

接着，指定该Pod只想运行在带有`disktype=ssd`标签的Node上：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

## 使用Capabilities

默认情况下，容器都是以非特权容器的方式运行。比如，不能在容器中创建虚拟网卡、配置虚拟网络。

Kubernetes提供了修改[Capabilities](http://man7.org/linux/man-pages/man7/capabilities.7.html)的机制，可以按需要给给容器增加或删除。比如下面的配置给容器增加了`CAP_NET_ADMIN`并删除了`CAP_KILL`。

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: friendly-container
    image: "alpine:3.4"
    command: ["/bin/echo", "hello", "world"]
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
        drop:
        - KILL
```

## 限制网络带宽

可以通过给Pod增加`kubernetes.io/ingress-bandwidth`和`kubernetes.io/egress-bandwidth`这两个annotation来限制Pod的网络带宽

```
apiVersion: v1
kind: Pod
metadata:
  name: qos
  annotations:
    kubernetes.io/ingress-bandwidth: 3M
    kubernetes.io/egress-bandwidth: 4M
spec:
  containers:
  - name: iperf3
    image: networkstatic/iperf3
    command:
    - iperf3
    - -s
```

>  **仅kubenet支持限制带宽**
>
> 目前只有kubenet网络插件支持限制网络带宽，其他CNI网络插件暂不支持这个功能。

kubenet的网络带宽限制其实是通过tc来实现的

```
# setup qdisc (only once)
tc qdisc add dev cbr0 root handle 1: htb default 30
# download rate
tc class add dev cbr0 parent 1: classid 1:2 htb rate 3Mbit
tc filter add dev cbr0 protocol ip parent 1:0 prio 1 u32 match ip dst 10.1.0.3/32 flowid 1:2
# upload rate
tc class add dev cbr0 parent 1: classid 1:3 htb rate 4Mbit
tc filter add dev cbr0 protocol ip parent 1:0 prio 1 u32 match ip src 10.1.0.3/32 flowid 1:3
```

## 调度到指定的Node上

可以通过nodeSelector、nodeAffinity、podAffinity以及Taints和tolerations等来将Pod调度到需要的Node上。具体使用方法请参考调度器章节。