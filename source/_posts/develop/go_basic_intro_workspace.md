---
title: Go语言入门总结之工作空间
date: 2018-07-10 16:41:10
categories:
- 网页开发

tags:
- Go
---
## 前言
&emsp;&emsp;梳理总结学习Go语言的笔记，关于Go中工作空间的理解以及设置。

<!-- more -->
## 参考链接
&emsp;&emsp;主要参考博客：[go语言的工作空间和GOPATH环境变量](http://lib.csdn.net/article/go/34084)

## 工作空间结构
&emsp;&emsp;go语言的工作空间其实就是一个文件目录，目录中必须包含src、pkg、bin三个目录。
&emsp;&emsp;其中src目录用于存放go源代码，pkg目录用于package对象，bin目录用于存放可执行对象。
&emsp;&emsp;使用go的编译命令工具可以将源代码或package编译后的二进制输出对应存储到bin和pkg目录中。
```shell
go install # compile source code
go build -> go install # compile again after build a new package
```
&emsp;&emsp;**一个工作空间中通常都会包含多个仓库**

## GOPATH环境变量
&emsp;&emsp;GOPATH环境变量支持多个值，如果你有多个工作空间，可以把多个工作空间值都添加到这个环境变量中，window系统使用分号";"分隔不同值，Linux或Unix系统使用冒号”:“分隔不同值。

## How to Write Go Code
&emsp;&emsp;`go tool`
&emsp;&emsp;go 命令依赖一个重要的环境变量：$GOPATH
### GOPATH
&emsp;&emsp;Most Go programmers keep all their Go source code and dependencies in a single workspace.
&emsp;&emsp;GOPATH下的src目录就是接下来开发程序的主要目录，所有的源码都是放在这个目录下面，那么一般我们的做法就是一个目录一个项目；**文件夹名称一般是代码包名称**
### go get
&emsp;&emsp;`go get` will `fetch`, `build`, and `install` it automatically

## 结束语
&emsp;&emsp;应该使用源码托管的方式进行对项目代码的组织管理。即可以在`src/github.com`下创建自己对应的目录。值得注意的是不支持`gitlab.com`...