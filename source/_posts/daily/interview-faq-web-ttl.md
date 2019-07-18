---
title: 知识梳理：网络传输层相关
date: 2019-06-08 16:55:11
updated: 2019-06-08 20:31:18
categories:
- 计算机基础

tags:
- 学习总结
---
# 前言
收录整理计算机网络传输层的高频面试题。包括：TCP与UDP的区别与联系，TCP的可靠性如何保证，TCP的三次握手四次握手说明及分析（为什么要三次握手及四次挥手），同时涉及Linux中与网络相关的命令简单介绍（`netstat`，`iptables`，`ifconfig`，`ping`，`traceroute`，`dig`等）以及包括与命令相关的高频问题，如：如何查看端口占用情况，如何解决连接过程中本地太多`TIME_WAIT`或`CLOSE_WAIT`或`SYN_RCVD`的情况。

<!-- more -->
# TCP与UDP的区别与联系
|区别|TCP|UDP|
|:---:|:---:|:---:|
|协议头所占字节长度|20个|8个|
|数据格式|字节流|数据报|
|建立连接|需要|不需要|
|通信双方数量关系|全双工（1:1）|n:m|
|可靠性保证|保证：序列号，滑动窗口，超时重传，流量控制，拥塞控制|不保证：尽力传输，UDP不提供重传机制，占用资源小，处理效率高。|
|主要应用场景|HTTP协议等|一些时延敏感的流量，如语音，视频等|

# TCP建立/断开连接过程
描述过程中需要说明如下字段：
- 标志位（SYN，ACK，FIN）与序列号（seq，ack）
- 状态
    - 建立连接：CLOSED，LISTEN，SYN_SEND，SYN_RCVD，ESTABLISHED
    - 断开连接：FIN_1(FIN_WAIT_1)，CLOSE_WAIT，FIN_2(FIN_WAIT_2)；LAST_ACK，TIME_WAIT；CLOSED

## 状态描述

![TCP状态转换图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/tcp_states.png)

|状态|描述|
|:----:|:----:|
|CLOSED|无连接是活动的或正在进行|
|LISTEN|服务器在等待进入呼叫|
|SYN_RECV|一个连接请求已经到达，等待确认|
|SYN_SENT|应用已经开始，打开一个连接|
|ESTABLISHED|正常数据传输状态|
|CLOSING|两边同时尝试关闭|
|FIN_WAIT1|应用说它已经完成|
|FIN_WAIT2|另一边已同意释放|
|CLOSE_WAIT|等待所有分组死掉|
|LAST_ACK|等待所有分组死掉|
|TIME_WAIT|另一边已初始化一个释放|

## 三次握手
### 描述
- 底层过程：sockets -> bind(ip:port) -> listen -> accept -> send/recv
- 第1次握手
    - A发送【SYN=1，seq=x】，并由`CLOSED`进入`SYN_SEND`状态
- 第2次握手
    - B**通过服务端口（监听端口，如应用服务器的80端口）**收到A的报文，由`CLOSED`状态进入`LISTEN`状态；B发送【SYN=1，ACK=1，ack=x+1，seq=y】，并由`LISTEN`状态进入`SYN_RECV`状态
- 第3次握手：
    - A发送【ACK=1，ack=y+1，seq=x+1】，并进入`ESTABLISHED`状态。**当B收到A的数据包后，也进入`ESTABLISHED`状态**

### 问题
- 为什么不是两次握手：第三次握手存在的意义是为了避免由于网络时延引起的A超时重传，旧的连接请求被B接收导致B建立两个连接的情况。
- 太多SYN_RCVD的分析及解决方法：后文提到，首先描述为什么出现这个状态，再分析可能出现大量该状态的原因并对应的解决办法。

## 四次挥手
### 描述
- 第1次挥手
    - A发送【FIN=1，ACK=1，seq=u，ack=z】，并进入`FIN_WAIT_1`状态
- 第2次挥手
    - B收到A的数据包，发送【ACK=1，ack=u+1】，并进入`CLOSE_WAIT`状态，A收到后进入`FIN_WAIT_2`状态。
- 第3次挥手
    - B在发送完`x`个序列数据后，发送断开连接请求【FIN=1，ACK=1，seq=z+`x`，ack=u+1】，并进入`LAST_ACK`状态。
