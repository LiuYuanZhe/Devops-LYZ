##  harbor简介

harbor是vmware开源的一款企业级docker registry，通过添加一些企业必需的功能特性，例如安全、标识和管理等，扩展了开源Docker Distribution。作为一个企业级私有Registry服务器，Harbor提供了更好的性能和安全。提升用户使用Registry构建和运行环境传输镜像的效率。Harbor支持安装在多个Registry节点的镜像资源复制，镜像全部保存在私有Registry中， 确保数据和知识产权在公司内部网络中管控。另外，Harbor也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等。

##  harbor的特性

- **基于角色的访问控制** ：用户与Docker镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一项目（project）里有不同的权限。
- **镜像复制** ： 镜像可以在多个Registry实例中复制（同步）。尤其适合于负载均衡，高可用，混合云和多云的场景。
- **图形化用户界面** ： 用户可以通过浏览器来浏览，检索当前Docker镜像仓库，管理项目和命名空间。
- **AD/LDAP 支持** ： Harbor可以集成企业内部已有的AD/LDAP，用于鉴权认证管理。
- **审计管理** ： 所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。
- **国际化** ： 已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来。
- **RESTful API** ： RESTful API 提供给管理员对于Harbor更多的操控, 使得与其它管理软件集成变得更容易。
- **部署简单** ： 提供在线和离线两种安装工具， 也可以安装到vSphere平台(OVA方式)虚拟设备。

 

# Harbor的安装与部署

环境：centos7.4

因为在内网中，所以统一使用离线包进行安装。

harbor需要依赖docker-compose来启动。

研发人员需要准备两个包：docker-compose和docker-harbor。

### docker-compose的安装

docker-compose是一个docker轻量级的编排(@1)工具，只需要下载它的运行文件。

地址：`https://github.com/docker/compose/releases`(因为在github中，请提前下载并且选择对应版本)

下载完成后将其拷入命令集中。

`cp docker-compose /usr/local/bin/docker-compose`

授予执行权限

`chmod +x /usr/local/bin/docker-compose`

ps：网上的命令讲解

`curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m`-o /usr/local/bin/docker-compose`

从github中下载 -s 减少输出信息，-o 输出，-L跟随链接重定向

可选：`yum install bash-completion` bash命令补全

`docker-compose --version` 

安装成功～

卸载：`rm  /usr/local/bin/docker-compose`

 

## Harbor安装

准备好offline安装包,开始安装

```
# cd /usr/local/src/
# tar zxf harbor-online-installer-v1.2.0.tgz  -C /usr/local/
# cd /usr/local/harbor/
```

修改harbor目录下的harbor.cfg文件

```
# vim /usr/local/harbor/harbor.cfg
hostname = 
#邮箱配置 按照个人需求配置
email_server = 
email_server_port = 
email_username = 
email_password =
email_from = UnixFBI <>
email_ssl = false
#禁止用户注册
self_registration = off
#设置只有管理员可以创建项目
project_creation_restriction = adminonly
```

 

执行安装脚本

`/usr/local/harbor/install.sh`

开始会加载镜像，由于镜像包都已经离线下载，只需要等待即可。

安装完成后执行命令`docker ps`就可以看到启动的harbor镜像，镜像是使用docker-compose部署，相关信息可以查看目录下的docker-compose.yml文件。

![img](./harbor使用img/harbor安装1.jpeg)

![img](./harbor使用img/harbor安装2.jpeg)

这时项目已经启动

 

harbor的日常启动关闭命令：

```
启动Harbor
# docker-compose start
停止Harbor
# docker-comose stop
重启Harbor
# docker-compose restart
```

 

 使用docker-compose管理。

这时可以访问harbor页面来查看。

浏览器输入部署地址，例：10.10.70.214 。

![img](./harbor使用img/harbor登录.jpeg)

测试账号:test02/Ll23456

 

附加：如果需要https进行访问

需要安装openssl服务。

修改harbor.cfg的默认访问方式

```
hostname = rgs.unixfbi.com
ui_url_protocol = https
ssl_cert = /etc/certs/ca.crt
ssl_cert_key = /etc/certs/ca.key
```

创建自签名证书：

```
mkdir /etc/certs
openssl genrsa -out /etc/certs/ca.key 2048 
Generating RSA private key, 2048 bit long modulus
....+++
..................................................+++
e is 65537 (0x10001)
```

 

```
openssl req -x509 -new -nodes -key /etc/certs/ca.key -subj "/CN=自己的仓库域名" -days 5000 -out /etc/certs/ca.crt
```

在进行安装的时候就会显示https访问

![img](./harbor使用img/https访问.png)

客户段连接仓库时需要配置：

```
mkdir -p /etc/docker/certs.d/域名
# -p 创建多级目录
```

使用scp或者rz将服务端创建的cert证书文件拷入该目录下，加载配置文件重启docker服务（在docker安装篇有），就可以使用docker login命令登录了，注意将仓库地址加入docker白名单。

无论是使用http/https，客户端与服务端的方式需要统一，否则会出现错误。

```
Error response from daemon: Get https://10.10.70.214/v1/_ping: dial tcp 10.10.70.214:443: getsockopt: connection refused
```

 

 