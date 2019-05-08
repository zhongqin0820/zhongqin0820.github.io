---
title: 杂谈:爬CSDN数据
date: 2019-05-08 16:33:45
updated: 2019-05-08 21:42:24
categories:
- 其它

tags:
- 修复问题
---
# 前言
学习搜索引擎使用时，在谷歌搜索自己的常用网名，看到了自己本科期间的CSDN博客内容。没想到，隔着几个条目，还看到了一个同样署名，发布着同样内容，只不过内容是繁体字（猜测可能跟这个网站拥有者相关），并且源地址是[https://twblogs.net/](https://twblogs.net/)。
特此简单分析一下，要如何做到这种水平的爬虫。

<!-- more -->
# whois
首先，这个网站并没有备案之类的内容（直接联系方式）。于是，我想到是否可以通过whois找到网站的拥有者的信息，从而协调删除我的文章。但是，并没有找到有效的信息，显示的是域名提供商godaddy的信息。下面给出log信息：

```txt
Domain Name: TWBLOGS.NET
 Registry Domain ID: 2330628228_DOMAIN_NET-VRSN
 Registrar WHOIS Server: whois.godaddy.com
 Registrar URL: http://www.godaddy.com
 Updated Date: 2019-03-03T19:49:21Z
 Creation Date: 2018-11-08T16:30:46Z
 Registry Expiry Date: 2019-11-08T16:30:46Z
 Registrar: GoDaddy.com, LLC
 Registrar IANA ID: 146
 Registrar Abuse Contact Email: abuse@godaddy.com
 Registrar Abuse Contact Phone: 480-624-2505
 Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
 Domain Status: clientRenewProhibited https://icann.org/epp#clientRenewProhibited
 Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
 Domain Status: clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited
 Name Server: JOBS.NS.CLOUDFLARE.COM
 Name Server: LANA.NS.CLOUDFLARE.COM
 DNSSEC: unsigned
 URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
```

没办法，既然无法删除，就来分析一下，这样一个爬虫克隆网站可能需要用到的技术。

# Clone CSDN分析
## 爬虫
- 一开始，我想到CSDN是否提供公共API提供访问服务。查询之下，发现[https://open.csdn.net/](https://open.csdn.net/)已经关闭。
- 不过，感觉CSDN的内容还是挺好爬的（RESTful的接口一目了然）。就是不知道其反爬机制做的如何。

### 粒度：图片
- [CSDN博客原文](https://blog.csdn.net/zokingo/article/details/78165518?utm_source=blogxgwz4)发表时间是`2017年10月06日 15:46:46`。由于图片使用的是七牛云的图床，被收回域名之后就宕掉了。
- [twblogs上的文章](https://www.twblogs.net/a/5b8319f72b717766a1eb0deb)，只爬取了作者信息以及文章相关的信息，但是并没有保存图片，因此当原文无法访问的时候，这个也无法访问。
- [被另一个网站爬取的文章](http://www.th7.cn/system/win/201710/230264.shtml)的时间虽然显示的是：`2017-10-06 20:30:31`，但是显然不可信。不过，这个网站的爬虫爬取内容比较精细，爬取内容的同时，还保存了图片，这也是为什么上面两个图片都宕掉了，而这个网站的还能正常显示。但是，并没有建立作者的关联关系。虽然有作者等信息，但是并没有独立的主页等。

### 粒度：评论
- 从[有评论的CSDN博客原文](https://blog.csdn.net/zokingo/article/details/46351577)与[twblogs上的克隆文](https://www.twblogs.net/a/5b8319ec2b717766a1eb0dbb/zh-cn)对比来看，其只爬作者及相关的文本内容而不爬评论。
- 很好理解，这样子一个系统的设计初衷就是爬取数据进行展示，肯定是最大限度的减少额外的资源占用。

### 步骤分析
- 用户名：**只需要知道用户名即可**，可惜忘记CSDN账号密码，我在想可能可以通过关注的方式获取对方的关注列表，从而获取用户列表。下面仅以爬取文章为例展开介绍，以用户`m0_38132420`为例
- 获取用户`m0_38132420`在CSDN对应的文章列表：[https://blog.csdn.net/m0_38132420/](https://blog.csdn.net/m0_38132420/)，获取对应文章的URL（主要是用户名和文章ID）。
- 拉取数据：[https://blog.csdn.net/m0_38132420/article/details/71699815](https://blog.csdn.net/m0_38132420/article/details/71699815)
- 文章页需要处理"获取更多"按钮，解析文章主体，获取标签tag列表（如有，需要按tag对其进行分类）等。
- 上述，最复杂的是文章页解析。

## 特色功能
- 简体转繁体，一开始看到文章同时有[简体](https://www.twblogs.net/a/5b8319ec2b717766a1eb0dbb/zh-cn)与[繁体](https://www.twblogs.net/a/5b8319ec2b717766a1eb0dbb)版本的时候还挺意外的，原来利用一些现有的工具就可以简单实现了。
- 该网站作者在设计API的时候还是挺考究的，将资源标识符hash加密存储，值得学习。

# 结束语
感叹一下数据隐私的重要性。在想，成立一家互联网隐私数据清理网站需要什么技术。
