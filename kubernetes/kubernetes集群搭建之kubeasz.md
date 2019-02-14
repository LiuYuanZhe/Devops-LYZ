### kubeasz安装

##### 基础系统配置

- 推荐内存2G/硬盘30G以上
- 最小化安装`Ubuntu 16.04 server`或者`CentOS 7 Minimal`
- 配置基础网络、更新源、SSH登陆等



从github上下载项目并将项目中内容移动到etc目录下

```shell
mkdir -p /etc/ansible
mv kubeasz/* /etc/ansible
```



github地址：https://github.com/gjmzj/kubeasz

##### 部署方案选择

- [单节点](https://github.com/gjmzj/kubeasz/blob/master/example/hosts.allinone.example)
- [单主多节点](https://github.com/gjmzj/kubeasz/blob/master/example/hosts.s-master.example)
- [多主多节点](https://github.com/gjmzj/kubeasz/blob/master/example/hosts.m-masters.example)
- [在公有云上部署](https://github.com/gjmzj/kubeasz/blob/master/example/hosts.cloud.example)

一般使用多主节点保证高可用，但此节点部署复杂，需要额外规划一个master vip（虚地址），此地址为同网段中空闲地址，etcd节点一般部署为奇数个。

多主节点还包含两个lb节点部署haproxy和keepalived，请根据需要选择此方案，关系较为复杂。

git项目中提供了basic和extra两个镜像包，可以保证离线安装。

将两个镜像包解压后的文件放入/etc/ansible/bin文件夹下。

使用ansible命令进行节点通信测试

```
ansible all -m ping
```

根据安装方案需求修改项目下的host文件，样例在example文件夹下。

#### 注意：host文件中的节点必须写为ip形式，不可以写别名（nodex），本项目不支持

执行脚本安装：

```shell
# 分步安装
ansible-playbook 01.prepare.yml
ansible-playbook 02.etcd.yml
ansible-playbook 03.docker.yml
ansible-playbook 04.kube-master.yml
ansible-playbook 05.kube-node.yml
ansible-playbook 06.network.yml
ansible-playbook 07.cluster-addon.yml
# 一步安装
#ansible-playbook 90.setup.yml
# 卸载
ansible-playbook 99.clean.yml
```



如果中途失败建议清理etc下kubernetes或etcd文件夹下的证书文件，使用journalctl -xe结合错误日志进行排查。





##### 新增node节点

新增`kube-node`节点大致流程为：

- 新节点预处理 prepare
- 新节点安装 docker 服务
- 新节点安装 kube-node 服务
- 新节点安装网络插件相关

修改host文件中的new node组，添加node

​	执行安装脚本

```
$ ansible-playbook /etc/ansible/20.addnode.yml
```

使用命令查看新增节点

```shell
kubectl get no	
```

成功后将host文件中new node组下的节点放入node中



##### 增加master节点

注意：目前仅支持按照本项目`多主模式`(hosts.m-masters.example)部署的`k8s`集群增加`master`节点

新增`kube-master`节点大致流程为：

- LB节点重新配置 haproxy并重启 haproxy服务
- 新节点预处理 prepare
- 新节点安装 docker 服务
- 新节点安装 kube-master 服务
- 新节点安装 kube-node 服务
- 新节点安装网络插件相关
- 禁止业务 pod调度到新master节点

###### 操作步骤

按照本项目说明，首先确保deploy节点能够ssh免密码登陆新增节点，然后在**deploy**节点执行两步：

- 修改ansible hosts 文件，在 [new-master] 组添加新增的节点，举例如下：

```
...
[new-master]
192.168.1.5                 	# 新增 master节点
```

- 执行安装脚本

```
$ ansible-playbook /etc/ansible/21.addmaster.yml
```

### 验证

```
# 在新节点master 服务状态
$ systemctl status kube-apiserver 
$ systemctl status kube-controller-manager
$ systemctl status kube-scheduler

# 查看新master的服务日志
$ journalctl -u kube-apiserver -f

# 查看集群节点，可以看到新 master节点 Ready, 并且禁止了POD 调度功能
$ kubectl get no
```



上述步骤验证成功，确认新节点工作正常后，为了方便后续再次添加节点，在ansible hosts文件中，把 [new-master] 组下的节点全部复制到 [kube-master] 组下，并清空 [new-master] 组中的节点。