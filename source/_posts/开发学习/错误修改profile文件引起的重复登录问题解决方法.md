---
title: 错误修改profile文件引起的重复登录问题解决方法
date: 2017-11-10 22:15:26
categories:
- 网页开发
tags:
- Linux
---
# 前言
今天为了安装`go`错误修改了`/etc/profile`文件，导致`reboot`的时候一直循环登录。最终通过搜索解决了问题。这里记录一下。

<!-- more -->
# 解决方法
整个问题的关键是利用超级管理员权限进行修改前面被我修改错的的`etc/profile`文件。但是我的用户是普通用户根本就切换不了。`su`命令是报一堆错误。正确的做法如下：
1. `alt+ctl+shft+f1-6`进入`tty`界面
2. 用普通用户账号登录(我的是普通用户，如果是`root`账号则直接进入步骤4)
3. 执行`export PATH=/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin`
4. 正常用`vim`修改`/etc/profile`，完整命令如下：`sudo vim /etc/profile`
5. 使之起效`source /etc/profile`
6. 最后用`alt+ctl+shft+f7`命令回到图形化界面，正常登录。

# 结束语
操作系统真有趣：）