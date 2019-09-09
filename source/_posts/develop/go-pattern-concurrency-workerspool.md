---
title: Go实现并发型模式：Workers Pool模式
date: 2019-07-10 14:32:16
updated: 2019-07-14 18:37:21
categories:
- 软件模式

tags:
- 并发模式
- Go
---
# 前言
Workers Pool模式也叫作Thread Pool模式/线程池模式。在Go的实现简单的Workers Pool模式，通过利用通道控制n个任务，让可预设的m个goroutine处于空闲或忙碌状态从而限制可用数量的goroutine来并发执行n个任务，进而更深入的控制资源池（所控制资源内容包括如：CPU、RAM、时间片、连接等等）。

<!-- more -->
# 场景描述
- 使用配额控制对共享资源的访问
- 为每个应用程序创建有限数量的Goroutines
- 为其他并发结构提供更多并行能力

本文中对该模式的讨论，goroutine的状态控制是通过自带线程模型中的Processor，更复杂的状态控制实现可参考[Building a Worker Pool in Golang](https://geeks.uniplaces.com/building-a-worker-pool-in-golang-1e6c0fdfd78c)。

如下图所示，借助管道可以很方便实现该模式。每个worker中的job可以通过[Pipeline模式](https://cvblogs.cn/2019/07/08/develop/go-pattern-concurrency-pipeline)实现。
<div style="width: 300px; margin: auto">

![流程图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/concurrency_worker1.png)
</div>

## 关键点
关于模式的具体实现过程中，可以抽象为如下内容：
- Dispatcher：调度器接口
- WorkerLauncher：工人（任务执行）接口
- dispatcher：具体调度器
- PrefixSuffixWorker：具体工人（任务执行）
- Request：具体业务内容封装

# 示意图
<div style="width: 300px; margin: auto">

![UML](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/concurrency_worker2.png)
</div>

# 样例代码
样例是实现的是一个将输入字符串转化为大写后为其拼接上前后缀。完整代码参见[Github](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/concurrency/workerPool)。

## 代码实现
字符串算法的流程实现基于[Pipeline模式](https://cvblogs.cn/2019/07/08/develop/go-pattern-concurrency-pipeline)。
```go
func (w *PreffixSuffixWorker) LaunchWorker(in chan Request) {
    // each work is a pipeline of Request(pipeline pattern)
    w.prefix(w.append(w.uppercase(in)))
}
```

`dispatcher`的具体实现中通过管道将任务`r Request`分配给具体的`worker`。
```go
// stores a channel of Request type
type dispatcher struct {
    inCh chan Request
}

func (d *dispatcher) LaunchWorker(w WorkerLauncher) {
    w.LaunchWorker(d.inCh)
}

func (d *dispatcher) Stop() {
    close(d.inCh)
}

func (d *dispatcher) MakeRequest(r Request) {
    select {
    case d.inCh <- r:
    case <-time.After(time.Second * 5):
        return
    }
}
```

## 测试代码
`worker`的数量在客户端指定。如下所示的测试代码中，虽然我们很难验证并发程序的具体执行过程。但是，可以通过下面的测试代码大致验证程序的执行过程。

```go
// 初始化调度器
bufferSize := 100
var dispatcher Dispatcher = NewDispatcher(bufferSize)

// 添加worker到调度器dispatcher中
// 此处也可用工厂方法将具体的WorkerLauncher初始化封装起来
workers := 3
for i := 0; i < workers; i++ {
    var w WorkerLauncher = &PreffixSuffixWorker{
        prefixS: fmt.Sprintf("WorkerID: %d -> ", i),
        suffixS: " World",
        id:      i,
    }
    // 分派任务给具体的worker执行任务
    dispatcher.LaunchWorker(w)
}

// 具体的任务数
requests := 10
var wg sync.WaitGroup
wg.Add(requests)

for i := 0; i < requests; i++ {
    // 新建任务
    req := Request{
        Data: fmt.Sprintf("(Msg_id: %d) -> Hello", i),
        Handler: func(i interface{}) {
            // 闭包函数Handler对结果进行测试
            s, ok := i.(string)
            defer wg.Done()
            if !ok {
                t.Fail()
            }
            ok, err := regexp.Match(`WorkerID\: \d* -\> \(MSG_ID: \d*\) -> [A-Z]*\sWorld`, []byte(s))
            if !ok || err != nil {
                t.Fail()
            }
            t.Log(s)
        },
    }
    // 添加任务
    dispatcher.MakeRequest(req)
}
// 任务添加完毕
dispatcher.Stop()
wg.Wait()
```

# 扩展
- 引入错误处理管道的问题：
> if our error channel is smaller than the number of work items that will produce an error, then workers will block as they try to put an error into it. This will cause a deadlock.
> It’s quite possible to address that problem as well, but it helps to show that developing a useful and bug-free worker pool in Go isn’t quite as simple as it’s often made out to be.
> **brandur** --- <cite>[The Case For A Go Worker Pool](https://brandur.org/go-worker-pool)</cite>

- 引入状态管理：可参考[Building a Worker Pool in Golang](https://geeks.uniplaces.com/building-a-worker-pool-in-golang-1e6c0fdfd78c)

# 学习资料
- 博客/出版物
    - [Go by Example: Worker Pools](https://gobyexample.com/worker-pools)
    - [The Case For A Go Worker Pool](https://brandur.org/go-worker-pool)
    - [Building a Worker Pool in Golang](https://geeks.uniplaces.com/building-a-worker-pool-in-golang-1e6c0fdfd78c)
    - [Implementing a worker pool](https://hspazio.github.io/2017/worker-pool/)
    - [Handling 1 Million Requests per Minute with Go](http://marcio.io/2015/07/handling-1-million-requests-per-minute-with-golang/)
    - 维基百科：[线程池](https://zh.wikipedia.org/wiki/%E7%BA%BF%E7%A8%8B%E6%B1%A0)

- 其它代码实现
    - [Jeffail/tunny](https://github.com/Jeffail/tunny)：a goroutine pool for Go
    - [goinggo/work](https://github.com/goinggo/work)：manages a pool of routines to perform work

# Changelog
- 2019/07/14：第一版内容