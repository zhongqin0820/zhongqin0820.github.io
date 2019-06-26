---
title: Go实现经典进程同步问题
date: 2019-05-29 10:47:11
updated: 2019-06-24 21:31:48
categories:
- 计算机基础

tags:
- Go
- 学习总结
---
# 前言
梳理总结进程管理相关基础概念以及并发模式相关内容：并发IO（select, poll, epoll）。
使用Go实现包括：读-写者问题、哲学家问题、生产-消费者问题。

<!-- more -->
# 前置参考资料
- [x] [进程、线程知识点总结和同步（消费者生产者，读者写者三类问题）、互斥、异步、并发、并行、死锁、活锁的总结](https://www.cnblogs.com/kubixuesheng/p/4355786.html)
- [x] [并发编程导论](https://segmentfault.com/a/1190000018949353)
- [x] [当我们在谈论高并发的时候究竟在谈什么?](https://segmentfault.com/a/1190000019360335)
- [x] [怎样理解阻塞非阻塞与同步异步的区别？ - 萧萧的回答 - 知乎](https://www.zhihu.com/question/19732473/answer/241673170)

# 基本概念
## 临界资源与临界区
- 临界资源：即共享内存中的内容
- 临界区：与临界资源操作相关的程序段
- 加锁：对临界区临界资源的访问应该是互斥的（即：任一时刻，只有一个进程可以进入临界区访问临界资源），为了达到这个目的，可以通过对临界区加锁达到。

## 进程通信与进程同步
- 同步是**并发程序**按照某种顺序（通过约定的通信方式）**有序访问临界资源**的目的，需要借助通信手段达到。
- 通信是**并发程序**达到同步的手段，包括：
    - 管道（有名、无名）
    - 共享内存与信号量
        - 信号量（锁）：当信号量变量值取值范围为0或1时被称为"互斥量"/"互斥锁"
        - 共享内存（程序读写的内容）
    - 信号：os.SIGNAL
    - socket：异构系统之间的通信方式

## 线程通信与线程同步
由于线程是CPU直接调度的基本单位，需要依赖于进程存在。因此线程的通信手段包括：
- 信号量与共享内存
- 信号：略与进程的接口不同
- [ ] 2019/06/11: 待进一步确认线程通信与进程通信的区别。

## 专有名词分析
- 串行与并行
    - 串行：程序按照某种调度算法，顺序执行，但是可能带来的问题是，由于CPU执行速度远快于IO操作，因此当程序进行IO操作时，串行程序在时间上会浪费CPU的资源（CPU需要等待IO操作的完成）。因此，**时间片轮转**调度算法将CPU按时间片出让给程序，同时利用**并发技术**合理充分利用CPU资源。
    - 并行：前提条件是可以有多个CPU运行多个程序，每个程序在执行时通常独占该CPU的资源。

- 并发与并行
    - 并发：一种技术手段，尽最大限度利用CPU资源。与时间片轮转算法息息相关，表现为在某个时间段內，所有的程序好像同时在执行，但实际上，可能每个时刻只有一个程序被调度（如果是并行的程序，则每个时刻可以有多个程序被调度）。
    - 并行：表现为任一时刻，多个程序同时执行，因此并行的前提是多核而并发。

- 同步与异步：
    - 二者都为并发程序的**执行表现**
    - 同步：并发程序按照某种约定的顺序进行执行，如果不对其进行控制，则结果不可重现
    - 异步：程序的执行，当遇到IO操作的时候，通常不要求立即返回结果，调用该操作的程序会继续执行。

- 阻塞与非阻塞：
    - **任一时刻**，一个CPU核心上只可以运行一个进程。但是可以利用**调度算法**使其在一段时间内运行**多个进程**。
    - 调度算法（以CPU时间片轮转算法为例）通常与**进程状态**（【新建】；【就绪】->【运行】->【等待】；【终止】）以及【系统调用，中断】相关联。
        - 【系统调用，中断】引起正在【运行】进程的【阻塞】，这种状态的改变会导致CPU空闲，调度算法合理调度就绪进程充分利用空闲CPU资源（即进程切换）。
    - 进程的切换通常涉及上下文的切换，需要将旧/新进程的上下文（程序计数器，寄存器内容）对应载入/载出内存（PCB）。
    - 阻塞通常是进程发起一个【系统调用】时（通常是IO操作），调用者由于需要等待系统调用返回的结果，将自己挂起（或阻塞）进入【等待】状态，出让CPU给其它进程。
    - 关于阻塞的讨论，需要在IO模型与进程通信中分别进行讨论：
        - 阻塞：当【系统调用】未返回结果时，调用进程不接着执行；区分于在进程通信中，同步通常意味着阻塞。
        - 非阻塞：【系统调用】未返回结果时，调用进程接着执行，通常伴随使用轮询的机制检查结果是否可用；区分于在进程通信中，异步通常意味着非阻塞。

## 并发IO
> Unix 中内置了 5 种 IO 模型，阻塞式（Blocking） IO, 非阻塞式（Non-Blocking） IO（NIO），IO 复用模型，信号驱动式 IO 和异步（Asynchronous） IO。其中 Blocking 与 Synchronous 大同小异，而 **NIO 与 Async 的区别在于 NIO 强调的是 轮询（Polling），而 Async 强调的是通知（Notification）**。
> **王下月邀熊** --- <cite>[并发编程导论](https://segmentfault.com/a/1190000018949353)</cite>

- 针对IO模型中的区分，非阻塞IO与异步IO都属于非阻塞系统调用
    - 同步阻塞IO：在此种方式下，用户进程在发起一个 IO 操作以后，必须等待 IO 操作的完成，只有当真正完成了 IO 操作以后，用户进程才能运行。
    - 同步非阻塞IO：在此种方式下，用户进程发起一个 IO 操作以后边可返回做其它事情，但是用户进程需要时不时的询问 IO 操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的 CPU 资源浪费。
    - 异步IO：在此种模式下，用户进程只需要发起一个 IO 操作然后立即返回，等 IO 操作真正的完成以后，应用程序会得到 IO 操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的 IO 读写操作，因为真正的 IO 读取或者写入操作已经由内核完成了。

- IO 多路复用通过一种机制，可以监视多个描述符，一旦某个描述符就绪(一般是读就绪或者写就绪)，能够通知程序进行相应的读写操作。常见IO多路复用的机制包括以下三种，本质上属于阻塞式读写。
    - select：轮询式，每次调用 select 都要在用户空间和内核空间里进行内存复制 fd 描述符等信息
    - poll：突破了 select 中描述符数目的限制
    - epoll：在传递内核与用户空间的消息时使用了内存共享，而不是内存拷贝
        - 通过基于内核提供的反射模式，有活跃 Socket 时，内核访问该 Socket 的 callback，**不需要遍历轮询**，分为：
        - 水平触发
            - 与select，poll的机制差不多
            - 读就绪：缓冲区不空
            - 写就绪：缓冲区不满
        - 边缘触发
            - 读就绪：缓冲区不可读变可读或者有新数据到来
            - 写就绪：缓冲区不可写变可写或者有就数据被取走
- [ ] 2019/06/13: [Linux IO模式及 select、poll、epoll详解](http://blog.taohuawu.club/article/linux-io-select-poll-epoll)

# Go中的并发模型
属于两级线程模型，一个KSE（Kernel Scheduling Entity）即系统线程，对应n个用户级线程即MPG模型中的M，一个M通过一个P可以调度m个G。即，KSE:M=1:n（涉及操作系统调度器），M:P:G=1:1:m（涉及goroutine调度器）。因此，KSE:G=n:m。

![调度模型](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/go/goroutine.png)

## MPG模型
- M：OS线程抽象，阻塞类型可以分为两类
    - 用户态阻塞/唤醒
    - 系统调用阻塞
- P：逻辑调度器，P是5状态模型，分配执行的上下文环境，每次只可以**随机调度（查找顺序【本地G队列】->【全局G队列】->【其它P的G队列】）**一个runnable的`G` mapping 一个`M`。
- G：goroutine，G是7状态模型，即程序中对应`go`关键字部分的内容，维护着执行需要的栈、程序计数器以及所映射M的信息。
    - 引起阻塞的操作：阻塞的系统调用，网络输入，通道操作，sync的原语操作
    - **每一个G都会由运行时系统根据其实际状况设置不同的状态，其主要状态如下。**
    - `Gidle` 表示当前G刚被新分配，但还未初始化。
    - `Grunnable` 表示当前G正在可运行队列中等待运行。
    - `Grunning` 表示当前G正在运行。
    - `Gsyscall` 表示当前G正在执行某个系统调用。
    - `Gwaiting` 表示当前G正在阻塞。
    - `Gdead` 表示当前G正在闲置。
    - `Gcopystack` 表示当前G的栈正被移动，移动的原因可能是栈的扩展或收缩。

> 注意，goroutine的调度，需要注意如果涉及使用外部变量，需要传参给内部。

## 共享内存与消息通信模型
- 共享内存：利用传统的锁机制
- 消息通信模型：利用通道，并配合利用锁机制

## 扩展阅读
- [Go的线程实现模型](http://www.ituring.com.cn/book/tupubarticle/16048)
- [也谈goroutine调度器](https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/)
- [Goroutine并发调度模型深度解析之手撸一个协程池](http://blog.taohuawu.club/article/42)

## 机制
### 锁
基于共享内存的并发模式下，实现goroutine的同步。涉及：
- sync
    - Mutex struct
        - `Lock()`
        - `Unlock()`
    - RWMutex struct：读者优先
        - `RLock()
        - `RUnlock()`
        - `Lock()`
        - `Unlock()`
    - WaitGroup struct：
        - 通过利用计数器，用于等待一批 Go 协程执行结束，可以用于确保多个goroutine 在main.goroutine退出前都能够执行
        - `Add(int)`
        - `Done()`
        - `Wait()`

> 注意，如果需要对其修改，需要传指针。

### 原子操作
- sync/atomic
    - 增减
    - 载入
    - 比较并交换
    - 交换
    - 存储

> 调用方便，调用时传地址，与锁机制在技术上相近。

### 通道
- 源码地址v1.12：[src/runtime/chan.go](https://golang.org/src/runtime/chan.go)
- [GopherCon 2017: Kavya Joshi - Understanding Channels](https://www.youtube.com/watch?v=KBZlN0izeiY)
    - hchan：buf（环形队列），lock，recvx，sendx，【recvq，sendq】与并发操作相关
    - sudoq：双向链表结构，元素是goroutine
    - 每次读/写操作都需要使用`lock`加锁
    - goroutine阻塞：向满的channel写/从空的channel读
    - goroutine唤醒：空的channel中被写入数据/满的channel中被取走数据【利用sudog数据结构，直接互相读/写值】

- channel的坑
    - channel必须带数据类型，可以先声明channel类型的变量，但在使用前，需要使用`make()`对其进行赋值，同时指定缓冲长度，即通道中允许排队的元素个数
    - 单向通道通常作为函数的参数，在变量声明中是不应该出现单向通道的，因为通道本来就是为了通信而生，只能接收或者只能发送数据的通道是没有意义的。
    - 程序中必须同时有不同的 goroutine 对非缓冲通道进行发送和接收操作，否则会造成死锁。
    - 如果写完需使用`close()`关闭channel，否则利用`for range`形式迭代会死锁
    - 如果想要获取通道内元素的个数，可以直接使用内置函数`len()`

- `select` 专门用于通道发送和接收操作，看起来和 `switch` 很相似，但是进行选择和判断的方法完全不同。
- 使用非缓冲通道代替传统同步机制（互斥锁等），从而保证goroutine在main.goroutine退出前执行完成。

# 经典同步问题
- 代码实现：[concurrent/classic](https://github.com/zhongqin0820/coding-playground/tree/master/go/concurrent/classic)

## 读-写者问题
- [问题描述](https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)
- [其他解决方案](https://songlee24.github.io/2015/04/30/linux-three-syn-problems/)

## 哲学家问题
- [问题描述](http://en.wikipedia.org/wiki/Dining_philosophers_problem)
- [讨论](https://forum.golangbridge.org/t/dining-philosophers-golang/11089/4)

## 生产-消费者问题
- [问题描述](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)
- [其它解决方案](https://tpaschalis.github.io/golang-producer-consumer/)
- 消费者按照生产者生产的顺序往外读，生产者并发生产

# 结束语
在进行并发编程的时候，需要考虑的问题是，goroutine的调度要把它想象成是同时在进行的。因此，如果没有对通道进行同步控制则每次执行的结果是不同的（不可重现）。

# Changelog
- 2019/06/11：添加针对基础概念的梳理，以及相关参考资料链接
- 2019/06/13：添加[Linux IO模式及 select、poll、epoll详解](http://blog.taohuawu.club/article/linux-io-select-poll-epoll)。
- 2019/06/18：修改完善MPG模型描述与添加扩展阅读材料
- 2019/06/24：添加在channel中的MPG模型执行过程
