---
title: Go实现经典算法问题：三大同步问题
date: 2019-05-29 10:47:11
updated: 2019-06-24 21:31:48
categories:
- 计算机基础

tags:
- Go
- 经典算法
---
# 前言
梳理总结Go中的并发模型，并使用Go实现包括：读-写者问题、哲学家问题、生产-消费者问题。

<!-- more -->
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
- [`sync`包](https://golang.google.cn/pkg/sync/)
    - `type Mutex struct`
        - `Lock()`
        - `Unlock()`
    - `type RWMutex struct`：读者优先
        - `RLock()`
        - `RUnlock()`
        - `Lock()`
        - `Unlock()`
    - `type WaitGroup struct`：
        - 通过利用计数器，用于等待一批 Go 协程执行结束，可以用于确保多个goroutine 在`main.goroutine`退出前都能够执行
        - `Add(int)`
        - `Done()`
        - `Wait()`
    - `type Once struct`：Once是只执行一次动作的对象
    - `type Cond struct`：用来控制某个条件下，goroutine 进入等待时期，等待信号到来，然后重新启动

> 注意，如果需要对其修改，需要传指针。

### 临时对象池
- [`sync`包](https://golang.google.cn/pkg/sync/)
- `sync.Pool`：当多个 goroutine 都需要创建同一个对象的时候，如果 goroutine 过多，可能导致对象的创建数目剧增。 而对象又是占用内存的，进而导致的就是内存回收的 GC 压力徒增。造成“并发大－占用内存大－ GC 缓慢－处理并发能力降低－并发更大”这样的恶性循环。

```go
type Pool struct {
    // 可选参数New指定一个函数在Get方法可能返回nil时来生成一个值
    // 该参数不能在调用Get方法时被修改
    New func() interface{}
    // 包含隐藏或非导出字段
}
// Get方法从池中选择任意一个item，删除其在池中的引用计数，并提供给调用者。
func (p *Pool) Get() interface{}
// Put方法将x放入池中。
func (p *Pool) Put(x interface{})
```

### 原子操作
- [`sync/atomic`包](https://golang.google.cn/pkg/sync/atomic/)
    - 增减
    - 载入
    - 比较并交换
    - 交换
    - 存储

> 调用方便，调用时传地址，与锁机制在技术上相近。

### 通道
#### channel的原理
- 源码地址v1.12：[src/runtime/chan.go](https://golang.org/src/runtime/chan.go)
- [GopherCon 2017: Kavya Joshi - Understanding Channels](https://www.youtube.com/watch?v=KBZlN0izeiY)
    - hchan：buf（环形队列），lock，recvx，sendx，【recvq，sendq】与并发操作相关
    - sudoq：双向链表结构，元素是goroutine
    - 每次读/写操作都需要使用`lock`加锁
    - `goroutine阻塞`：向满的channel写/从空的channel读
    - `goroutine唤醒`：空的channel中被写入数据/满的channel中被取走数据【利用sudog数据结构，直接互相读/写值】

#### channel的坑
- channel必须带数据类型，可以先声明channel类型的变量，但在使用前，需要使用`make()`对其进行赋值，同时指定缓冲长度，即通道中允许排队的元素个数
- 单向通道通常作为函数的参数，在变量声明中是不应该出现单向通道的，因为通道本来就是为了通信而生，只能接收或者只能发送数据的通道是没有意义的。
- 程序中必须同时有不同的 goroutine 对非缓冲通道进行发送和接收操作，否则会造成死锁。
- 如果写完需使用`close()`关闭channel，否则利用`for range`形式迭代会死锁
- 如果想要获取通道内元素的个数，可以直接使用内置函数`len()`
- `select` 专门用于通道发送和接收操作，看起来和 `switch` 很相似，但是进行选择和判断的方法完全不同，`select`是随机选择就绪的通道，`switch`是根据条件判断进行选择。
- 使用非缓冲通道代替传统同步机制（互斥锁等），从而保证`goroutine`在`main.goroutine`退出前执行完成。

#### channel常用代码思想
```go
// 下列程序实现了并发的读/写
// Do()包装一个匿名函数的goroutine对通道进行写操作，调用返回一个发送通道
func Do() <-chan int {
    var a chan int = make(chan int)
    i := 0
    go func() {
        for ; i < 5; i++ {
            a <- i
        }
        // 发送完毕，需要关闭该通道
        close(a)
    }()
    return a
}

// 使用for range读取值，等待组的引入是为了保证多个gorountine的执行完整
func TestClassic(t *testing.T) {
    var wg sync.WaitGroup = sync.WaitGroup{}
    wg.Add(1)
    go func() {
        for v := range Do() {
            log.Println(v)
        }
        wg.Done()
    }()
    wg.Wait()
    t.Log()
}
```

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
- 2019/06/18：修改完善MPG模型描述与添加扩展阅读材料
- 2019/06/24：添加在channel中的MPG模型执行过程
- 2019/07/01：添加Cond与Once的内容，以及关于临时对象池的内容
