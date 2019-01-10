---
title: Docker入门基础总结
date: 2018-07-02 11:54:11
updated: 2018-07-02 11:54:11
categories:
- 网页开发

tags:
- Docker
---

## 前言
总结Docker的基础用法以及Docker-Compose的使用。

<!-- more -->
## Docker
每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。
为了能够保存（持久化）数据以及共享容器间的数据，`Docker`提出了`Volume`的概念.
### 初始化`Volume`：共享宿主机数据
与Dockerfile不同的是,`-v`可以在容器上挂载指定的主机目录:`docker run -v /home/adrian/data:/data debian ls /data`
一般情况下,两种方式进行共享容器间的数据.
#### 运行时
```bash
docker run -it --name container-test -h CONTAINER -v /data debian /bin/bash
```
#### Dockerfile
在Dockerfile中通过使用`VOLUME`指令来达到相同的目的.

### docker build
```bash
docker build -t 目标镜像名 Dockerfile所在目录
```
### 重命名镜像
```bash
docker tag IMAGEID(镜像id) REPOSITORY:TAG（仓库：标签）
```

## Docker-Compose
docker compose 是一个整合发布应用的利器。docker-compose 是用来做docker 的多容器控制。如何编排 docker compose 配置文件是很重要的。
### 文件配置
compose 文件是一个定义服务、 网络和卷的 YAML 文件 。Compose 文件的默认路径是 ./docker-compose.yml
```bash
docker container create
docker network create
docker volume create
```
## 结束语
关于Docker的使用需要通过实际的项目去熟悉，如果只是知道一些概念会造成自己已经学会，但真正到使用的时候依旧无法手写Dockerfile...

## 参考链接
- [Docker入门教程](http://www.docker.org.cn/book/docker/what-is-docker-16.html)
- [docker-compose.yml 配置文件编写详解](https://blog.csdn.net/qq_36148847/article/details/79427878)
- [使用docker-compose 大杀器来部署服务 上](https://www.cnblogs.com/neptunemoon/p/6512121.html)