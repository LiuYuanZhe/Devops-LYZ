spinnaker搭建

spinnaker工具基于`helm`( helm在kubernetes中有相关说明 )安装，需要依赖`redis` , `cassandra`两种数据库。

## 在线环境下:

首先先在helm中下载spinnaker的部署文件：

`curl -Lo values.yaml https://raw.githubusercontent.com/kubernetes/charts/master/stable/spinnaker/values.yaml`

运行命令安装

`$ helm install -n my-spinnaker stable/spinnaker -f values.yaml --timeout 3600  --version 0.3.5 --namespace spinnaker`



## 离线环境下（重要）：

从仓库中拉取下容器云平台下的项目，在`kubernetes/cicd/spinnaker`下有三个文件夹，分别是`redis`,`cassandra`,`spinnaker`，前两个是spinnaker的数据库，后一个是`spinnaker`的客户端。

### `redis`使用`kuberntes`命令启动：

```shell
kubectl create -f .	
```



### `cassandra`数据库使用docker启动：

`docker run  --restart=always --name spinnaker-cassandra --net=host --env CASSANDRA_START_RPC=true -v /data/spinnaker/cassandra/datadir:/var/lib/cassandra -d 10.10.70.65/library/cassandra:latest `



### `spinnaker` 使用`helm`启动：

`helm install --namespace cicd --name spinnaker2 ./spinnaker/ `

建立`secret`使`spinnaker`可以关联到`kubernetes`集群：

`kubectl create secret generic --from-file=config my-kubeconfig -n cicd`



然后即可在页面中访问spinnaker的页面，在kubernetes中搭建的spinnaker默认没有密码，可以直接对接集群。