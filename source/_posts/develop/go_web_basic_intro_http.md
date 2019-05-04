---
title: web开发之Go的http包
date: 2018-07-10 10:47:10
updated: 2019-04-29 21:29:14
categories:
- 网页开发

tags:
- Go
---
## 前言
梳理总结Go中的`net/http`包中内容，包括简单的HTTP协议介绍以及对应源码部分分析。

<!-- more -->

## HTTP协议
HTTP协议即超文本传输协议，基于TCP/IP协议模型，位于应用层。其目前版本演化经从`^HTTP/1.1(80端口)`到`HTTP/2.0(443端口)`。一次HTTP连接需要建立在TCP连接之上，采用**请求/响应**的方式工作，以报文（请求报文/响应报文）为传输载体。
net/http中实现的是`^HTTP/1.1`版本。

## request.go
### 请求报文的组成
主要由3部分组成：请求行（Request line）、请求头（Request header，由于与响应报文有通用部分，又将其再细分出一部分作为General header）及请求主体（Request body）。
#### 请求方法
HTTP协议定义了许多HTTP方法：常见的`GET`与`POST`方法。
##### GET方法用于获取资源
- GET提交的数据会放在URL之后，**以?分割URL和传输数据，参数之间以&相连**，如`EditPosts.aspx?name=test1&id=123456`。

```
GET /chn/yxsz/index.htm HTTP/1.1            //请求行：请求方法 + 请求路径 + 协议版本
User - Agent：Mozilla/5.0                   //请求头：header（字段名）：value（值）
                                            //请求体前空一行
//请求体部分内容，GET方法无请求体部分
```
##### POST方法用于传输数据
- POST方法是把提交的数据放在请求主体中，[即对应源码`Body io.ReadCloser`中](https://github.com/golang/go/blob/a27ede0ba9cd038582ea459f3c0e8419af4a2b88/src/net/http/request.go#L182)。

```
POST /test/demo_form.asp HTTP/1.1       //请求行
Host: w3schools.com                     //请求头

name1=value1&name2=value2               //将传输数据放于请求体
```

## response.go
### 响应报文组成
主要由3部分组成：状态行（Status line）、响应头（Response header）、响应体（Response body)。

#### 状态码
状态码从1xx到5xx，[各自有不同的意义](https://github.com/CyC2018/CS-Notes/blob/master/notes/HTTP.md#%E4%B8%89http-%E7%8A%B6%E6%80%81%E7%A0%81)。

### 实例
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

### HTTP协议与`Connection: keep-alive`
HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（面对无连接）。
**网页优化方面有一项措施是减少HTTP请求次数，就是把尽量多的css和js资源合并在一起，目的是尽量减少网页请求静态资源的次数，提高网页加载速度，同时减缓服务器的压力。**

## 源码理解部分
### 工作方式
以下均是net/http包中服务器端的几个概念：
```text
Request：用户请求的信息，用来解析用户的请求信息，包括post、get、cookie、url等信息
Response：服务器需要反馈给客户端的信息
Conn：用户的每次请求链接
Handler：处理请求和生成返回信息的处理逻辑
```

### net/http包执行流程
1. 创建Listen Socket, 监听指定的端口, 等待客户端请求到来。
2. Listen Socket接受客户端的请求, 得到Client Socket, 接下来通过Client Socket与客户端通信。
3. 处理客户端的请求, 首先从Client Socket读取HTTP请求的协议头, 如果是POST方法, 还可能要读取客户端提交的数据, 然后交给相应的handler处理请求, handler处理完毕准备好客户端需要的数据, 通过Client Socket写给客户端。

### net/http包其它内容
Go的net/http有两个核心功能：Conn、ServeMux
#### Conn
Go为了实现高并发和高性能, 使用了go routines来处理Conn的读写事件, 这样每个请求都能保持独立，相互不会阻塞，可以高效的响应网络事件。这是Go高效的保证。
#### ServeMux
主要是为了进行多路由处理。因为涉及到对映射的操作所以需要引入sync.RWLock对其进行读写控制。

## 结束语
Go中的http提供了很完善的实现。