#泰华DevOps故障问题解决方案记录

1.1	业务产品相关

1.1.1	流水线中的项目编译构建基础镜像
 流水线中的的构建过程目前是由容器来提高构建环境，针对公司内目前的代码框架，通常是maven、ionic、node进行构建，基础镜像安装了需要的构建组件，基础镜像的Dockerfile如下所示：

1.	FROM 10.10.70.65/base/andreptb/maven:3.3.9-jdk8  
2.	COPY settings.xml /usr/share/maven/conf/settings.xml  
3.	COPY linux-x64-57_binding.node /usr/local/  
4.	ENV SASS_BINARY_PATH=/usr/local/linux-x64-57_binding.node   
5.	COPY  node-v8.6.0-linux-x64.tar.gz /home/node-v8.6.0-linux-x64.tar.gz  
6.	WORKDIR /home/  
7.	RUN tar xvf node-v8.6.0-linux-x64.tar.gz \  
8.	&& mv node-v8.6.0-linux-x64/ /usr/local/  
9.	RUN echo '' > /etc/apt/sources.list.d/jessie-backports.list \  
10.	&& echo "deb http://mirrors.aliyun.com/debian jessie main contrib non-free" > /etc/apt/sources.list \  
11.	&& echo "deb http://mirrors.aliyun.com/debian jessie-updates main contrib non-free" >> /etc/apt/sources.list \  
12.	&& echo "deb http://mirrors.aliyun.com/debian-security jessie/updates main contrib non-free" >> /etc/apt/sources.list  
13.	
14.	RUN apt-get update && apt-get install -y libltdl7  
15.	RUN  export PATH=/usr/local/node-v8.6.0-linux-x64/bin:$PATH \  
16.	&& npm config set registry https://registry.npm.taobao.org \  
17.	&& npm install -g @angular/cli \  
18.	&& npm install uglify-js -g \  
19.	&& npm install -g cordova \  
20.	&& npm install -g ionic  
21.	ENV PATH=/usr/local/node-v8.6.0-linux-x64/bin:$PATH  
22.	RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \  
23.	locale-gen  
24.	ENV LANGUAGE=en_US.UTF-8  
25.	ENV LANG=en_US.UTF-8  
26.	ENV LC_ALL=en_US.UTF-8  
    10.10.70.65的镜像仓库中的镜像名称为 10.10.70.65/base/ionic-maven:1.3
    1.1.2	编译构建后的web.xml包含乱码（中文乱码）
    前期版本中的Jenkinsfile中使用的Pod镜像名称为10.10.70.65/base/ionic-maven:1.0，由于存在语言环境的问题，对该基础镜像进行了更新，只需要将镜像修改为10.10.70.65/base/ionic-maven:1.3


1.2	Jenkins
1.2.1	Jenkins生成SVN中的用户名密码认证ID
0、	在新建SVN checkout需要的credentials


2、登录Jenkins界面：在Credentials菜单查看并获取，可以将该ID写入到Jenkinsfile文件中


1.2.2	Jenkins 出现无法连接jenkins agent
 通常存在以下两种可能 
a. Kubernetes资源不够，需要等待创建，如果等待时间非常长，可以将该构建删除，重新执行构建
b. 其他网络问题，导致jenkins-agent无法向jenkins master上报状态，此时也可以将该构建删除，重新执行构建
1.2.3	JenkinsFile中的Pod定义
在Jenkinsfile内定义Pod时，可以采用以下配置，该配置会固定将Jenkins的agent调度到node10节点，该节点资源充足，是为Jenkins预留的节点：

1.	podTemplate(label: label, yaml: """  
2.	apiVersion: v1  
3.	kind: Pod  
4.	metadata:  
5.	labels:  
6.	some-label: some-label-value  
7.	spec:  
8.	containers:  
9.	- name: maven  
10.	image: 10.10.70.65/base/ionic-maven:1.3  
11.	imagePullPolicy: Always  
12.	command:  
13.	- cat  
14.	tty: true  
15.	resources:  
16.	requests:  
17.	memory: "4096Mi"  
18.	cpu: "500m"  
19.	limits:  
20.	memory: "4096Mi"  
21.	cpu: "500m"  
22.	volumeMounts:  
23.	- mountPath: /var/run/docker.sock  
24.	name: docker-sock  
25.	- mountPath: /usr/bin/docker  
26.	name: docker  
27.	volumes:  
28.	- name: docker-sock  
29.	hostPath:  
30.	path: /var/run/docker.sock  
31.	- name: docker  
32.	hostPath:  
33.	path: /usr/bin/docker  
34.	tolerations:  
35.	- key: "jenkins"  
36.	operator: "Equal"  
37.	value: "value"  
38.	effect: "NoSchedule"  
39.	"""  
40.	)  
    1.2.4	Jenkins分配用户与角色
    需要安装插件Role-based Authorization Strategy，安装该插件后，会在Jenkins的系统管理界面出现Manager And Assign Roles

