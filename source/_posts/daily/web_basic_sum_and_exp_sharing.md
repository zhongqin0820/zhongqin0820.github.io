---
title: 知识梳理：华为网挑资料分享及网络基础相关梳理
date: 2019-05-07 18:33:23
updated: 2019-06-08 20:12:33
categories:
- 计算机基础

tags:
- 学习总结
---
# 前言
由于本人使用的笔电无法支持`eNSP`软件，无缘复赛（已通过初赛），仅分享华为网挑初赛参赛经验及复赛准备资料，顺便总结梳理计算机网络基础知识。

<!--more-->
# 华为网挑
属于华为四大挑战赛，当时是为了复习网络知识报名参加的，考查内容非常全面。通过初赛，可以学到很多的新概念。
不过由于复赛需要自备电脑，而由于本人笔电无法支持`eNSP`软件，导致最终无缘复赛。不过，通过练习`eNSP`组网，对网络知识的掌握有了一个更直观的感受，收获还是挺大的。

## 初赛
- 录取率很高（40%），可以平常心参加，以学习的目的参加。可以学到:
    - Docker入门
    - OpenStack入门
    - 5G入门
    - MySQL数据库使用
    - ...
- 如果时间不够，可以参考这个不错的[Github Repo](https://github.com/edisonleolhl/Huawei)，资料总结的很到位.

## 复赛
虽然无缘复赛，但是借机会也学习了一下`eNSP`的使用，分享几个实用资源。

- [华为ensp模拟器15个视频教程](https://www.bilibili.com/video/av4475686)
- **eNSP软件** 百度云盘链接提取码: [gedn](https://pan.baidu.com/s/1eM7o_KnEfgSQtIW0YpTy4Q)

# 计算机网络基础
重点放在面试常考知识，所以这里以TCP/IP四层网络结构展开介绍。从高层向下进行梳理。

## 应用层
### DHCP
- 动态主机配置协议(Dynamic Host Configuration Protocol)。
- 当我们将客户主机ip地址设置为动态获取方式时，DHCP服务器就会根据DHCP协议给客户端分配IP，使得客户机能够利用这个IP上网。
- 利用的网络层的路由协议。配置内容通常包括：**[本机IP，局域网掩码，网关 IP，DNS IP]**

#### 路由协议
两台主机能够`ping`通的前提条件是需要知道双方的IP。由交换机或路由器对其进行路由转发。
- 直连(位于同一局域网，通常可以直接设置同一网段的IP)，直接`ping`目的主机IP即可。
- 不直连: 需要配置[本机IP，局域网掩码，网关 IP，DNS IP]
    - 静态路由: 手动设置
    - 动态路由: 使用路由协议(网络层协议)，自动配置
        - 内部网关
            - OSPF
            - RIP
        - 外部网关
            - BGP

### DNS
- 域名系统(Domain Name System)或域名解析协议。
- 若DNS的IP通常是动态的，需要利用DHCP协议对其进行动态分配。常见的是`8.8.8.8`等。
- 主要提供域名解析服务、别名、负载均衡等
    - A字段: 指定主机名（或域名）对应的IPv4地址记录
    - CNAME字段: 别名(例如: www.baidu.com. 的cname就是www.a.shifen.com.)
- 使用UDP作为传输协议
- 可以提供HTTP，SMTP，FTP等协议使用
- 占用端口: 53
- dig命令：功能类似`traceroute`，显示DNS整个查询过程。

### HTTP
- 超文本传输协议(Hyper Text Transport Protocol)
- 占用端口: 80
- 协议版本、方法、请求格式、响应格式
    - GET
    - POST
- 传输层使用TCP需要建立连接：长连接、短连接
- 本身无状态，需要使用cookie、session维持
- **状态码**：
    - [HTTP/1.1: Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)
    - 重定向相关
        - 301：Permanently Moved
        - 302：Temporarily Moved
    - 客户端相关
        - 401：Unauthorized
    - 服务端相关
        - 500：Internal Server Error
        - 502：Bad Gateway

#### HTTP 2.0
- 二进制分帧层
- 服务器推送功能：解决每个资源都要建立一次连接，请求一个资源，推送相关的资源
- 首部压缩

#### HTTPS
- 推荐阅读材料：[揭秘HTTPS的"秘密"](https://sylvanassun.github.io/2017/08/06/2017-08-06-DigestHttps/)
- 参考学习资料：[什么是HTTPS](https://www.bilibili.com/video/av22101017)
- 占用端口: 443
- 较HTTP，添加了Secure Layer层(**安全**)。
    - 加密传播
        - SSL/TLS
    - 校验机制
    - 身份证书
- 加密过程(混合加密)
    - 建立通信过程中（3次握手），使用`非对称加密算法`，对建立连接后的数据进行`对称加密`的密钥，进行加密的技术。
    - 使用了权威的CA`认证中心`防止发送加密`对称密钥`被中间人劫持或伪造。

### 其它
- SMTP、POP3、IMAP
    - [ ] [电子邮件协议](https://github.com/CyC2018/CS-Notes/blob/master/notes/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%20-%20%E5%BA%94%E7%94%A8%E5%B1%82.md#%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E5%8D%8F%E8%AE%AE)

- FTP
    - [ ] [文件传输协议](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)

### RPC 与 WebSocket
- RPC
    - [ ] 《Go语言高级编程(Advanced Go Programming)》第四章：RPC与Protobuf
    - [ ] [远程过程调用](https://zh.wikipedia.org/wiki/%E9%81%A0%E7%A8%8B%E9%81%8E%E7%A8%8B%E8%AA%BF%E7%94%A8)

- WebSocket：HTML5中集成，依赖HTTP建立连接，使用自有帧协议传输数据，服务端推送
    - [github.com/gorilla/websocket](https://github.com/gorilla/websocket)
    - 如何理解HTTP 2.0提供的服务端推送？
        - [HTTP 2.0与WebSocket的关系是怎么样的？](https://www.cnblogs.com/shiyunfront/p/7338306.html)
        - [捋一捋http、https、http2、WebSocket、SSE之间的关系](https://blog.csdn.net/m0_37886783/article/details/79882014)
    - [ ] [RFC6455](https://tools.ietf.org/html/rfc6455)

## 传输层
### TCP
- **面向流**，可靠的，全双工，流量控制，拥塞控制
    - 面向流：指传输的数据格式
    - 可靠的：设置序列号以及超时重传机制，确保数据一定被发送并被接收
    - 全双工：连接的任何一方都可以发起或断开连接，只有双方的连接都断开连接才真正结束。连接同理。
    - 流量控制：接收方设置一个接收窗口大小给发送方，从而控制发送方的发送速率。特别注意，**当`窗口=0`的情况，发送方无法发送数据**
    - 拥塞控制：存在的意义是为了降低链路上的拥塞情况。发送方维护一个拥塞窗口，通过一定的机制达到控制链路上的网络情况的目的。
        - 慢开始 & 拥塞避免：设置窗口大小1，2，4，2^n；出现超时重传现象时，对其减半。
        - 快重传 & 快恢复
- 3次握手，4次挥手
- [server底层过程（涉及Linux网络编程相关内容）](https://jiajunhuang.com/articles/2019_11_01-network_programming.md.html)：`socket` -> `bind(ip:port)` -> `listen`（维持一个`syn`队列） -> `accept`（维持一个`accept`队列） -> `send/write` -> `close`（四次挥手发生）
- client底层过程：`socket` -> `connect`（三次握手发生） -> `read/recv` -> `close`（四次挥手发生）
- TCP状态转换：[【Unix 网络编程】TCP状态转换图详解](https://blog.csdn.net/wenqian1991/article/details/40110703)
- 黏包与拆包：[黏包问题的成因与解决方案](https://www.cnblogs.com/kakawith/p/8378425.html)

### UDP
- 面向数据报，尽力交付，简单。
- UDP不提供重传机制，占用资源小，处理效率高。
- 一些时延敏感的流量，如语音，视频等，通常使用UDP作为传输层协议。
- 丢包：[UDP主要丢包原因及具体问题分析](https://www.cnblogs.com/Zhaols/p/6105926.html)

## 网络层/IP
### IP协议
- IPv4
- IPv6
- IP数据包

### ICMP/IGMP
- `ping`
- `traceroute`

### 其它
- NAT（Network Address Translation）：网关
    - 地址转换映射表：(`<公网IP, 端口号>` : `<私网IP, 端口号>`)
    - 问题：NAT穿透

## 数据链路层
- 碰撞检验
    - CSMA/CD
    - PPP
- 地址解析
    - ARP（Address Resolution Protocol）： IP -> MAC地址
    - RARP（Reverse Address Resolution Protocol）：MAC地址 -> IP

### 基础概念
- 网卡
    - 系统安装了一个网卡之后会为其生成一个网络设备实例，如eth0。
    - 同时，系统也可以通过虚拟化技术生成一个虚拟化网络设备。通过虚拟化设备的组合实现复杂功能和网络拓扑。
        - Veth：成对出现
        - Bridge：用于桥接，相当于现实世界的交换机
        - 802.1.q VLAN device
        - TAP
- 网桥/虚拟网桥
- LAN/VLAN
- 交换机：
    - 按照兼容的协议栈分为：二层、三层（路由器）

# 其它学习资料
- 《TCP/IP Vol.1》
- [泰克教育之2018年华为ICT大赛赛前辅导——网络安全](https://www.bilibili.com/video/av23543455)
- [计算机网络常见协议讲解](https://www.bilibili.com/video/av22861093)
- [什么是HTTPS](https://www.bilibili.com/video/av22101017)
- [网络编程所需要熟悉的那些函数](https://jiajunhuang.com/articles/2019_11_01-network_programming.md.html)

# Changelog
- 2019/06/08：整理部分内容，重新划分，这里只作简述以及资料索引。
- 2019/10/04：完善WebSocket以及NAT和TCP部分的内容
- 2019/11/01：添加[网络编程所需要熟悉的那些函数](https://jiajunhuang.com/articles/2019_11_01-network_programming.md.html)
