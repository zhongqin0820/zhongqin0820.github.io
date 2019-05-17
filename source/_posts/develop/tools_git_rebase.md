---
title: git文件太大无法上传解决办法
date: 2019-05-11 17:16:12
updated: 2019-05-11 17:16:12
categories:
- 其它
tags:
- 修复问题
---
# 前言
不小心将数据集备份到了记录里，由于文件过大（允许的文件大小~50M），无法被上传到GitHub。一顿骚操作后最终使用`git rebase`调整提交记录到添加时，最终上传成功。

<!--more-->
# 解决方案
前两个居然没有起作用，最后加上第三个命令才最终从工作空间删除了记录。
```sh
# Remove files from the working tree and from the index
git rm path/to/the_file

# Rewrite branches
git filter-branch --tree-filter 'git rm path/to/the_file' --all

# Reapply commits on top of another base tip
git rebase -i HEAD~n
```

# 其它命令学习
由于使用了`git rebase`，所以需要对各个提交进行修改。
## vim中的搜索替换
```sh
# 命令格式
:{作用范围}s/{目标}/{替换}/{替换标志}

# 作用模式
## 当前行
:s/搜索词/替换词

## 全文
:%s/搜索词/替换词

## 选区模式
### normal 模式按 v 进入 visual模式，输入:，出现
:<','>
### 补充命令
:<','>s/搜索词/替换词
#### 指定行数:2~5行
:2,5s/搜索词/替换词
#### 当前行.与接下来2行
:.,+2s/搜索词/替换词

# 替换标志
## g表示全局
## i(大小写不敏感)，I(大小写敏感)
## c需要确认：y(es),n(o),a(ll),q(uit)
```

# 参考资料
- [在 Vim 中优雅地查找和替换](https://harttle.land/2016/08/08/vim-search-in-file.html)