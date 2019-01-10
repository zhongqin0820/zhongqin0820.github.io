---
title: 去除mac终端you have a new mail提示
date: 2018-07-23 20:10:30
updated: 2018-07-23 20:10:30
categories:
- 其它
tags:
- 修复问题
---
# 前言
&emsp;&emsp;Mac自从安装了Alfred之后,不知道下载了哪个插件脚本,导致每次打开终端都会提示:You have received a new mail...
&emsp;&emsp;命令行输入`mail`后会看到好几封发给`local`的消息,提示`Subject:cron...achromatopsia.qs`之类的信息,**应该是某个任务想要开启定时任务却没有权限导致的**.
&emsp;&emsp;因为一直没有找到解决办法(不知道这个`achromatopsia.qs`是什么鬼),忍了好久,上周终于把它解决了.记录一下.

<!-- more -->
# 解决方案
整个解决过程的脚本如下：
```bash
cd /usr/lib/cron # 会发现,其实在Finder中它只是/private/var/at的软链接
ll # 查看该目录结构下的内容,我们需要进入tabs目录,但是直接进入没有权限
sudo su # 获取管理员权限进行进一步操作
pwd # 打印当前目录会发现,的确是在/private/var/at中
cd tabs # 进入tabs目录后,会发现有一个以用户名命名的文件
vi 用户名(文件名)
# 注释掉对应的任务,我的内容是:
# 50 * * * * /Users/用户名/Library/achromatopsia.qs/achromatopsia.qs cr
```