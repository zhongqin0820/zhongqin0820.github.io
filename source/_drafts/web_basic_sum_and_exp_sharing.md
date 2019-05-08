---
title: 华为网挑资料分享及网络基础简单梳理
date: 2019-05-07 18:33:23
updated: 2019-05-07 18:33:23
categories:
- 计算机基础

tags:
- 学习总结
---
# 前言
由于本人使用的笔电无法支持`eNSP`软件，无缘决赛，仅分享华为网挑初赛参赛经验及复赛准备资料，顺便总结梳理计算机网络基础知识。

<!--more-->
# 华为网挑
属于华为四大挑战赛，当时是为了复习网络知识报名参加的，考查内容非常全面。通过初赛，可以学到很多的新概念。
不过由于复赛需要自备电脑，而由于本人笔电无法支持`eNSP`软件，导致最终无缘复赛。不过，通过练习`eNSP`组网，对网络知识的掌握有了一个更直观的感受，收获还是挺大的。

## 初赛
- 平常心参加，以学习的目的参加。可以学到:
	- Docker入门
	- OpenStack入门
	- 5G入门
	- MySQL数据库使用
	- ...

- 可以参考这个不错的[Github Repo](https://github.com/edisonleolhl/Huawei)，资料总结的很到位.

## 复赛
虽然无缘复赛，但是借机会也学习了一下`eNSP`的使用，分享几个实用资源。

- Bilibili: [华为ensp模拟器15个视频教程](https://www.bilibili.com/video/av4475686)
- eNSP软件 百度云盘链接提取码: [gedn](https://pan.baidu.com/s/1eM7o_KnEfgSQtIW0YpTy4Q)

# 计算机网络基础
重点放在面试常考知识，所以这里介绍TCP/IP四层网络结构。从上到下进行梳理。

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

#### dig命令
- 类似`traceroute`，显示DNS整个查询过程，以下为`dig www.baidu.com`的控制台输出。

```console
; <<>> DiG 9.10.6 <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY， status: NOERROR, id: 55761
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; ANSWER SECTION:
www.baidu.com.		595	IN	CNAME	www.a.shifen.com.
www.a.shifen.com.	315	IN	A	14.215.177.39
www.a.shifen.com.	315	IN	A	14.215.177.38

;; Query time: 3 msec
;; SERVER: 10.8.8.8#53(10.8.8.8)
;; WHEN: Tue May 07 22:21:28 CST 2019
;; MSG SIZE  rcvd: 101
```

### HTTP
- 超文本传输协议(Hyper Text Transport Protocol)
- 占用端口: 80
- 协议版本、方法、请求格式、响应格式
	- GET
	- POST
- 长连接、短连接
- cookie、session
- **状态码**：[HTTP/1.1: Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)
    - 重定向相关
        - 301：Permanently Moved
        - 302：Temporarily Moved
    - 客户端相关
        - 401：Unauthorized
    - 服务端相关
        - 500：Internal Server Error
        - 502：Bad Gateway

#### HTTP 2.0
- 服务器推送

#### HTTPS
- 占用端口: 443
- 较HTTP，添加了Secure Layer层(**安全**)。
	- 加密传播
		- SSL/TLS
	- 校验机制
	- 身份证书

##### 学习资料
- Bilibili [什么是HTTPS](https://www.bilibili.com/video/av22101017)

### 其它
- POP与SMTP
    - 端口：20/21
- FTP
    - [ ] [文件传输协议](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)
- RPC
    - [ ] [远程过程调用](https://zh.wikipedia.org/wiki/%E9%81%A0%E7%A8%8B%E9%81%8E%E7%A8%8B%E8%AA%BF%E7%94%A8)


## 传输层
### TCP
- 面向流，可靠的，全双工，拥塞控制，流量控制
- 3次握手，4次挥手
- TCP状态转换
    - [ ] [【Unix 网络编程】TCP状态转换图详解](https://blog.csdn.net/wenqian1991/article/details/40110703)

### UDP
- 面向数据报，尽力交付，简单

## 网络层/IP
### IP协议
- IPv4

- IPv6

- IP数据包

### ICMP/IGMP

#### ping

- `ping www.baidu.com -c 4`

```console
PING www.a.shifen.com (14.215.177.38): 56 data bytes
64 bytes from 14.215.177.38: icmp_seq=0 ttl=49 time=43.626 ms
64 bytes from 14.215.177.38: icmp_seq=1 ttl=49 time=18.597 ms
64 bytes from 14.215.177.38: icmp_seq=2 ttl=49 time=8.021 ms
64 bytes from 14.215.177.38: icmp_seq=3 ttl=49 time=7.723 ms

--- www.a.shifen.com ping statistics ---
4 packets transmitted， 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 7.723/19.492/43.626/14.606 ms
```

#### traceroute

- `traceroute www.baidu.com`

```console
traceroute: Warning: www.baidu.com has multiple addresses; using 14.215.177.38
traceroute to www.a.shifen.com (14.215.177.38), 64 hops max, 52 byte packets
 1  192.168.168.1 (192.168.168.1)  5.065 ms  1.543 ms  4.529 ms
 2  172.18.167.254 (172.18.167.254)  3.889 ms  9.503 ms  15.372 ms
 3  10.44.144.201 (10.44.144.201)  12.631 ms  1.449 ms  1.058 ms
 4  10.44.32.201 (10.44.32.201)  0.993 ms  1.013 ms  8.065 ms
 5  10.44.16.201 (10.44.16.201)  27.660 ms  3.002 ms  3.271 ms
 6  10.10.1.42 (10.10.1.42)  3.467 ms  4.370 ms  24.068 ms
 7  218.19.145.193 (218.19.145.193)  4.647 ms  3.056 ms  3.054 ms
 8  121.8.132.53 (121.8.132.53)  4.780 ms  6.345 ms  31.777 ms
 9  183.56.31.37 (183.56.31.37)  7.999 ms
    121.8.195.229 (121.8.195.229)  5.452 ms
    183.56.31.21 (183.56.31.21)  10.185 ms
10  149.176.37.59.broad.dg.gd.dynamic.163data.com.cn (59.37.176.149)  16.129 ms
    117.176.37.59.broad.dg.gd.dynamic.163data.com.cn (59.37.176.117)  13.930 ms  41.164 ms
11  219.135.202.1 (219.135.202.1)  8.531 ms
    183.56.34.5 (183.56.34.5)  18.750 ms
    219.135.202.1 (219.135.202.1)  11.771 ms
12  113.96.4.122 (113.96.4.122)  12.056 ms
    113.96.4.98 (113.96.4.98)  11.813 ms  22.077 ms
13  * * 94.96.135.219.broad.fs.gd.dynamic.163data.com.cn (219.135.96.94)  12.481 ms
14  14.215.32.126 (14.215.32.126)  36.516 ms
    14.215.32.130 (14.215.32.130)  36.093 ms
    14.29.117.238 (14.29.117.238)  25.962 ms
```

### 其它
- ARP

- NAT

## 数据链路层
- CSMA/CD
- PPP

### 重要概念
- LAN/VLAN
- 交换机
- 网桥
- 网卡

# 学习资料
- Bilibili: [泰克教育之2018年华为ICT大赛赛前辅导——网络安全](https://www.bilibili.com/video/av23543455)
- Bilibili: [计算机网络常见协议讲解](https://www.bilibili.com/video/av22861093)
- Bilibili: [什么是HTTPS](https://www.bilibili.com/video/av22101017)