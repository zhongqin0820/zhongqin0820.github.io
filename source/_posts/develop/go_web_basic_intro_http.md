---
title: web开发之Go的http包
date: 2018-07-10 10:47:10
updated: 2018-07-10 10:47:10
categories:
- 网页开发

tags:
- Go
---
## 前言
&emsp;&emsp;梳理总结Go中的http包

<!-- more -->

## http请求包
### 组成
&emsp;&emsp;Request包分为3部分，第一部分叫Request line（请求行）, 第二部分叫Request header（请求头）,第三部分是body（主体）。

### `get()` 和 `post()`方法的区别
- GET提交的数据会放在URL之后，**以?分割URL和传输数据，参数之间以&相连**，如EditPosts.aspx?name=test1&id=123456。
- POST方法是把提交的数据放在HTTP包的body中。

## http响应包
```text
HTTP/1.1 200 OK						//状态行:HTTP协议版本号， 状态码， 状态消息
Server: nginx/1.0.8					//服务器使用的WEB软件名及版本
Date:Date: Tue, 30 Oct 2012 04:14:25 GMT		//发送时间
Content-Type: text/html				//服务器发送信息的类型
Transfer-Encoding: chunked			//表示发送HTTP包是分段发的
Connection: keep-alive				//保持连接状态
Content-Length: 90					//主体内容长度
//空行 用来分割消息头和主体
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"... //消息体
```

## http协议与`Connection: keep-alive`
&emsp;&emsp;HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（面对无连接）。
&emsp;&emsp;**网页优化方面有一项措施是减少HTTP请求次数，就是把尽量多的css和js资源合并在一起，目的是尽量减少网页请求静态资源的次数，提高网页加载速度，同时减缓服务器的压力。**

### 工作方式
&emsp;&emsp;以下均是服务器端的几个概念：
```text
Request：用户请求的信息，用来解析用户的请求信息，包括post、get、cookie、url等信息
Response：服务器需要反馈给客户端的信息
Conn：用户的每次请求链接
Handler：处理请求和生成返回信息的处理逻辑
```

### http包执行流程
1. 创建Listen Socket, 监听指定的端口, 等待客户端请求到来。
2. Listen Socket接受客户端的请求, 得到Client Socket, 接下来通过Client Socket与客户端通信。
3. 处理客户端的请求, 首先从Client Socket读取HTTP请求的协议头, 如果是POST方法, 还可能要读取客户端提交的数据, 然后交给相应的handler处理请求, handler处理完毕准备好客户端需要的数据, 通过Client Socket写给客户端。

## http包详解
Go的http有两个核心功能：Conn、ServeMux
### Conn
Go为了实现高并发和高性能, 使用了goroutines来处理Conn的读写事件, 这样每个请求都能保持独立，相互不会阻塞，可以高效的响应网络事件。这是Go高效的保证。

### demo
r.Form里面包含了所有请求的参数，比如URL中query-string、POST的数据、PUT的数据，所以当你在URL中的query-string字段和POST冲突时，会保存成一个slice，里面存储了多个值，Go官方文档中说在接下来的版本里面将会把POST、GET这些数据分离开来。

### request.Form
&emsp;&emsp;`request.Form`是一个`url.Values`类型，里面存储的是对应的类似`key=value`的信息

&emsp;&emsp;开发Web的一个原则就是，**不能信任用户输入的任何信息**，所以验证和过滤用户的输入信息就变得非常重要，我们经常会在微博、新闻中听到某某网站被入侵了，存在什么漏洞，这些大多是因为网站对于用户输入的信息没有做严格的验证引起的，所以为了编写出安全可靠的Web程序，[验证表单输入的意义重大](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/04.2.md)。

## 结束语
&emsp;&emsp;Go中的http提供了很完善的实现。