---
title: Docker理论基础总结
date: 2019-04-07 10:53:11
updated: 2019-05-28 11:53:11
categories:
- 网页开发

tags:
- Docker
---
# 前言
总结Docker的理论基础与基本使用：基本Client命令、Dockerfile书写。

<!-- more -->
# 基本组成
Docker是典型的C/S架构，主要组件包括：
1. Docker Client：命令行客户端，提供Restful API，可通过多种方式与Docker Daemon通信
2. Docker Daemon：守护进程，运行在宿主机后台
3. Docker Image：镜像（静态）
4. Docker Container：容器（镜像的动态运行实例）
5. Docker Registry：镜像仓库

# 依赖的Linux内核特性
主要依赖的Linux内核特性：namespace(命名空间)以及cgroups(控制组)和advanced unified file system(AUFS)。
- namespace（命名空间**资源隔离**，6个）：UID(User ID)、PID(进程隔离)、NET(管理网络接口)、IPC(进程间通信)、MNT(文件挂载)、UTS(主机名和域名的隔离)
- control groups(cgroups**资源限制或控制、监控、计量**)：资源控制、优先级控制、资源计量、资源限制
- AUFS：多层统一文件系统，CoW（写时复制技术），添加一层可写层。节省空间。

二者结合赋予Docker Container的能力（资源隔离，资源限制或控制、监控、计量等）：
1. 文件系统隔离、控制
2. 进程隔离
3. 网络隔离
4. 资源隔离和分组

# 基本操作
基本操作分为对容器与镜像的基本操作。

## 容器基本操作
主要涉及docker client命令的基本使用。
```shell
docker run -d # 创建守护式（后台运行）容器
docker start -i xxx # 以交互式方式启动容器（重启），退出时可用Ctrl+P与Ctrl+Q以后台的方式继续运行容器
docker attach xxx # 附加到正在运行的容器中，与docker start 的区别是，后者是暂停的容器
docker exec xxx # 在运行的容器中启动新的进程，[-i,-t,-d] 对应 【交互式，指定终端，守护式运行】

docker inspect xxx # 查看容器配置
docker logs xxx # 查看容器的log输出 [-f,-t,--tail] 对应【持续，时间戳，尾部】

docker top xxx # 查看容器內实时进程状态
docker ps xxx # 与top 类似的查看命令
docker port xxx # 查看容器的映射端口
```

## 镜像基本操作
主要是对Dockerfile的字段说明。注意区分作用是在**镜像构建**阶段还是在**创建容器实例**阶段。
```shell
# 构建信息
FROM # 基础镜像信息
MAINTAINER # 镜像维护者信息

# 与命令相关
RUN ["echo", "hello"] # 指定构建镜像运行的命令
CMD # 指定创建的容器实例运行的命令，只能有一个
ENTRYPOINT # 与CMD类似，但是可以通过在创建容器时，利用--entrypoint覆盖

# 与文件相关
COPY # 复制宿主机文件/目录中的内容到容器
ADD # 相较COPY，可以是一个远程的资源文件路径URL（添加文件/目录中的内容）到容器
WORKDIR # 添加工作目录，具有连续性，/path/to/workdir
VOLUME # 挂载数据卷，建议使用数据卷容器替代

# 与权限相关
USER # 指定运行容器的username

# 与网络相关
EXPOSE # 指定暴露的端口号

# 其它
ONBUILD # 构建镜像阶段使用
ENV # 环境变量
ARG # 定义构建镜像时需要的参数
```

# 网络连接
主要包括：网络基础、容器间的连接、与外部网络的连接 三部分。这篇[博文](https://www.cnblogs.com/Hai--D/p/7017933.html)总结的很好。这里不再缀述。

# 资料汇总
- [Docker从入门到实践](https://www.bilibili.com/video/av42752665)
- [Docker入门学习笔记1](https://cvblogs.cn/2018/07/02/develop/docker_basic_intro/)
- [Docker入门学习笔记2](https://github.com/zhongqin0820/coding-playground/wiki/Docker%E5%85%A5%E9%97%A8%E5%AD%A6%E4%B9%A0)
- **《自己动手写Docker》**