---
title: vim的mru插件报错解决方案
date: 2018-01-10 16:55:24
updated: 2018-01-10 16:55:24
categories:
- 其它
tags:
- Linux
---
# 前言
使用oh-my-zsh的快捷配置过程中，出现每次打开vim都报:
```bash
Error detected while processing function <SNR>37_MRU_LoadList:
line 4:
...
```
找了很久终于在stack overflow上找到[解决方案](https://stackoverflow.com/questions/15397567/vim-error-detected-while-processing-function-snr37-mru-loadlist)，备记一下。

<!-- more -->
# 解决方案
因为使用`mru.vim - Plugin to manage Most Recently Used (MRU) files`插件不当引起的。删除存储文件后就可以了。
使用如下命令
```bash
:call delete(g:MRU_File)
```
就恢复正常了...