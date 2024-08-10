---
title: Docker 基础
author: shane
date: 2024-08-08
description: Docker 容器基础
categories: [Linux]
tags: [Docker, archlinux]
comments: true
toc: true
---

## Docker 简介

[Docker Hub](https://hub.docker.com/)

## 常见参数

- `docker run`: 创建并运行一个容器
- `-d`: 让容器在后台运行
- `-name`: 给容器取一个名字以示区分
- `-p`: 端口映射，前为宿主机端口，后为容器内端口
- `-e`: 环境变量，根据不同镜像的文档决定

## 常见命令

- `docker pull`: 从镜像仓库下载镜像
- `docker push`: 推送镜像
- `docker images`: 查看所有本地镜像
- `docker rmi`: 删除本地镜像
- `docker build`: 从 DOCKERFILE 中构建镜像
- `docker run`: **创建并运行**镜像
- `docker stop`: 将容器进程停止
- `docker start`: 启动容器进程
- `docker ps`: 查看当前容器的运行状态
- `docker rm`: 删除容器，区别于 `rmi` 是删除镜像
- `docker exec`: 进入容器进行一些调整
- `docker info`: 查看当前 docker 信息
- `docker system df`: 查看磁盘使用情况

## Docker 相关位置

- docker 根目录: `/var/lib/docker/`
- 镜像存储位置: `/var/lib/docker/overlay2/`
- 镜像元数据: `/var/lib/docker/image/`
- 容器数据: `/var/lib/docker/containers/`

> 直接操作这些目录容易出现问题，如果要修改 docker 的默认存储位置，可以修改 Docker 守护进程的配置文件。
{: .prompt-tip }

## 数据卷

每个镜像中是软件所需的最小系统，无法修改其中的内容，此时就需要用到数据卷对容器内文件进行修改。数据卷是一个虚拟目录，是**容器内目录**与**宿主机目录**之间的桥梁。

- `docker volume create`: 创建数据卷
- `docker volume ls`: 查看所有数据卷
- `docker volume rm`: 删除指定数据卷
- `docker volume inspect`: 查看某个数据卷的详情
- `docker volume prune`: 清楚数据卷

在执行 `docker run` 命令时，使用 `-v 数据卷:容器内目录` 可以完成数据卷挂载，因此我们需要先删除已有的容器。

> 当创建容器时，如果挂在了数据卷且数据卷不存在，会自动创建数据卷。
{: .prompt-tip }

## 自定义镜像

### 镜像结构

docker 中的镜像是**分层(layer)**的，添加安装包、依赖、配置等，每次操作都会形成一个新的层。**基础镜像**包括应用依赖的系统函数库、环境、配置、文件等。

如果 `docker pull` 下载的镜像发现本地已经存在相同的层，就不会重复下载。

### Dockerfile

通过一些指令，基于基础镜像描述镜像结构，实现镜像环境构建。

| 指令 | 描述 |
|:---:|:---:|
| FROM | 指定基础镜像 |
| MAINTAINER | 指定维护者信息 |
| RUN | 运行命令 |
| CMD | 指定容器启动时要运行的命令 |
| EXPOSE | 声明暴露的端口 |
| ENV | 指定环境变量 |
| ADD | 复制指定的 <src> 路径下的内容到容器中的 <dest> 路径下 |
| COPY | 复制本地主机的 <src> 路径下的内容到容器中的 <dest> 路径下 |
| ENTRYPOINT | 指定容器启动后的命令 |
| VOLUME | 创建一个可以从本地主机或其他容器挂载的挂载点 |
| USER | 指定运行容器时的用户名或 UID |
| WORKDIR | 指定工作目录 |
| ONBUILD | 配置当所创建的镜像作为其他新创建镜像的基础镜像时，所执行的操作指令 |

例如 java 项目，可以直接基于 JDK 为基础镜像，省略繁琐的步骤。

```dockerfile
# 基础镜像
FROM openjdk:11.0-jre-buster
# 拷贝jar包
COPY docer-demo.jar /app.jar
# 入口
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

编写好 Dockerfile 后，可以使用下面的命令构建镜像

```shell
docker build -t IMAGE_NAME:TAG .
```
其中

- `-t`: 给镜像命名，格式为 `NAME:TAG`，不指定 tag 时，默认为 latest
- `.`: Dockerfile 所在的目录

## 自定义 Docker 网络

如果要实现 Docker 容器名相互访问，需要自定义 Docker 网络。

- `docker network create`:  创建一个网络
- `docker network ls`: 查看所有网络
- `docker network rm`: 删除指定网络
- `docker network prune`: 清除未使用的网络
- `docker network connect`: 使指定容器连接加入某网络
- `docker network disconnect`: 使指定容器连接离开某网络
- `docker network inspect`: 查看网络详细信息

在容器创建的时候可以添加 `--network NETWORK_NAME` 连接自定义网络。