管理员用户登录Jenkins，创建用户与角色，并赋予权限
a．注册用户

b.创建Project Role，该角色的权限会匹配项目


a.	为用户赋权


1.3	Spinnaker
1.3.1	无法正常访问
当Spinnaker 无法正常访问时，通常是由于Kubernetes环境问题，解决方法主要有以下几点：
a．	尝试重启Kubernetes集群的全部服务器
b．	命令行登录10.10.70.61（root/1234546a?），查看是Spinnaker的哪个组件引起的异常：


对于状态为异常的组件可使用以下命令进行删除
kuectl delete pod 异常组件的名称 -n cicd 
执行删除后，该Pod会由Kubernetes自动重建，等待一段时间，再进行访问，看是否正常。
1.3.1.1	Spinnaker界面无法显示Pipeline数据
该问题通常是由Spinnaker依赖的redis重启导致，由于Spinnaker将数据存入Canssanda，并使用Redis作为缓存，如果Redis重启或者重建时，会需要一定的时间在数据库进行业务数据的同步操作。

1.4	Sonarqube
1.4.1	Sonarqube的用户权限配置
1、用户


1、	项目为私有

3、为项目分配用户，并设置权限



Sonnarqube 无法访问
通常是由于Kubernetes环境问题，可以尝试重建Sonnarqube 的组件，可以执行以下命令获取Sonarqube 组件的Pod名称

执行以下命令，进行删除操作：

执行删除命令后，Kubernetes会自动重建
1.5	Kubernetes
1.5.1	Kubernetes集群信息
1. [root@node1 ~]# kubectl get nodes  
2. NAME      STATUS    ROLES         AGE       VERSION  
3. node1     Ready     master,node   186d      v1.9.3+coreos.0  
4. node10    Ready     node          33d       v1.9.3+coreos.0  
5. node2     Ready     master,node   186d      v1.9.3+coreos.0  
6. node3     Ready     node          186d      v1.9.3+coreos.0  
7. node4     Ready     node          186d      v1.9.3+coreos.0  
8. node5     Ready     node          102d      v1.9.3+coreos.0  
9. node6     Ready     node          102d      v1.9.3+coreos.0  
10. node7     Ready     node          67d       v1.9.3+coreos.0  
11. node8     Ready     node          53d       v1.9.3+coreos.0  
12. node9     Ready     node          33d       v1.9.3+coreos.0  
13. 
14. [root@node1 ~]# cat /etc/hosts  
15. 127.0.0.1 localhost localhost.localdomain  
16. ::1 localhost6 localhost6.localdomain  
17. # Ansible inventory hosts BEGIN  
18. 10.10.70.61 node1 node1.cluster.local  
19. 10.10.70.62 node2 node2.cluster.local  
20. 10.10.70.63 node3 node3.cluster.local  
21. 10.10.70.64 node4 node4.cluster.local  
22. 10.10.70.54 node5 node5.cluster.local  
23. 10.10.70.57 node6 node6.cluster.local  
24. 10.10.71.31 node7 node7.cluster.local  
25. 10.10.71.35 node8 node8.cluster.local  
26. 10.10.71.40 node9 node9.cluster.local  
27. 10.10.71.36 node10 node10.cluster.local  
28. # Ansible inventory hosts END  

其中Kubernetes集群使用的共享存储（NFS）所在节点：10.10.70.39
代理节点10.10.70.58
镜像仓库：10.10.70.65
1.5.2	运行在容器内的应用出现无法连接数据库的问题
该问题是由于jdbc的问题，当数据库出现宕机一段时间后，再重新启动数据库后，应用程序是无法连接数据库的，需要通过重建应用来解决该问题
2、	Kubernetes某个节点无法SSH登录上去
可以通过vmware client登录到虚拟机控制台，手动重启虚拟机
 目前有两个vmware client： http://192.168.60.110   wangdekui/123456a?
​                     

Vmware client  http://10.10.70.201  administrator/password





 jenkins

 内存溢出，