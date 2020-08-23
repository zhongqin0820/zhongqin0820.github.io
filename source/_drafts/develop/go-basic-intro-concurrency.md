---
title: Go语言入门总结：并发
date: 2019-07-03 17:39:22
updated: 2019-07-03 21:56:49
categories:
- Go语言入门

tags:
- Go
---
# 前言
梳理总结学习Go语言的笔记。相较并行，并发是一种结构上的关系，而并行需要硬件的支持是一种执行关系。并发的意义是为了能够最大化利用单个CPU的计算资源。Go在语言层面上支持并发；本篇文章从单个协程到多个协程的并发执行，逐步分析如何使用Go进行并发编程；同时，分析了异步与同步两种并发目的如何通过Go实现。完整代码访问[GitHub](https://github.com/zhongqin0820/coding-playground/blob/master/go/concurrent/basic/README.md)。

<!-- more -->
# 单个协程
在Go中使用`go`关键字开启一个`goroutine`（协程）。相较使用有名函数调用，也可以使用匿名函数实现在同一区域的代码段同时定义与调用函数。Go将`main.goroutine`视作一个协程（即：主协程）；因此，此处实际有2个协程同时调度执行，而程序的退出是基于主协程的生命周期；
```go
// 开启一个协程打印msg
go func(msg string) {
    println(msg)
}("hello world without sleep")
// 使用time.Sleep阻塞主协程，确保子协程的执行完成
// time.Sleep(1e9)
```
如果不对多个`goroutine`进行并发控制（此例中使用`time.Sleep`），则可能在`main.goroutine`退出执行时，其它`goroutine`还未执行完毕返回结果。

# 多个协程
为了"复用"匿名函数的代码，可以将其赋值给一个变量（使用类型自动推导或自己指定类型，前者更为方便）；
```go
// 定义一个匿名函数，并将其赋值给变量f
var f = func(msg string) {
    println(msg)
}
// 开启多个协程打印msg
go f("hello")
go f("world")
// 依旧使用time.Sleep阻塞主协程，由于无法预知每个子协程的具体执行时间
// 可能无法确保所有子协程的执行完成
// time.Sleep(1e9)
```
在Go中，为了确保多个协程的并发执行，在标准库`sync`中定义了`sync.WaitGroup`（等待组）对并发协程进行管理，其确保在主协程退出前，所有的子协程都执行完毕。
```go
// 由于需要对wg修改，此处传入指针
var fw = func(wg *sync.WaitGroup, msg string) {
    defer wg.Done()
    println(msg)
}
// 开启多个协程并发打印msg
wg := sync.WaitGroup{}
wg.Add(2)// 此处2为子协程数
go fw(&wg, "hello")
go fw(&wg, "world")
wg.Wait()
```

# 异步
异步编程是并发模型的一种，目的是实现协程的异步调用；函数作为Go中的第一等公民，可以通过回调函数的方式实现异步编程。下面是一个回调函数的定义：
```go
// 注意：当函数做参数时应该只传入函数的签名，应当将其看作是一种类型f func(s string)
var callback = func(msg string, f func(s string)) {
    // callback函数：开启一个协程并发调度函数：f func(s string)
    go func() {
        f(strings.ToUpper(msg))
    }()
}
```
在调用时，因为涉及多个协程的调度，应该使用`sync.WaitGroup`对其进行并发控制；
```go
wg := sync.WaitGroup{}
wg.Add(1)
// 注意此处调用回调封装时，传入的是实参，例如string类型的变量实例以及签名为func(string)的函数定义
// func(string)的函数定义中应该只是对其签名func(s string)中参数的调用
callback("hello world callback", func(s string) {
    defer wg.Done()
    log.Println("call: ", s)
})
log.Println("wait for async call")
wg.Wait()
log.Println("end of async call")
// 前两行的输出顺序不一定
// 2019/07/03 20:10:45 wait for async call
// 2019/07/03 20:10:45 call:  HELLO WORLD CALLBACK
// 2019/07/03 20:10:45 end of async call
```
但异步编程并不总是没有缺点，过深的回调，可能使程序陷入`回调地狱`，如：
```go
var wg sync.WaitGroup = sync.WaitGroup{}
wg.Add(2)
// 此处是回调函数的调用
callback("outter", func(s string) {
    defer wg.Done()
    callback(fmt.Sprintf("inner: %s\n", s), func(s string) {
        defer wg.Done()
        log.Println("call: ", s)
    })
})
log.Println("wait for async callback")
wg.Wait()
log.Println("end of async call")
```

# 同步
同步是对多个`goroutine`对临界资源进行并发访问时确保任意时刻，仅有一个`goroutine`对临界区的临界资源进行访问。

## 锁机制
在标准库中提供了传统的同步机制，如：锁机制，原子操作以及信号机制等。以锁机制为例，这里通过一个计数器实例进行说明：
```go
// Counter 结构，匿名嵌套sync.Mutex（相当于OOP中的继承）
type Counter struct {
    sync.Mutex // embedding
    value      int
}
// 使用锁机制进行同步
// 使用sync.WaitGroup对多个并发协程进行控制，确保在主协程退出前，所有子协程都执行完毕
c := Counter{}
wg := sync.WaitGroup{}
for i := 0; i < 4; i++ {
    wg.Add(1)
    go func() {
        c.Lock()
        defer c.Unlock()
        defer wg.Done()
        // 此处c.value相当于临界资源
        c.value++
    }()
}
wg.Wait()
println(c.value)
// 4
```

## 通道
但在Go中更惯用的方式是使用顺序通讯机制，即Communication Sequence Protocol（CSP）模型；Go内置`chan`结构实现该模型，使用内置`make`函数创建对应`chan T`变量；需要特别注意的是，如使用`for range`读通道数据需要在对其写结束后使用`close()`对其进行手动关闭。

### 单个协程单个无缓冲通道
针对单个协程打印的例子，可以使用单个无缓冲通道对其进行实现。
```go
// 针对单个协程打印的例子，使用无缓冲通道传输数据
c := make(chan string)
go func(c chan<- string, msg string) {
    c <- msg
}(c, "hello world")
msg := <-c
println(msg)
```

### 多个协程多个无缓冲通道
使用`select`关键字操作多个通道，与`switch`一样，`select`只会执行一次，如果想要多次执行，可以外包一层`for`循环。
```go
// 针对多个协程中的例子，将其改写成如下的函数，注意通道数据的方向
// 定义一个匿名函数，并将其赋值给变量f
var f = func(c chan<- string, msg string) {
    c <- msg
}
// 使用select进行读写控制
func MyPrint(c <-chan string, quitCh chan<- bool) {
    for {
        select {
        case msg := <-c:
            println(msg)
        case <-time.After(time.Second * 2):
            // time.After() 返回一个接收通道：<- chan time.Time
            println("break after 2s")
            quitCh <- true
            break
        }
    }
}
// 开启多个协程并发打印msg
c := make(chan string)
q := make(chan bool)
go f(c, "hello")
go MyPrint(c, q)
go f(c, "world")
<-q
```

### 有缓冲通道
相较无缓冲通道，有缓冲通道可以加速程序的执行，本例使用带缓冲通道对数据传输进行管理，同时使用`sync.WaitGroup`确保所有子协程（此例中共两个子协程）的执行。
```go
c := make(chan int, 2)
wg := sync.WaitGroup{}
wg.Add(2)
// 生产者
go func() {
    defer wg.Done()
    for i := 0; i < 5; i++ {
        c <- i
    }
    close(c)// 需要在写完后关闭通道，否则for range会死锁
}()

// 消费者
go func() {
    defer wg.Done()
    // 使用for range迭代管道
    for v := range c {
        println(v)
    }
}()
wg.Wait()
```
稍微优雅一些的方式是使用`for range`一个返回接收通道的协程。
```go
f := func() <-chan int {
    // 内部定义一定匿名goroutine
    c := make(chan int, 2)
    go func() {
        for i := 0; i < 5; i++ {
            c <- i
        }
        close(c)
    }()
    return c
}
// for range迭代获得通道内容
for v := range f() {
    println(v)
}
```

# Changelog
- 2019/07/03：初版文章内容
