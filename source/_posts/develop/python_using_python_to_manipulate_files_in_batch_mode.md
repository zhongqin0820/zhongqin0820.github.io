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
使用在线的网站"Youtube Multi Downloader Online"解析下载油管视频，[链接地址](https://youtubemultidownloader.com/playlist.html)。取消勾选Add prefix order number还是不管用，每次下载下来视频都会带上前缀序号。使用脚本对其进行处理。
整理出来备记一下。同时更新整理关于文件相关的操作。

<!-- more -->
# 批处理去除文件前缀
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

# 文件基本操作知识梳理
## `open()`函数
- 指定文件名
- 指定操作模式：mode
    - 默认r（写）
    - r,w,a（写、读、追加）
    - r+,w+,a+（+表示更新）
    - b（以二进制形式，默认为文本模式）

- 指定编码信息：encoding
    - 默认None

## 异常机制
- FileNotFoundError
- LookUpError
- UnicodeDecodeError

## 文件读写对比
可以指定size限制每次读取的数据量，通常不使用。

- read([size])
- readline([size])
- readlines([size])

```py
# 使用for语句，比较适用于打开比较大的文件
file = open('data')
try:
    for line in file:
        print(line)
finally:
    file.close()

# 使用with语句，可以保证文件关闭
with open('data') as file:
    while True:
        line = file.readline()
        if line:
            print(line)
        else:
            break

# 一次读入文件所有行，并关闭文件
with open('data') as file:
    lines = file.readlines()
    for line in lines:
        print(line)
```

## 其它
- json模块
- pickle模块
- shelve模块

# 学习资料
- [千行代码入门Python](https://github.com/xianhu/LearnPython/blob/93bb78ab29707008416aaa495bbe81d70fdc918c/python_base.py#L302)