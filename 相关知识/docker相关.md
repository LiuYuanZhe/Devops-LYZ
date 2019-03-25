docker修改启动命令：

```sh
# docker run -i -t image /bin/bash ./home/start.sh	
```

记录一个自己的命令。

指定端口映射启动一个镜像及内部服务

```
# docker run -i -t -d -p host_port:container_port  image  /bin/bash ./home/start.sh
```

