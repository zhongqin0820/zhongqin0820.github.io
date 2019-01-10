---
title: softmax用在哪里？
date: 2017-12-30 22:23:10
updated
categories:
- 深度学习
tags:
- 基础概念

---
# 前言
相信学习机器学习过程中，会接触到的一个名词就是激活函数。譬如Sigmoid、ReLu以及Softmax。
今天看到自己TODOList有一个softmax，发现自己又忘了它是干什么的了。
于是借着机会再学一遍= =以下是笔记。
建议直接学习Siraj的视频！

<!-- more -->
# 引用链接
[深度学习常用激活函数之— Sigmoid & ReLU & Softmax](http://blog.csdn.net/leo_xu06/article/details/53708647)
[Siraj Raval激活函数讲解](https://weibo.com/tv/v/F52uUDZlF?fid=1034:f52462e17a913be82dc3de16b8b73811)
# softmax作用
它保证所以的输出神经元之和为1.0，所以一般输出的是小于1的概率值，可以很直观地比较各输出值。只用于多于一个输出的神经元。

# 激活函数
学习了Siraj Raval关于激活函数的讲解之后对其有了更深的印象。
激活函数是深度学习中的东西，因为构建一个网络要求它是非线性的（这样它才能表达更多的信息。）

# 结束语
有一种学了很多，想动手实践一下的冲动！但是又担心自己做不好= =