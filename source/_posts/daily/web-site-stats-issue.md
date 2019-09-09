---
title: 杂谈：好迷的站点数据
date: 2019-08-18 22:56:26
updated: 2019-08-18 22:56:26
categories:
- 其它

tags:
- 修复问题
---
# 前言
最近想要绑定新的域名，因为不想备案，而国内不备案的主机只要有https不备案也可以访问，加之一直用的cloudflare的服务，非常方便。一直以来，都觉得这个网站应该只有爬虫或者搜索引擎会访问。但是，居然在cloudflare的数据统计处看到有10k+的SSL请求/月。好迷的数据。
<div style="width: 300px; margin: auto">

![cloudflare stat](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/tools/cloudflare-stat.png)
</div>

<!-- more -->
# 其它
刚好自己最近在做的一个小项目也涉及这种跳转链接计数的问题。
猜测它是将所有资源的请求过程都记录了。每个资源对应一个SSL请求，这样也许说的通。但是，拿一篇文章需要请求10个资源来说，似乎还是有些夸张。

# Changelog
- 2019/08/18：第一版内容