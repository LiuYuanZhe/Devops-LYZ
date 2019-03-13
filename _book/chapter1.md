# First Chapter

## Second
### three

在 Docker 里跑 Java，趟坑总结， jvm无法感知到自己在容器中进行，默认，堆的上限是物理机内存的四分之一，当容器的jvm没有设置xmx，即便容器内存设置的很大，也没有解决问题，导致容器会周期性重启（没有gc，逐渐累积到容器内存的限制值）。结论：要管控jvm 堆大小等参数，或使用特殊镜像。

在设置jvm启动参数的时候 -Xmx的这个值一般要小于docker限制内存数，个人觉得 -Xmx:docker的比例为 4/5 - 3/4

**a.JVM 做不了内存限制，一旦超出资源限制，容器就会出错**

**b.即使你多给些内存资源，也没什么卵用，只会错上加错**

**c.解决方案：用 Dockfile 中的环境变量来定义 JVM 的额外参数**

**d.更进一步：使用由 Fabric8 社区提供的基础 Docker 镜像来定义 Java 应用程序,将始终根据容器调整堆大小**

 





数据库无法加载：

怀疑项目启动连接数据库时，容器还未准备好网络。

因此呢，可以修改 c3p0 连接池配置，使 initialPoolSize=0。initialPoolSize 表示连接池初始化时创建的连接数，为0后，c3p0会在第一次接收用户请求时 才建立连接。

结果，没有用







sonarqube 账号密码: admin/admin

token:

liuyuanzhe: 119af2b611fb690815ecd1ce84fd3422c355510a

mvn:

 mvn sonar:sonar   -Dsonar.host.url=http://10.10.70.211:32119   -Dsonar.login=119af2b611fb690815ecd1ce84fd3422c355510a