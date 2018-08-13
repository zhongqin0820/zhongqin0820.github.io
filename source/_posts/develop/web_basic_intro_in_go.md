---
title: web开发之Go中的网络基础概念
date: 2018-07-11 15:47:10
categories:
- 网页开发

tags:
- Go
---
## 前言
&emsp;&emsp;梳理总结Go中关于网络的一些基础概念

<!-- more -->

## 传输协议
### socket
&emsp;&emsp;网络层的“ip地址”可以唯一标识网络中的主机，而传输层的“协议+端口”可以唯一标识主机中的应用程序（进程）。这样利用三元组（ip地址，协议，端口）

&emsp;&emsp;Socket有两种：`TCP Socket`和`UDP Socket`，TCP和UDP是协议，而要确定一个进程的需要三元组，需要IP地址和端口。

### WebSocket
&emsp;&emsp;**服务器可以实现主动的push消息，以简化以前ajax轮询的模式**
&emsp;&emsp;WebSocket采用了一些特殊的报头，使得浏览器和服务器只需要做一个握手的动作，就可以在浏览器和服务器之间建立一条连接通道。且此连接会保持在活动状态，你可以使用JavaScript来向连接写入或从中接收数据，就像在使用一个常规的TCP Socket一样。它解决了Web实时化的问题

### REST
&emsp;&emsp;REST是一种**架构风格**，汲取了WWW的成功经验：无状态，以资源为中心，充分利用HTTP协议和URI协议，提供统一的接口定义，使得它作为一种设计Web服务的方法而变得流行。在某种意义上，通过强调URI和HTTP等早期Internet标准，REST是对大型应用程序服务器时代之前的Web方式的回归。目前Go对于REST的支持还是很简单的，**通过实现自定义的路由规则，我们就可以为不同的method实现不同的handle，这样就实现了REST的架构**。

### RPC
&emsp;&emsp;`RPC（Remote Procedure Call Protocol）`——远程过程调用协议，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。

在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。

## Cookie与Session
### cookie
&emsp;&emsp;cookie是有时间限制的，根据生命期不同分成两种：会话cookie和持久cookie；**cookie位于客户端**
- 如果不设置过期时间，则表示这个cookie的生命周期为从创建到浏览器关闭为止，只要关闭浏览器窗口，cookie就消失了。这种生命期为浏览会话期的cookie被称为会话cookie。会话cookie一般不保存在硬盘上而是保存在内存里。
- 如果设置了过期时间(`setMaxAge(606024)`)，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie依然有效直到超过设定的过期时间。存储在硬盘上的cookie可以在不同的浏览器进程间共享，比如两个IE窗口。而对于保存在内存的cookie，不同的浏览器有不同的处理方式。

### session
&emsp;&emsp;session的基本原理是由服务器为每个会话维护一份信息数据，客户端和服务端依靠一个全局唯一的标识来访问这份数据，以达到交互的目的。当用户访问Web应用时，**服务端程序会随需要创建session**。

## 结束语
&emsp;&emsp;通过Go可以很方便的进行高级网络编程。
