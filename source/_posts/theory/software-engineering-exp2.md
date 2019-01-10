---
title: 孟宁《软件工程C实战》实验报告2
date: 2017-09-24 21:45:35
updated: 2017-09-24 21:45:35
categories:
- 软件工程

tags:
- 实验报告
---
### 前言
&emsp;&emsp;本篇博文是孟宁老师的[Mooc网课](https://mooc.study.163.com/course/USTC-1000002006#/info)的实验报告内容。实验主要是关于字符串的处理，需要对字符串函数熟悉，可以减少自己造轮子。<u>非常有意义的实验！</u>

<!--more-->

### 实验内容
&emsp;&emsp;设计实现一个迷你命令行解析小程序。
<br/>![实验效果](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/2017-09-23%2023-47-38%E7%9A%84%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

### 实验代码
&emsp;&emsp;重新看自己的代码，发现写的还是很糟糕的。例如，函数没有采用驼峰式等。等到下回有机会改进把。
```c
/*************************************************************************
    > File Name: MiniCmd.c
    > Author: Qin Zhong
    > Mail: zhongqin0820@163.com
    > Created Time: 2017年09月16日 星期六 16时01分33秒
 ************************************************************************/

#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#define MAXN 128
void PrintHelp()
{
    printf("help cmd list\n");
    printf("------------------------------------------------------\n");
    printf("help for help list\n");
    printf("quit for exit this program\n\n");
}
void ErrorCmdPrint()
{
    printf("error cmd\n\n");
    PrintHelp();
}
int main()
{
    char cmd[MAXN];
    memset(cmd,0,sizeof(char)*MAXN);
    while(1)
    {
        gets(cmd);
        if(strcmp(cmd,"help") == 0)
        {
        	PrintHelp();
        }
        else if(strcmp(cmd,"quit") == 0)
        {
        	printf("bye~\n");
        	exit(0);
        }
        else if(strcmp(cmd,"clear") == 0)
        {
            system("cls");
        }
        else if(strstr(cmd,"hello") != NULL)
        {
        	printf("你好,%s\n",cmd);
        }
        else if(strcmp(cmd,"null") == 0)
        {
        	printf("null\n");
        }
        else if(strstr(cmd,"whois") != NULL)
        {
        	printf("me\n");
        }
        else
        {
            ErrorCmdPrint();
        }
    }
    return 0;
}
```
### 课程链接
[软件工程（C编码实践篇）](https://mooc.study.163.com/course/USTC-1000002006#/info)
### 结束语
&emsp;&emsp;其实上周就开始做了，但是还是太懒了..最近又忙，看了第三课老师的讲解，发现自己还是想简单了。另外，代码到底用tab还是空格对于程序美观真的很重要..