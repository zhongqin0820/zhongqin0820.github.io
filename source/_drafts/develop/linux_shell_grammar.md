---
title: Linux之Shell脚本入门
date: 2019-05-08 12:20:12
updated: 2019-05-08 12:20:12
categories:
- 网页开发

tags:
- Linux
---
# 前言
系统梳理shell脚本入门需知需会基础概念和语法。

<!-- more -->
# 符号
## 引号
### `"`
- 双引号內`$`，`` \ ``，`` ` ``具有特殊含义
- `$`：变量调用
- `` \ ``：转义

### `'`
- 所有字符串都没有特殊含义

### `` ` ``
- 使用反引号內命令的执行结果作**命令替换**

## 括号

### `()`
- `()`：定义一个数组变量
- `(())`：高级数学运算

### `[]`
- `[]`：条件测试
- `[[]]`：条件测试，其中可以使用正则表达式

### `{}`
- 语句块

## `$`
- `$()`：命令替换
- `$(())`：数学运算
- `$[]`：数学运算
- `${}`：**变量替换**

# 流程控制
- `break`：跳出流程控制
- `continue`：跳出本次循环，执行下一次循环

## 判断
### `if`语句
```sh
if condition; then
# do something
elif condition; then
# do something elif
else
# do something else
fi
```

### `case`语句
```sh
case c in
case1)
# do something
;;
*)
# do default thing
;;
esac
```

### `select`语句
```sh
select item in array; do
# wait for input from user
break
done
# use item below
```

## 循环
### `for`语句
```sh
for item in array; do
# do something
done
```

### `while`语句
```
while condition; do
# do something
done
```

# 函数定义

```sh
# !/bin/bash

# A simple bash script to move up to desired directory level directly

function jump()
{
    # original value of Internal Field Separator
    OLDIFS=$IFS

    # setting field separator to "/"
    IFS=/

    # converting working path into array of directories in path
    # eg. /my/path/is/like/this
    # into [, my, path, is, like, this]
    path_arr=($PWD)

    # setting IFS to original value
    IFS=$OLDIFS

    local pos=-1

    # ${path_arr[@]} gives all the values in path_arr
    for dir in "${path_arr[@]}"
    do
        # find the number of directories to move up to
        # reach at target directory
        pos=$[$pos+1]
        if [ "$1" = "$dir" ];then

            # length of the path_arr
            dir_in_path=${#path_arr[@]}

            #current working directory
            cwd=$PWD
            limit=$[$dir_in_path-$pos-1]
            for ((i=0; i<limit; i++))
            do
                cwd=$cwd/..
            done
            cd $cwd
            break
        fi
    done
}
```

# 其它
- 开头：`#!/bin/sh`指定用来执行该文件的程序
- 注释：`#`后为注释
- 变量：所有的变量都由字符串组成，定义变量`=`左右不可以有空格
- `exit`：退出程序，后跟不同状态码具有不同意义，参考[Linux之Shell命令的状态码](https://cvblogs.cn/2017/11/08/develop/linux_Shell_status_code/)
- 执行脚本：需要确保具有执行权限（`chmod +x filename.sh`），使用`filename.sh`执行

# 学习资料
- [Introduction to Linux Shell and Shell Scripting](https://www.geeksforgeeks.org/introduction-linux-shell-shell-scripting/)