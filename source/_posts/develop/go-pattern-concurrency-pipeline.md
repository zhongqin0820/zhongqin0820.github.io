---
title: Go实现并发型模式：Pipeline模式
date: 2019-07-08 14:29:59
updated: 2019-07-11 22:47:21
categories:
- 软件模式

tags:
- 并发模式
- Go
---
# 前言
Pipeline模式也叫作流水线模式。它将消息分发给多个工作协程，通过通道收集结果。在并发编程中流水线模式的应用十分普遍，是一个十分有用的工具。

<!-- more -->
# 场景描述
通过使用一些相互调用的函数，可以使流水线模式更加复杂。他们甚至可以被循环执行。Go中的流水线模式以类似的方式工作，但流水线中的每一步都将以不同的方式进行，并使用通道进行同步。

尽管创建更多的goroutine并不意味着更优的执行性能，但是创建goroutine和channels的开销可能会使算法的复杂度变小。
## 关键点
- 分解工作流执行步骤
- 并发执行各个步骤，通过利用通道传递结果。

# 示意图
本例子中通过实现一个分布式计算1～N整数平方和来熟悉流水线模式。这个例子，大致由以下几个步骤组成：
- 生成一个从1到n的列表，其中n可以是任意整数
- 对生成的列表中的每一个整数求平方
- 求和平方并返回结果

<div style="width: 300px; margin: auto">
![示意图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/concurrency_pipeline.png)
</div>

# 样例代码
- 完整代码参见[Github](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/concurrency/pipeline)

## 代码实现
- 主流程实现

```go
// separate every operation in a different Goroutine and connect them with channels.
func LaunchPipeline(amount int) int {
    firstCh := generator(amount)
    secondCh := power(firstCh)
    thirdCh := sum(secondCh)
    result := <-thirdCh
    return result
}
```

- 以步骤2为例：通过在函数内部开启一个goroutine执行一个匿名函数，并返回一个`<-chan int`通道。

```go
func power(in <-chan int) <-chan int {
    out := make(chan int, 100)

    go func() {
        for v := range in {
            out <- v * v
        }
        close(out)
    }()

    return out
}
```

## 测试代码
使用表驱动测试
```go
func TestLaunchPipeline(t *testing.T) {
    //  a table of tests
    tableTest := [][]int{
        {3, 14},
        {5, 55},
    }

    var res int
    for _, test := range tableTest {
        res = LaunchPipeline(test[0])
        if res != test[1] {
            t.Fatal()
        }
        t.Logf("%d == %d\n", res, test[1])
    }
}
```

# 扩展
使用流水线模式，我们可以非常容易地创建非常复杂的并发工作流。在本例中，我们创建了一个线性工作流，但是它也可以有conditionals、pools以及fan-in和fan-out 行为。

# 学习资料
- 博客/出版物
    - [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)
    - [Pipeline Patterns in Go](https://medium.com/statuscode/pipeline-patterns-in-go-a37bb3a7e61d)
    - [Go Pipelines](https://gist.github.com/brandur/0ed11ad9480809aad0dacecfcac41790)
    - [pipeline模式的理解](http://weakyon.com/2015/11/07/pipeline-mode.html)

- 其它代码实现
    - PacktPublishing/Go-Design-Patterns：[Chapter09](https://github.com/PacktPublishing/Go-Design-Patterns/tree/master/Chapter09)

# Changelog
- 2019/07/11：第一版内容