---
title: Linux之curl添加Https
date: 2017-11-09 22:55:11
updated: 2017-11-09 22:55:11
categories:
- 网页开发

tags:
- Linux
---
# 前言
系统时Ubuntu 16.04 今天有需要使用`curl`命令`down`一些`https`的内容。提示`Protocol https not supported or disabled in libcurl`。用`curl -V`一看发现支持的协议果然缺了`https`...上网查了一些资料，历经坎坷。总算最后解决了问题。整理备忘一下。

<!-- more -->
# openssl
可以用`openssl version -a`确认一下`openssl`是否已经安装，如果没有则需要下载安装。因为如果`curl`要支持`https`，就必须先安装`openssl`。
## 安装方法
可以有两种方法进行安装。我用了第二种方法。
### 方法一
```bash
sudo apt-get install openssl 
sudo apt-get install libssl-dev
```
我试了并不好用。提示我安装`libssl-dev`安装的依赖包有版本冲突...遂放弃。

### 方法二
到`openssl`官网下载压缩包 http://www.openssl.org/source/ 我这里下载的是`openssl-1.0.1em.tar.gz`
```bash
tar zxvf openssl-1.0.1q 
./config shared #（安装的默认路径是/usr/local/ssl，如果你想更改目录，请加上–prefix=/yourpath）这一步对后面正常覆盖安装curl更新协议非常关键 

make 
make test 
sudo make install#必须用root权限安装，因为之前不看log...导致，一直以为自己安装是正确的
```
`vim /etc/ld.so.conf`在文件末尾加入 `/usr/local/ssl/lib`，刷新动态库配置
用`openssl version -a `测试，可以正常`log`出相关信息。

# curl
记住自己之前安装`openssl`的路径。可以使用`which openssl`查看，我的在`/etc/lib/openssl`?有点记不太清了..
```bash
tar zxvf curl-7.30.0.tar.gz 
cd curl-7.30.0/ 
./configure –with-ssl=/etc/lib/openssl#（我这里没有配置prefix选项，直接将curl安装到默认的目录/usr/local） 
```
`config`之后，会有以下信息打出，可以在`Protocols`一行确认一下是否引入了协议。要是没有就得再仔细找找问题。 
如果一切顺利，使用`make`之后再`sudo make install`。
最后使用`curl -V`进行验证。

# 引用链接
- [curl 不支持 https](http://blog.csdn.net/Timsley/article/details/50782742)
- [ubuntu 14.04下安装openssl](http://blog.csdn.net/Timsley/article/details/50776615)

# 结束语
尽管距离解决完问题还没有一天但是还是有点忘记了具体的细节。因此备记一下还是有点必要的。另外，发现，本来想用`echo $?`来筛查错误命令返回码。但是发现有的命令不管错误还是正确总是返回0:）
