---
title: Ubuntu安装Docker和docker的使用
categories:
  - 技术大杂烩
tags:
  - Docker
date: 2020-12-01 10:58:04
---





# Docker 概述

>`Docker `是一个开源的应用容器引擎，基于`Go语言`并遵从 Apache2.0 协议开源。
>
>`Docker `可以让开发者打包他们的应用以及依赖包到一个`轻量级`、`可移植`的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
>
>容器是完全使用`沙箱机制`，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

# Docker应用场景

> `Web `应用的自动化打包和发布。
>
> 自动化测试和持续集成、发布。
>
> 在服务型环境中部署和调整数据库或其他的后台应用。
>
> 从头编译或者扩展现有的 `OpenShift` 或 `Cloud Foundry` 平台来搭建自己的 `PaaS` 环境

# Docker 的优点

> `Docker` 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来`快速交付`，`测试`和`部署代码`，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。





# 在Ubuntu中安装Docker

1. 更新ubuntu的apt源索引

```dockerfile
sudo apt-get update
```

2. 安装包允许apt通过HTTPS使用仓库

```dockerfile
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

3. 添加Docker官方GPG key

```dockerfile
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4. 设置Docker稳定版仓库

```dock
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

5. 添加仓库后，更新apt源索引

```dockerfile
sudo apt-get update
```

6. 安装最新版Docker CE（社区版）

```dockerfile
sudo apt-get install docker-ce
```

7. 检查Docker CE是否安装正确

```dockerfile
sudo docker run hello-world
```

8. 出现如下信息，表示安装成功

![](1.png)



# Docker使用

```dockerfile
# 启动docker
sudo service docker start

# 停止docker
sudo service docker stop

# 重启docker
sudo service docker restart
```

## 容器使用

1. 启动容器

+++info  参数说明

-i: 交互式操作

-t: 终端

redis:redis镜像

+++

```dockerfile
docker run -it redis
```

2. 停止一个容器

```dockerfile
docker stop <容器 ID>
```

3. 重启,停止的容器

```dockerfile
docker restart <容器 ID>
```

4. 进入容器

- 在使用 `-d` 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入

```dockerfile
docker attach <容器 ID>
docker exec -it <容器 ID>           # 推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。
```

5. kill掉一个已经在运行的容器

```dockerfile
docker kill <容器 ID>
```

6. 清理掉所有处于终止状态的容器

```
docker container prune
```

7. 删除容器

```dockerfile
docker rm -f <容器 ID>  
```



## 镜像使用

1. 列出镜像

```dockerfile
docker images
```

+++info 各个选项说明

REPOSITORY：镜像所在的仓库名称

TAG：镜像标签

IMAGEID：镜像ID

CREATED：镜像的创建日期(不是获取该镜像的日期)

SIZE：镜像大小

+++

![](2.png)

2. 搜索镜像

```dockerfile
docker search redis
```

+++info 各项说明

NAME: 镜像仓库源的名称

DESCRIPTION: 镜像的描述

OFFICIAL: 是否 docker 官方发布

START: 类似 Github 里面的 star，表示点赞、喜欢的意思。

AUTOMATED: 自动构建。

+++

![](3.png)

3. 拉取镜像

>Docker维护了镜像仓库，分为共有和私有两种，共有的官方仓库[Docker Hub(https://hub.docker.com/)](https://hub.docker.com/)是最重要最常用的镜像仓库。私有仓库（Private Registry）是开发者或者企业自建的镜像存储库，通常用来保存企业 内部的 Docker 镜像，用于内部开发流程和产品的发布、版本控制。

```dockerfile
docker pull django
```

4. 删除镜像

```dockerfile
docker image rm 镜像名或镜像id
```

5. 镜像备份与迁移

```dockerfile
docker save -o ./ubuntu.tar ubuntu		# 镜像打包成文件
docker load -i ./ubuntu.tar			# 将镜像加载到本地
```



## 仓库管理

> `仓库（Repository）`是集中存放镜像的地方。以下介绍一下 [Docker Hub](https://hub.docker.com/)。当然不止 Docker hub，只是远程的服务商不一样，操作都是一样的。

### Docker Hub

> 目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)。
>
> 大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

### 注册

> 在 [https://hub.docker.com](https://hub.docker.com/) 免费注册一个 Docker 账号。



- 登录

```dockerfile
docker login
```

- 退出

```dockerfile
docker logout
```

- 可以通过 docker search 命令来查找官方仓库中的镜像，并利用 docker pull 命令来将它下载到本地。
- 以 Mysql为关键词进行搜索

```python
docker search mysql
```

- 使用 docker pull 将官方 ubuntu 镜像下载到本地：

```dock
docker pull mysql
```

- 推送对象
  - 用户登录后，可以通过 docker push 命令将自己的镜像推送到 Docker Hub。
  - 以下命令中的 username 请替换为你的 Docker 账号用户名。

```dockerfile
docker tag mysql username/mysql
docker image ls
docker push username/mysql
docker search username/mysql
```

