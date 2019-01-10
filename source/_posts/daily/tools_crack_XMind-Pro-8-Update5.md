---
title: XMind Pro 8 Update5 下载破解安装指南 - 以Mac为例
date: 2017-11-07 05:17:30
updated: 2019-01-10 16:17:20
categories:
- 其它
tags:
- 修复问题
---
## 前言

&emsp;&emsp;更新Windows破解失效解决方法。（在Win环境下，修改`hosts`文件需要将域名指向`127.0.0.1`而非`0.0.0.0`，修改完之后，需要以管理员方式运行`powershell`，使用`ifconfig /flushdns`）更新本地dns缓存。以下为原回答。

&emsp;&emsp;当前时间：2017年11月7日，此方法适用所有终端平台[Mac Win Linux]，只不过略有不同(例如一些文件的位置，但是还是能够在网上找到的)，由于笔者手头需要安装在Mac下，因此以Mac为例子讲解。
![镇楼](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-11-07%2003.48.30.png)

<!-- moew -->
## 官网下载安装包

- [xmind官方地址](http://www.xmind.net/)：可能会比较慢
- [百度网盘地址](http://pan.baidu.com/s/1bzjqJs)：可能会很快失效
- <strong>国内网站的版本是不行</strong>的(即xmind.cn以及你所搜到的最多的makeding上的)，已经被国内的二道贩子公司修改过了，所以是一定不可能破解的，要去英文官网下载。才能用补丁破解。

## 补丁下载破解

1. [点击下载XMindCrack.jar](http://pan.baidu.com/s/1jIcO2mq)：可能会很快实效
2. 移动XMindCrack.jar到xmind安装目录文件夹下
  a.  Finder-->应用程序-->XMind-->右键-->包内容-->Contents-->Eclipse
  b.  ![如果一切顺利](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-11-07%2004.25.05.png)
3. 在XMind.ini 文末追加 -javaagent:/Applications/XMind.app/Contents/Eclipse/XMindCrack.jar
  a. 注意此处最好添加绝对路径，并且一定不要安装在随手会被删除的地方:)<strong>
4. 修改计算机的hosts文件,防止需要重复注册（即防止“流氓验证”）</strong>
  a. 打开Spotlight搜索：/private/etc/
  b. 选择在Finder中打开
  c. ![如果一切顺利](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-11-07%2004.19.15.png)
  d. 下拉找到hosts,拷贝到其他地方用文本编辑器打开，对照着文件前面的格式，末尾追加:127.0.0.1         www.xmind.net
5. 断网(其实已经不需要了，但是最好还是断网)
6. 打开软件，邮箱任意即可，填写序列号
```
XAka34A2rVRYJ4XBIU35UZMUEEF64CMMIYZCK2FZZUQNODEKUHGJLFMSLIQMQUCUBXRENLK6NZL37JXP4PZXQFILMQ2RG5R7G4QNDO3PSOEUBOCDRYSSXZGRARV6MGA33TN2AMUBHEL4FXMWYTTJDEINJXUAV4BAYKBDCZQWVF3LWYXSDCXY546U3NBGOI3ZPAP2SO3CSQFNB7VVIY123456789012345
```
## 结束语
&emsp;&emsp;这是我安装的最坎坷的一个软件（为此，我居然花了将近七个小时从10:00pm--05:00am），只能说XMind.cn实在是在防盗版这块做的很强大...以及他们的营销...
&emsp;&emsp;分享给有需要的人。
&emsp;&emsp;另外，如果你觉得这篇文章确实帮助到了你。可以通过下面的微信支付给我打赏:)谢谢。