- 第4次挥手
    - A收到后发送【ACK=1，ack=z+`x`+1】并进入`TIME_WAIT`状态后需等待2MSL(240s)的时间进一步确认连接确实断开才将连接状态置为`CLOSED`状态，而B在收到A的确认后即进入`CLOSED`状态。

### 问题
- 为什么不是两次挥手：由于连接是双工的，己方断开连接只是说明己方无需再向对方发送数据，而对方还可能继续发送数据，所以还不能挂断连接
- 为什么不是三次挥手：由于**网络时延**的存在，因此本地为了确保对等方的数据都被接收及确认完毕，并且保证它的关闭连接请求的确认被收到而进行的等待。否则可能造成数据传输不完全，以及对等方的连接可能无法释放的问题。
- 太多CLOSE_WAIT及TIME_WAIT的分析及解决方法：后文提到，首先描述为什么出现这个状态，再分析可能出现大量该状态的原因并对应的解决办法。

# TCP的可靠性如何保证
- [参考资料](https://github.com/CyC2018/CS-Notes/blob/master/notes/计算机网络%20-%20传输层.md)

## 可靠性与网络时延及超时重传
现实环境中，数据包传输过程中因为各种原因，存在网络时延，为了保证协议的可靠性（发送方发送的数据完整且能够被接收方完全接收不丢包），TCP协议设置序列号以及滑动窗口的方式对数据包进行按序按能力接收。同时使用超时重传机制，确保数据一定被发送并被接收

## 流量控制
接收方设置一个接收窗口大小给发送方，从而控制发送方的发送速率。特别注意，当窗口=0的情况，发送方无法发送数据

## 拥塞控制
存在的意义是为了降低链路上的拥塞情况。发送方维护一个拥塞窗口，通过一定的机制达到控制链路上的网络情况的目的。
- 慢开始 与 拥塞避免：设置窗口大小1，2，4，2^n；出现超时重传现象时，对其减半。
- 快重传 与 快恢复

# Linux中命令
## netstat
通过`netstat`查看系统当前系统网络状态信息，包括端口，连接情况等。
```sh
# 参看不同状态的TCP连接数
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
# 输出
# LAST_ACK 1
# CLOSE_WAIT 44
# ESTABLISHED 31
# FIN_WAIT_2 2
# TIME_WAIT 4
```

## iptables

## ifconfig

## ping

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

## traceroute

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
    ...
```

## dig命令

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
;www.baidu.com.            IN    A

;; ANSWER SECTION:
www.baidu.com.        595    IN    CNAME    www.a.shifen.com.
www.a.shifen.com.    315    IN    A    14.215.177.39
www.a.shifen.com.    315    IN    A    14.215.177.38

;; Query time: 3 msec
;; SERVER: 10.8.8.8#53(10.8.8.8)
;; WHEN: Tue May 07 22:21:28 CST 2019
;; MSG SIZE  rcvd: 101
```

# 如何查看端口占用情况
- 参考资料：[使用netstat、lsof查看端口占用情况](http://lazybios.com/2015/03/netstat-notes/)

```sh
netstat -an | grep 22

lsof -i:22
```

# 解决本地太多TIME_WAIT或CLOSE_WAIT或SYN_RCVD
针对TCP创建的每个连接都会有一个对应的socket连接文件描述符。过多的连接如果无法回收会导致系统资源耗尽，无连接可用的情况。

## TIME_WAIT
### 产生原因
`TIME_WAIT`是TCP四次挥手中的主动发起断开连接请求的发起方收到对等方[FIN，ACK]包并发送ACK包后的状态，`TIME_WAIT`的产生及为什么需要四次握手的原因：由于**网络时延**的存在，因此本地为了确保对等方的数据都被接收及确认完毕，并且保证它的关闭连接请求的确认被收到而进行的等待。否则可能造成数据传输不完全，以及对等方的连接可能无法释放的问题。
它主要通过保持这个状态`2MSL`（max segment lifetime）时间之后，彻底关闭回收资源。

由于需要保持2MSL=240s（RFC 793中规定MSL为2分钟）的时间，因此**太多的`TIME_WAIT`**通常由于HTTP协议中**短连接引起**。

> 注意，MSL要大于等于TTL（Time To Live由源主机设置初始值但不是存的具体时间，而是存储了一个IP数据报可以经过的最大路由数，每经过一个处理他的路由器此值就减1，当此值为0则数据报将被丢弃，同时发送ICMP报文通知源主机。）

### 如何避免分析
- 修改MSL时间
- 重用连接
- 快速回收

### 具体操作
通过修改`/etc/sysctl.conf`文件，服务器能够快速回收和重用 `TIME_WAIT` 的资源，执行`/sbin/sysctl -p`让参数生效。
```conf
net.ipv4.tcp_tw_reuse = 1       #表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭
net.ipv4.tcp_tw_recycle = 1     #表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭
net.ipv4.tcp_fin_timeout=30     #表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间
net.ipv4.tcp_syncookies = 1     #表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭
```

## CLOSE_WAIT
### 产生原因
CLOSE_WAIT是TCP四次挥手的中间状态，是服务端口收到FIN包并发送ACK包后所处的状态。导致 CLOSE_WAIT 大量存在的原因：就是在对方关闭连接之后服务器程序自己没有进一步发出 ACK 信号。换句话说，就是在对等方连接关闭之后，程序里没有检测到，或者程序压根就忘记了这个时候需要关闭连接，于是这个资源就一直被程序占着；服务器对于程序抢占的资源没有主动回收的权利，除非终止程序运行。

### 如何避免分析
1、代码一定要规范，尤其是在写一些关于自愿申请的部分，一定要在写函数之前写上注释告诉自己别忘了释放资源。
2、数据库连接和访问要设置超时时间，这点避免阻塞。
3、服务器的线程数也需要设置，使得问题尽可能的出现。

### 具体操作
解决方案：代码需要判断 socket ，一旦读到 0，断开连接，read 返回负，检查一下 errno，如果不是 AGAIN，就断开连接。
所以解决 CLOSE_WAIT 大量存在的方法还是从自身的代码出发。

## SYN_RCVD
### 产生原因
SYN_RCVD是TCP三次握手的中间状态，是服务端口（监听端口，如应用服务器的80端口）收到SYN包并发送[SYN，ACK]包后所处的状态。这时如果再收到ACK的包，就完成了三次握手，建立起TCP连接。
如果服务器上**出现大量的SYN_RCVD状态的TCP连接**说明这些连接一直没有收到ACK包，这主要有两种可能，一种是对方（请求方或客户端）没有收到服务器发送的[SYN,ACK]包，另一种可能是对方收到了[SYN,ACK]包却没有ACK。

### 如何避免分析
- 对于第一种情况一般是由于网络结构或安全规则限制导致[SYN,ACK]包无法发送到对方，这种情况比较容易判断：只要在服务器上能够ping通互联网的任意主机，基本可以排除这种情况。
- 对于第二种情况要稍微复杂一些，这种情况还有两种可能：
    - 一种是对方根本就不打算ACK，一般在对方程序有意为之才会出现，如SYN Flood类型的DOS/DDOS攻击；
    - 另一种可能是对方收到的[SYN,ACK]包不合法，常见的是SYN包的目的地址（服务地址）和应答[SYN,ACK]包的源地址不同。这种情况在只配置了DNAT而不进行SNAT的服务网络环境下容易出现，主要是由于inbound（SYN包）和outbound（[SYN,ACK]包）的包穿越了不同的网关/防火墙/负载均衡器，从而导致[SYN,ACK]路由到互联网的源地址（一般是防火墙的出口地址）与SYN包的目的地址（服务的虚拟IP)不同，这时客户机无法将SYN包和[SYN,ACK]包关联在一起，从而会认为已发出的SYN包还没有被应答，于是继续等待应答包。这样服务器端的连接一直保持在SYN_RCVD状态（半开连接）直到超时。

### 具体操作
```sh
sysctl -w net.ipv4.tcp_syncookies=1        #启用使用syncookies
sysctl -w net.ipv4.tcp_synack_retries=1    #降低syn重试次数
```

# 参考资料
- [IP头，TCP头，UDP头，MAC帧头定义](https://www.cnblogs.com/li-hao/archive/2011/12/07/2279912.html)
- [解决 Linux 下 TIME_WAIT 和 CLOSE_WAIT 过多的问题](https://blog.minhow.com/2018/04/15/linux/solve-time-and-close-wait/)
- [对服务器上出现大量的SYN_RCVD状态的TCP连接的问题分析](https://daviswang.iteye.com/blog/819176)
- [TCP CLOSE_WAIT 过多解决方案](https://blog.51cto.com/jin771998569/1688253)

# TODO
- [ ] 2019/06/08: 需要完成iptables与ifconfig的使用说明
