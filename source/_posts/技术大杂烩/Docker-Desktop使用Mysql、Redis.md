---
title: Docker_Desktop使用Mysql、Redis
categories:
  - 技术大杂烩
tags:
  - Docker_Desktop
date: 2020-12-05 09:29:08
---







# Docker Desktop使用Mysql

1. 拉取镜像

```dockerfile
docker pull mysql
```

2. 启动镜像, 第一次启动最少需要指定`MYSQL_ROOT_PASSWORD`

```dockerfile
 docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=000 -d mysql
```

![](1.png)

3. 运行Mysql

```dockerfile
docker exec -it mysql mysql -uroot -p000
```

![](2.png)





# Docker Desktop使用Redis

1. 拉取镜像

```dockerfile
docker pull redis
```

2.启动镜像

```dockerfile
docker run -it -p 6380:6379 redis
```

![](3.png)

3. 运行Redis

```dockerfile
redis-cli -p 6380			# 指定端口，可开启多个redis服务，互不影响，本地服务均可开启
```

![](4.png)