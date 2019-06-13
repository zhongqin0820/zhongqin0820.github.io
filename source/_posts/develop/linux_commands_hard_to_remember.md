---
title: 一些容易被我忘记的控制台命令以及快捷键
date: 2017-11-05 19:47:26
updated: 2019-06-08 21:36:26
categories:
- 网页开发
tags:
- 开发总结
---
## 前言
总结一些容易被我忘记的控制台命令，以及面试过程中常问的几个操作/命令。

<!-- more -->
## 面试常问
### 进程相关命令
```shell
top         #display and update sorted information about processes
ps  [-o]    #process status
pstree      #a tree of precesses
kill        #terminate or signal a process
```

### 系统资源查看
系统资源使用情况包括：CPU、内存、磁盘空间、设备...

- [How to Monitor Linux Systems Performance with iostat command](https://www.linuxtechi.com/monitor-linux-systems-performance-iostat-command/)

```shell
# CPU
iostat [-cdx]     #report I/O statistics: cpu,disk,i/o statistics

# Memory related
free -h           #display free and used memory

# 查看磁盘空间
du                #display disk usage statistics
df                #display free disk space

# 设备
lspci             #list all pci devices

```

### 网络相关的命令
```shell
# 检查网络连通情况
ping        #send ICMP ECHO_REQUEST packets to network hosts
traceroute  #print the route packets take to network host
dig         #DNS查询情况

# 检查网卡状况
netstat  [-an]   #show network status
lsof -i:22       #查看指定端口

# 检查网络配置
iptables    #filtering packets and NAT
ifconfig    #configure network interface parameters
ipconfig    #view and control IP configuration state
```

### 文件查看相关命令
```shell
# 内容查看
cat
more
less [shift+f]
head
tail [-f]

# 查看文件信息
basename    #文件名
dirname     #路径名

# 查找
grep        #用于查找文件中的匹配记录，''中可以包含正则表达式
find        #用于查找文件或文件夹，可以指定要查找的目录，默认当前
```

### 重定向相关

| 方法 | 代码 | 操作符 |
| :--------: | :--------: | :--------: |
| stdin | `0` | `<` 或 `<<` |
| stdout | `1` | `>` 或 `>>` |
| stderr | `2` | `2>` 或 `2>>` |

## 一些容易被我忘记的控制台命令

```shell
# 文件/目录 基本操作
cd -                #返回上一个访问的目录
cp 文件名 目标目录    #将文件拷贝到目标目录下

# 文件查看
cat 文件名(|less)    #在终端下查看文件，可以利用管道技术，配合more/less使用
more                #按页来查看文件的内容(只能往后)
less                #往前往后翻看文件

# 中断操作
control+d           #中断a.out运行

# 其它
wc [-lwc] 文件名     #统计文件的行数、字数、字节数
awk                 #pattern-directed scanning and processing language
man [-k]            #查看命令
control + insert    #ctrl+C
shift + insert      #ctrl+v

# console 编辑
control+u           #删除从开头到光标文本
control+k           #删除从光标到尾部文本
control+w

# 前后台运行程序
control + z         #将当前进程放置后台运行
fg + 后台进程序号     #恢复对应后台进程到前台
```

### 正则表达式相关
| 符号 | 说明 | 举例 |
| :--------: | :--------: | -------- |
| `.` | 匹配一个字符 |  |
| `+` | 匹配一个或多个字符 |  |
| `*` | 匹配0个或多个字符 | 使用`{m,n}`或`{m,}`或`{n}`指定字符个数 |
| `?` | 匹配0个或一个字符 |  |
| `^` | 字符串开头，**在字符集合中用作取非** | `^[]`或`[^a-z]`(等价于`\D`) |
| `$` | 字符串结尾 |  |
| `[]` | 定义一个字符集合 |`^[]`，`[0-9]`或`[a-zA-Z]`|
| `` \ `` | 后跟元字符，有特殊意义 | `\d`与`\D`，`\w`与`\W`，`\s`与`\S` |
| `()` | 嵌套子表达式 |  |

## 一些浏览器快捷键
```commands
cmd + 鼠标点击       #在新标签页打开点击的链接
```

## 参考资料
- [Linux中more和less命令用法](https://www.cnblogs.com/aijianshi/p/5750911.html)

## Changelog
- 2019/06/08：修改格式内容，添加端口参看命令`lsof -i:22`及`dig`命令。

