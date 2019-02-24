## kubernetes 部署项目

kubernetes部署项目最终以pod的方式运行在node节点，每一个pod可以包含一个或多个镜像。pod又被replica set（副本集）控制，replica set又被deployment控制（在spinnaker中创建的application就是replica set）。replica set是replica controller的升级版本，现在主要以rs为主。

##### 比较 Deployment 与 ReplicaSet

ReplicaSet 也是用来管理多个 Pod 的副本，那么 Deployment 和 ReplicaSet 的区别在哪里呢？

当我们创建了 Deployment 之后，实际上也创建了 ReplicaSet，所以说 Deployment 管理着 ReplicaSet（实际上 Deployment 比 ReplicaSet 有着更多功能）。



可以使用deploy来进行扩容

具体命令：

```shell
# 获取deploy名称
kubectl get deploy -n namespace
# 扩容
kubectl scale deploy -n namespace --replicas=n	
```

![](./img/使用scale进行扩容.jpeg '')

可以利用replica set或者deploy进行滚动升级。



部署一个可视化平台（visplat）为例

首先进行yaml的准备，可以分开写也可以写在一起。

明确目的：部署可访问的pod，有对外接口提供对容器的访问，并且在启动的时候替换部分变量。

1.准备deploy或者replica set的部署yaml文件（这里是deploy），用于部署最终生成pod。

2.准备configmap的yaml部署文件，在pod启动中容器启动时替换事先写入的环境变量，spinnaker的变量就是这么实现的。

3.准备service的yaml部署文件，在pod启动成功后创建nodeport进行容器内端口的访问，spinnaker也是这样提供的，spinnaker还创建了loadblance服务。

首先要先创建相应的命名空间

namespace-test.yaml

```shell
apiVersion: extensions/v1beta1
kind: namespace
metadata:
  name:test
  label:
    name:xxxx
```



```shell
kubectl creater -f namespace-test.yaml
```





deploy部署文件

```shell
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: pubinfo-visplat-instance1
  namespace: test
  labels:
    pubinfo-visplat-compent: pubinfo-visplat-instance1
spec:
  replicas: 1 
  selector:
    matchLabels:
      pubinfo-visplat-compent: pubinfo-visplat-instance1
  template:
    metadata:
      labels:
        pubinfo-visplat-compent: pubinfo-visplat-instance1
    spec:
      #hostNetwork: true
      containers:
      - name: pubinfo-instance1-visplat
      #启动pod要拉取的镜像
        image: "10.10.70.65/pubinfo/visplat:1-821063867a9a275439006b8a0d9b64ec6015856f"
        #镜像拉取策略，这个是一直拉取，ifnotpresent是如果没有才拉取
        imagePullPolicy: Always
        ports:
        #容器的端口
        - containerPort: 8084
        #分配的pod的资源，requests是初始，limit是最大使用资源
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1048Mi"
            cpu: "500m"
        #表明了要使用configmap
        env:
        - name: SERVICE_HOME
          valueFrom:
            configMapKeyRef:
              name: pubinfo-visplat-instance1
              key: SERVICE_HOME
        - name: SERVICE_SECURITY
          valueFrom:
            configMapKeyRef:
              name: pubinfo-visplat-instance1
              key: SERVICE_SECURITY    
      # 挂载时区盘，保证和容器的时间一致
      volumes:
       - name: localtime
         hostPath:
           path: /etc/localtime 
```



configmap(optional是可选性，不可选如果未读到变量会阻止pod启动)

```shell
apiVersion: v1
kind: ConfigMap
metadata:
  name: pubinfo-visplat-instance1
  namespace: test
data:
  SERVICE_HOME: http://10.10.70.211:38084
  SERVICE_SECURITY: http://10.10.70.211:38084/visplat-app/login/cas
```



service

```shell
apiVersion: v1
kind: Service
metadata:
  name: pubinfo-visplat-instance1-svc
  namespace: test
  labels:
    pubinfo-visplat-compent: pubinfo-visplat-instance1
spec:
  selector:
    pubinfo-visplat-compent: pubinfo-visplat-instance1
  # nodeport 外部可以使用nodeip：port访问，clusterip只能集群内部访问
  type: NodePort
  ports:
  #clusterip端口 容器端口 传输协议 对外发布的端口
  - port: 8084
    targetPort: 8084
    protocol: TCP
    nodePort: 38084
```





将这三个文件放在统一文件夹下，在kubenetes节点使用命令`kubectl create -f .`,可以看到deployment service was created输出，然后使用`kubectl get pod -n namespace`。





yaml文件：

[简化 Kubernetes Yaml 文件创建](https://yq.aliyun.com/articles/341213)由于Yaml文件格式比较复杂，即使是老司机有时也不免会犯错或需要查询文档，因此可以dry-run 一下，`kubectl run myapp --image=nginx --dry-run -o yaml`会输出模拟运行 nginx 镜像的yaml 文件内容，copy-paste 即可。或者你可以`kubectl get deployment my-nginx -o yaml `查看一个已有 kubernetes object 的配置，依葫芦画瓢。