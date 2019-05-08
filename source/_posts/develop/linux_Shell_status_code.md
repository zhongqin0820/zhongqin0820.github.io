---
title: linux之Shell命令的状态码
date: 2017-11-08 21:59:10
updated: 2019-05-08 12:00:11
categories:
- 网页开发

tags:
- Linux
---

## 前言
通过书写shell脚本可以便捷的执行任务，状态码对shell脚本具有重要意义。为了规范书写，这篇文章就状态码的查看、意义、用法进行简单梳理。

<!-- more -->
退出状态码是一个 `0 ~ 255` 之间的整数值，在命令结束运行时由命令传给shell。你可以捕捉这个值并在脚本中使用。

## 查看状态码
Linux 使用了`$?` 专属变量来保存上个执行的命令的退出状态码。你必须在要查看的命令之后马上查看或使用`$?`变量。它的值会变成shell中执行的最后一条命令的退出状态码。

- 注意：`$?`是变量。所以要在控制台打印它。不能直接输入：`$?`,而是应该要使用`echo`这样的命令打印到控制台。

## Linux 状态码的意义

默认状态下，shell脚本会以脚本中的最后一个命令作为退出状态码。

所以一般情况下，在shell脚本中以 exit 命令的值来指定shell命令的退出状态码。但是退出状态码的范围是 0 ~ 255, 退出值超出这个范围将会执行取模运算。

例如通过exit 命令指定返回值为300，经过取模运算，那么退出状态码就为44。


| Code  |              Description              |
| :---: | :-----------------------------------: |
|   0   |                命令成功结束                 |
|   1   |                通用未知错误                 |
|   2   |               误用shell命令               |
|  126  |                命令不可执行                 |
|  127  |                 没找到命令                 |
|  128  |                无效退出参数                 |
| 128+x |            Linux 信号x的严重错误             |
|  130  | Linux 信号2 的严重错误，即命令通过SIGINT（Ctrl＋Ｃ）终止 |
|  255  |                退出状态码越界                |


## 用法说明
exit命令用于退出当前shell，在shell脚本中可以终止当前脚本执行。

```
# 常用参数
# 格式：`exit n`
# 退出。设置退出码为`n`。（`Cause the shell to exit with a status of n.`）
# 格式：`exit`
# 退出。退出码不变，即为最后一个命令的退出码。（`If n is omitted, the exit status is that of the  last  command executed.` ）
```

## 引用
- [Shell中的特殊变量$?-查看上一条Shell命令的退出状态码（exit status）](http://blog.csdn.net/wlovh1989/article/details/51113488)
- [shell 程序 返回码 退出码](http://jackwxh.blog.51cto.com/2850597/827600)
- [Linux Shell 编程常见规则及退出状态码](http://www.cnblogs.com/qianzhilan/p/4391210.html)