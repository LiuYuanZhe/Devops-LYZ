# kubernetes 部署节点准备（centos系统）

修改hosts文件，添加想要部署的集群机器

kubespray可以起别名，使用nodeX来识别机器，kubeasz只能使用ip标示主机来安装，在ansible的主机列表中要做区分

部署时只在主节点上执行脚本，由ansible进行下发二进制文件等命令，所以要配置节点互信，部署节点到集群节点的ssh免密登录。



设置yum仓库

kubespray安装

k8s.repo文件：

```shell
[k8s-repo]  
name=k8s main Repository  
baseurl=http://10.10.70.65:9090/repo/  
enabled=1  
gpgcheck=0  
```



kubeasz安装

配置epel镜像源，之前最好把阿里的也添加一下

```shell
# 文档中脚本默认均以root用户执行
# 安装 epel 源并更新
yum install epel-release -y
yum update
# update可以换成upgrade update升级系统内核，upgrade只升级软件内核
# 安装python
yum install python -y
```



在deploy节点安装ansible工具

```
yum install git python-pip -y
# pip安装ansible(国内如果安装太慢可以直接用pip阿里云加速)
#pip install pip --upgrade
#pip install ansible
pip install pip --upgrade -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
pip install --no-cache-dir ansible -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```



### 所有节点都需配置

###### 开启ip转发：

编辑文件/usr/lib/sysctl.d/50-default.conf

增加net.ipv4.ip_forward = 1 

执行sysctl -p



###### 关闭防火墙与selinux：

```shell
systemctl stop firewalld  
systemctl disable firewalld  
```



###### 关闭swap

```shell
sudo swapoff -a
# 注释掉/etc/fstab 中的swap
```



###### 关闭selinux

```shell
SELINUX=permissive
SELINUXTYPE=targeted
```



安装python，ansible相关

```
yum install python-pip ansible python-netaddr   python-pip python-devel  
```

如果使用kubespray安装，需要升级jinja，在断网的情况下准备好的文件夹下有下载好的python本地包，安装jinja需要依赖markupsafe，使用pip命令安装markupsafe包，在安装jinja2.9的whl包，否则在执行脚本的时候会报错误：

```shell


{"failed": true, "msg": "The conditional check '{%- set certs = {'sync': False} -%}\n{% if gen_node_certs[inventory_hostname] or\n  (not etcdcert_node.results[0].stat.exists|default(False)) or\n    (not etcdcert_node.results[1].stat.exists|default(False)) or\n      (etcdcert_node.results[1].stat.checksum|default('') != etcdcert_master.files|selectattr("path", "equalto", etcdcert_node.results[1].stat.path)|map(attribute="checksum")|first|default('')) -%}\n        {%- set _ = certs.update({'sync': True}) -%}\n{% endif %}\n{{ certs.sync }}' failed. The error was: no test named 'equalto'\n\nThe error appears to have been in '/root/kubespray/roles/etcd/tasks/check_certs.yml': line 57, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: "Check_certs | Set 'sync_certs' to true"\n  ^ here\n"}

```



