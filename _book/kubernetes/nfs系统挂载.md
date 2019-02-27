安装（centos7.x）：

```shell
yum install rpcbind	
yum install nfs-utils
```

nfs系统配置目录：

在/etc/exports

```shell
vim /etc/exports	
# 加载文件
exportfs -r
```

每一个文件夹一行

目录  ip(参数)

```shell
/date 10.10.70.*(rw，sync)
```





| 参数             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| ro               | 只读访问                                                     |
| rw               | 读写访问                                                     |
| sync             | 所有数据在请求时写入共享                                     |
| async            | nfs在写入数据前可以响应请求                                  |
| secure           | nfs通过1024以下的安全TCP/IP端口发送                          |
| insecure         | nfs通过1024以上的端口发送                                    |
| wdelay           | 如果多个用户要写入nfs目录，则归组写入（默认）                |
| no_wdelay        | 如果多个用户要写入nfs目录，则立即写入，当使用async时，无需此设置 |
| hide             | 在nfs共享目录中不共享其子目录                                |
| no_hide          | 共享nfs目录的子目录                                          |
| subtree_check    | 如果共享/usr/bin之类的子目录时，强制nfs检查父目录的权限（默认） |
| no_subtree_check | 不检查父目录权限                                             |
| all_squash       | 共享文件的UID和GID映射匿名用户anonymous，适合公用目录        |
| no_all_squash    | 保留共享文件的UID和GID（默认）                               |
| root_squash      | root用户的所有请求映射成如anonymous用户一样的权限（默认）    |
| no_root_squash   | root用户具有根目录的完全管理访问权限                         |
| anonuid=xxx      | 指定nfs服务器/etc/passwd文件中匿名用户的UID                  |
| anongid=xxx      | 指定nfs服务器/etc/passwd文件中匿名用户的GID                  |



服务端开启服务

```shell
systemctl start nfs
systemctl start rpcbind	
```

查看绑定端口

```shell
rpcinfo -e
```

![](/Users/rqw1991/Devops-LYZ/kubernetes/img/tcpinfo查看绑定端口.jpeg)



客户端(ip允许网段)可以远程挂载服务端的目录：

```shell
#挂载
mount 10.10.70.211:/mountData /mountData
#卸载
umount /mountData
#查看
ll
# 服务端测试
echo "test server data write" > /mountData/test
#客户端查看
ll /mountData
```

可以结合kubernetes的pv与pvc使用或者挂载pod的volume使用了

![](/Users/rqw1991/Devops-LYZ/kubernetes/img/挂载目录服务端.jpeg)

![](/Users/rqw1991/Devops-LYZ/kubernetes/img/挂载目录客户端.jpeg)