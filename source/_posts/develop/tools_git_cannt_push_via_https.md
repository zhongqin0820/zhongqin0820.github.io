---
title: git利用https无法push解决方法
date: 2017-12-08 10:22:26
updated: 2017-12-08 10:22:26
categories:
- 其它
tags:
- 修复问题
---
#### 错误信息：`fatal: Unable to find remote helper for 'https'`
昨天想要提交代码的时候，死活无法`push`，稍晚的时候才想起来原来是自己太着急了，代码都没有`commit`，那么何谈`push`呢。

<!-- more -->
> 切记本地与远端的区别。远端只需要将它看作是一个存储数据作备份的地方。真正的内容其实是需要在本地完成的。而`git`这个工具的本质就是，先`add`修改（如果是对已有文件作修改则只需要用`commit -am`提交修改信息。`git`会保存所有的提交版本记录。）

另外，由于用`conda`进入了虚拟环境，导致`push`的时候依旧报错。报错信息是`fatal: Unable to find remote helper for 'https'`。上网查找的时候说是我没有安装`curl`等（但是其实根据一个正常工作的`git config --list`信息对比，我的`git`及`curl`是可以正常用的）。
> 最后，其实是因为我进入了`conda`的环境。退出来虚拟环境，就可以正常提交了。

#### 结束语
还是环境变量的问题。