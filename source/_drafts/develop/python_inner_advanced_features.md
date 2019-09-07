---
title: python之高级语言特性：切片
date: 2017-12-11 20:19:34
updated: 2017-12-11 20:19:34
categories:
- 网页开发
tags:
- Python
---
# 前言
由于最近大都使用Python作为主要的开发语言，并且以后也将长期使用（用于编程科学计算相关的内容）等。决定还是好好学习一下python。
以下内容是学习[廖雪峰Python教程-切片](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431756919644a792ee4ead724ef7afab3f7f771b04f5000)的笔记梳理。

<!-- more -->
# 切片
对这种经常取指定索引范围的操作，用循环十分繁琐，因此，Python提供了切片（Slice）操作符，能大大简化这种操作。
`L[0:3]`表示，从索引`0`开始取，直到索引`3`为止，但**不包括索引`3`**。即索引`0`，`1`，`2`，正好是`3`个元素。
如果第一个索引是`0`，还可以省略：`L[:3]`
类似的，既然Python支持`L[-1]`取倒数第一个元素，那么它同样**支持倒数切片**:`L[-2:]`。记住倒数第一个元素的索引是`-1`。
什么都不写，只写`[:]`就可以原样复制一个`list`
tuple也是一种list，唯一区别是tuple不可变。因此，tuple也可以用切片操作，只是操作的结果仍是tuple。
Python没有针对字符串的截取函数，只需要切片一个操作就可以完成，非常简单。

# 小结
切片利用的是`[(起始位置，如果是0可以被省略):(切片结束位置，如果是全部可以被省略):(截取数据的步长，如果不指定，连同前面的`:`可以被省略)]`。

# 多维数组的切片
由于python不支持多维列表这一说（我们无法利用列表来模拟多维数组），因此我们得引入`numpy`进行转化。
也是前几个冒号组成范围，最后一个冒号是步长。如果省略则为1。

# 引用链接
[Python 基础——range() 与 np.arange()](http://blog.csdn.net/lanchunhui/article/details/49493633)
[http://old.sebug.net/paper/books/scipydoc/numpy_intro.html](http://old.sebug.net/paper/books/scipydoc/numpy_intro.html)
[廖雪峰Python教程-切片](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431756919644a792ee4ead724ef7afab3f7f771b04f5000)
