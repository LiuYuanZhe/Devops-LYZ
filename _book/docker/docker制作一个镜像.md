安装完docker之后，准备好基础镜像后。

```
FROM centos:7
# FROM提供基础镜像
MAINTAINER lyz 870873754@qq.com

#RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 运行命令，在前一个命令的基础上创建容器中执行
RUN sed -i.bak 's#aliyun\.com#aliyuncs.com#' /etc/yum.repos.d/CentOS-Base.repo
RUN yum install wget -y
RUN wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" https://edelivery.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.tar.gz
RUN wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
RUN wget http://mirrors.hust.edu.cn/apache/ant/binaries/apache-ant-1.9.13-bin.tar.gz
RUN wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.35/bin/apache-tomcat-8.5.35.tar.gz
RUN cd /opt
RUN tar -xvf apache-tomcat-8.5.35.tar.gz -C /opt/
RUN tar -xvf apache-ant-1.9.13-bin.tar.gz -C /opt/
RUN tar -xvf jdk-8u191-linux-x64.tar.gz -C /opt/
# ENV是设置环境变量
ENV JAVA_HOME /opt/jdk1.8.0_191
ENV ANT_HOME /opt/apache-ant-1.9.13
ENV CLASSPATH .:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH $JAVA_HOME/bin:$ANT_HOME/bin:$PATH
RUN wget -P /opt/apache-tomcat-8.5.35/webapps http://mirrors.jenkins-ci.org/war/2.150/jenkins.war
RUN yum install git -y
CMD /opt/apache-tomcat-8.5.35/bin/catalina.sh run
# expose memcached port 指定端口与容器外通信
EXPOSE 8080
```

 

```
nohup docker build -t jenkins/centos:v2 . >docker:v2.log &
或
docker build -t jenkins/centos:v2
```

 

如果是内网环境也可以将wget的内容使用copy命令复制到镜像内。

-d:以守护进程启动

-i:容器的标准输入开启

-t:启动一个伪终端并绑定到启动的容器中

```
docker run -d -i -t --name jenkins_procloud -p 8080:8080 jenkins/centos:v1
```

启动

<http://94.191.66.171:8080/jenkins/> 可以访问了

 