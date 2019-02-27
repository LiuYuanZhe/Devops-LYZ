# docker整理

`docker run --name postgres -v /data/pg:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD=postgres -d 10.10.70.65/base/postgres:9.5`

 数据库启动命令

-e：设置环境变量





## docker fastdfs部署方法



```shell
docker pull 10.10.70.65/base/fastdfs:5.11
docker inspect 10.10.70.65/base/fastdfs:5.11
docker run -dti --network=host --name tracker -v /data/fastdfs/tracker/:/var/fdfs 10.10.70.65/base/fastdfs:5.11 tracker
docker run -d --network=host --name storage -e TRACKER_SERVER=10.10.70.214:22122 -v /data/fastdfs/storage/:/var/fdfs -e GROUP_NAME=group1 10.10.70.65/base/fastdfs:5.11 storage
docker ps
docker exec -it storage /bin/bash

```

启动一个tracker负责追踪，storage负责存储