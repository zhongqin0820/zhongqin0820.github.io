---
title: Go语言入门总结:工作空间
date: 2018-07-10 16:41:10
updated: 2018-07-10 16:41:10
categories:
- 网页开发

tags:
- Go
---
## 前言
梳理总结学习Go语言的笔记，关于Go中工作空间的理解以及设置。

<!-- more -->
## 工作空间结构
go语言的工作空间其实就是一个文件目录，目录中必须包含src、pkg、bin三个目录。
- src目录用于存放go项目源代码
- pkg目录用于package对象
- bin目录用于存放可执行对象。

使用go的编译命令工具可以将源代码或package编译后的二进制输出对应存储到bin（目标文件）和pkg（库文件）目录中。

```shell
go install # compile source code
go build && go install # compile after build a new package
```

## GOPATH环境变量
GOPATH环境变量支持多个值，如果你有多个工作空间，可以把多个工作空间值都添加到这个环境变量中，window系统使用分号`;`分隔不同值，Linux或Unix系统使用冒号`:`分隔不同值。**请不要将GOROOT目录添加到GOPATH**。
而为了方便开发，还需要将`$(GOPATH)/bin`加入系统变量`PATH`中。

## 结束语
GOPATH下的src目录就是接下来开发程序的主要目录，所有的源码都是放在这个目录下面，应该使用源码托管的方式进行对项目代码的组织管理。即`src/github.com/user_name/repo_name`下为对应工程目录。一般我们的做法就是一个目录一个项目；**文件夹名称一般是代码包名称**。
值得注意的是这种方式不支持gitlab。