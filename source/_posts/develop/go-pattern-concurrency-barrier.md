---
title: Go实现并发型模式：Barrier模式
date: 2019-06-29 21:23:00
updated: 2019-07-11 11:54:01
categories:
- 软件模式

tags:
- 并发模式
- Go
---
# 前言
Barrier模式也叫同步屏障模式，属于一种实现同步操作的并发模式，其通过设置一个屏障来收集当前协程/线程所需要的返回结果。

<!-- more -->
# 场景描述
该模式在并发应用中非常常见的。例如：当我们有一个微服务应用中的某个服务需要通过归并组合另外三个微服务返回的结果作为当前这个服务的结果时。

同步屏障模式通过同步阻塞来实现屏障操作，直到组成该返回结果所依赖的其它一个或多个不同协程/线程（微服务）的结果都被得到。在Go中，虽然标准库中提供了一些同步原语，如锁机制等，但是更惯用的方法是通过通道来实现。

## 关键点
同步屏障模式的目标如下：
- 组合来自一个或多个协程的同类型结果
- 控制来自多个不同协程的数据传输通道保证没有不一致的数据被返回。我们不希望因为数据通道中数据返回了一个错误导致收集到的数据不完整。

# 示意图
通过实现一个HTTP GET返回结果的收集器来熟悉该模式。大致示意图如下，其中实线表示一个请求`makeRequest(chan<- barrierResp, string)`，虚线表示返回结果通道`<-chan barrierResp`。

该示例中，主协程`r1.goroutine`开启两个子协程`r2.goroutine`与`r3.goroutine`处理请求，并通过一个公共通道返回处理结果。

<div style="width: 300px; margin: auto">

![示意图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/develop/pattern/concurrency_barrier.png)
</div>

# 样例代码
- 完整代码参见[Github](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/concurrency/barrier)

## 代码实现
```go
// 主流程
func barrier(endpoints ...string) {
    requestNumber := len(endpoints)
    in := make(chan barrierResp, requestNumber)
    defer close(in)
    responses := make([]barrierResp, requestNumber)
    // 开启多个协程工作并通过公共管道收集结果
    for _, endpoint := range endpoints {
        go makeRequest(in, endpoint)
    }
    var hasError bool
    // 确认结果是否有错误
    for i := 0; i < requestNumber; i++ {
        resp := <-in
        if resp.Err != nil {
            // 找到错误，则按照约定添加前缀"ERROR: "
            fmt.Println("ERROR: ", resp.Err)
            hasError = true
        }
        responses[i] = resp
    }
    // 没有找到错误，将结果输入控制台，并关闭通道退出主流程
    if !hasError {
        for _, resp := range responses {
            fmt.Println(resp.Resp)
        }
    }
}

// 返回结果
func captureBarrierOutput(endpoints ...string) string {
    // a pipe allows us to connect an io.Writer interface to an io.Reader interface
    // so that the reader input is the Writer output
    reader, writer, _ := os.Pipe()
    os.Stdout = writer
    out := make(chan string)
    go func() {
        var buf bytes.Buffer
        // 从控制台处读结果，并写入到out中
        io.Copy(&buf, reader)
        out <- buf.String()
    }()
    barrier(endpoints...)
    // 所有结果收集完才会关闭
    writer.Close()
    temp := <-out
    return temp
}
```

## 测试代码
共包含三个测试样例，以正确调用样例为例。
```go
// makes two calls to the correct endpoints
t.Run("Correct endpoints", func(t *testing.T) {
    endpoints := []string{
        "http://httpbin.org/headers",
        "http://httpbin.org/user-agent",
    }
    result := captureBarrierOutput(endpoints...)
    if !strings.Contains(result, "Accept-Encoding") || !strings.Contains(result, "User-Agent") {
        t.Fail()
    }
    t.Log(result)
})
```

# 扩展
同步屏障不仅可以用于网络请求结果的聚合中，我们还可以将其用于需要将一个资源耗费大的任务分割成多个相对占用资源较小的任务中，分别在多个goroutine中执行多个小任务后返回结果的应用场景中。例如：大文件词频统计。

# 学习资料
- 博客/出版物
    - [Barrier Synchronization Pattern](https://pdfs.semanticscholar.org/08c5/6ccee20203f561e8603b616cc2c8eac98eae.pdf) Rajesh K. Karmani
    - 维基百科：[同步屏障](https://zh.wikipedia.org/wiki/%E5%90%8C%E6%AD%A5%E5%B1%8F%E9%9A%9C)

- 其它代码实现
    - PacktPublishing/Go-Design-Patterns：[Chapter09](https://github.com/PacktPublishing/Go-Design-Patterns/tree/master/Chapter09)
    - ishare/go-patterns：[concurrency/barrier](https://github.com/ishare/go-patterns/blob/master/concurrency/barrier.md)

# Changelog
- 2019/07/11：第一版内容
