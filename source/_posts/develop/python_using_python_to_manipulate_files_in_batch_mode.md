---
title: python之批处理命名文件
date: 2017-11-07 13:53:10
updated: 2017-11-07 13:53:10
categories:
- 网页开发

tags:
- Python
---
## 前言
最近经常上油管下东西，用的在线的网站"Youtube Multi Downloader Online"解析下载，还是蛮好用的（需要翻墙）。[链接地址](https://youtubemultidownloader.com/playlist.html)。就是每次下载下来视频都会带上前缀序号。有点烦人（取消勾选Add prefix order number还是不管用）。就打算用脚本语言处理一下。选了Python，可惜学了个半桶水，这么简单的东西，却总是在要用的时候，忘记该怎么写...
索性整理出来备记一下。

<!-- more -->
## 涉及的知识
- 字符串处理：
  1. 字符串查找
  2. 字符串切片
  3. 字符串拼接
- 基本的文件操作：
  1. 找出当前目录下的文件 
  2. 重命名文件
## 源码
```python
#coding=utf-8
import os
dirf = "/Users/Zoking/Downloads/graffle"#文件所在目录
indwx = "0"								#删除的内容标志，需要区别
for file in os.listdir(dirf):			#列出dirf目录下的所有文件
    if file.find(indwx) == 0:			#找到需要删除的位置
        nfile = file[3:len(file)]		#切片
        os.rename(os.path.join(dirf,file),os.path.join(dirf,nfile))#重命名文件rename(旧，新)
```
## 结束语
感觉自己真的是心贪术不专，需要好好梳理一下！