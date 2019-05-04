---
title: 一些容易被我忘记的控制台命令以及快捷键
date: 2017-11-05 19:47:26
updated: 2019-4-09 21:35:26
categories:
- 网页开发
tags:
- 开发总结
---

## 前言
总结一些容易被我忘记的控制台命令，以及面试过程中常问的几个操作。


<!-- more -->
## 面试常问
### 进程相关命令
```shell
top
ps
pstree
kill
```

### 系统资源相关命令
系统资源包括：CPU、内存、磁盘空间、设备...
```shell
iostat
```

### 网络相关的命令
```shell
ping
netstat
iptables
ifconfig
ipconfig
```

### 文件查看相关命令
```shell
cat
more
less
grep
find
```

## 一些容易被我忘记的控制台命令

```shell
# 文件/目录 基本操作
cd -                #返回上一个访问的目录
cp 文件名 目标目录    #将文件拷贝到目标目录下

# 文件查看
cat 文件名(|less)    #在终端下查看文件，可以利用管道技术，配合more/less使用
more                #按页来查看文件的内容(只能往后)
less                #往前往后翻看文件

# 中断操作
control+d           #中断a.out运行
```

## 一些浏览器快捷键

``` console
cmd + 鼠标点击       #在新标签页打开点击的链接
```

## 参考资料
- [Linux中more和less命令用法](https://www.cnblogs.com/aijianshi/p/5750911.html)

