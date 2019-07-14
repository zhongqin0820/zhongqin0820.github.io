---
title: Go实现并发型模式：Future模式
date: 2019-07-06 22:24:08
updated: 2019-07-11 21:47:24
categories:
- 软件模式

tags:
- 并发模式
- Go
---
# 前言
Future模式也叫做Future & Promise 模式，属于一种实现异步调用的并发模式，在Node.js的设计中大量使用该模式，是事件驱动编程的典型案例。

<!-- more -->
# 场景描述
大部分情况下，Future模式用于需要等待IO操作，相当于一个占位符，如：数据库的读取或网络资源请求等。在资源请求过程中，主线程可以做其它工作，直到请求结果返回，才接着执行。如：[Concurrency Patterns: Golang](https://medium.com/@thejasbabu/concurrency-patterns-golang-5c5e1bcd0833)中所提到的网络请求案例。

为了进一步体会Future模式的精髓，在本例中，通过实现一个简单的异步请求来熟悉Future模式。
## 关键点
- 其思想是预先定义行为，让Future调用者提供可能的解决方案。
- 链式组合成功/失败调用：
    - `func (s *MaybeString) Success(f SuccessFunc) *MaybeString`
    - `func (s *MaybeString) Fail(f FailFunc) *MaybeString `
- 通过回调的方式实现，将回调函数句柄`f ExecuteStringFunc`传入执行函数中，并开启一个goroutine并发执行（没有进行同步操作，相当于异步）。
    - `func (s *MaybeString) Execute(f ExecuteStringFunc)`
- 回调函数返回期望的结果或错误，如：`type ExecuteStringFunc func() (string, error)`

# 示意图
主流程中通过新建一个goroutine执行一个Future调用，在Future的成功调用中可以嵌套调用更多的Future，直到返回结果。

在下面的示意图中，`r2.goroutine`异步调用执行，如果没有错误，则会开启一个新协程接着执行`Success()`中的嵌套Future，否则直接通过调用`Fail()`结束执行。

<div style="width: 300px; margin: auto">
![示意图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/concurrency_future.png)
</div>

# 样例代码
- 完整代码参见[Github](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/concurrency/future)

## 代码实现
- 句柄声明

```go
// executed if everything went well
type SuccessFunc func(string)

// receive an error when something goes wrong
type FailFunc func(error)

// defines the operation we want to perform
type ExecuteStringFunc func() (string, error)
```

- 封装实现：
    - 参照建造者模式创建实例的方式，分别链式赋值`successFunc`与`failFunc`
    - 在`func (s *MaybeString) Execute(f ExecuteStringFunc) `中开启一个协程：根据`ExecuteStringFunc`方法返回的结果异步执行`SuccessFunc`或`FailFunc`。

```go
type MaybeString struct {
    successFunc SuccessFunc
    failFunc    FailFunc
}

// accept a SuccessFunc function that will be executed if everything goes correctly
// and return the same pointer to the future object (so we can keep chaining)
func (s *MaybeString) Success(f SuccessFunc) *MaybeString {
    s.successFunc = f
    return s
}

// accept a FailFunc function to later return the pointer.
func (s *MaybeString) Fail(f FailFunc) *MaybeString {
    s.failFunc = f
    return s
}

// pass an ExecuteStringFunc type (a function that accepts nothing and returns a string or an error)
func (s *MaybeString) Execute(f ExecuteStringFunc) {
    // launch the Goroutine that executes the f method (an ExecuteStringFunc)
    // and takes its result
    go func(s *MaybeString) {
        str, err := f()
        if err != nil {
            s.failFunc(err)
        } else {
            s.successFunc(str)
        }
    }(s)
}
```

## 测试代码
- 并发测试，使用一个超时机制来进行测试。

```go
// waits for a second, then prints a message, sets the test as failed
// and lets the WaitGroup continue by calling its Done() method
func timeout(t *testing.T, wg *sync.WaitGroup) {
    // the execution time is now nearly zero, so our timeouts have not been executed
    // actually, they were executed, but the tests already finished and their result was already stated.
    time.Sleep(time.Second)
    t.Log("Timeout!")
    t.Fail()
    wg.Done()
}
```

- 扩展闭包

```go
// use closures to introduce a context into this type of function.
// it takes a string as an argument and returns an ExecuteStringFunc method
// that returns the previous argument with the suffix Closure!
func setContext(msg string) ExecuteStringFunc {
    msg = fmt.Sprintf("%s Closure!\n", msg)
    return func() (string, error) {
        return msg, nil
    }
}

// a test that uses the closure
t.Run("Closure Success result", func(t *testing.T) {
    var wg sync.WaitGroup
    wg.Add(1)
    //Timeout!
    go timeout(t, &wg)
    ms.Success(func(s string) {
        t.Log(s)
        wg.Done()
    }).Fail(func(e error) {
        t.Fail()
        wg.Done()
    })
    ms.Execute(setContext("Hello"))
    wg.Wait()
})
```

# 扩展
在本例中，我们还可以利用接口分别封装`Success`，`Fail`以及`Execute`方法对其进行进一步抽象。


# 学习资料
- Blog
    - [Concurrency Patterns: Golang](https://medium.com/@thejasbabu/concurrency-patterns-golang-5c5e1bcd0833)
    - 维基百科：[Future与promise](https://zh.wikipedia.org/wiki/Future%E4%B8%8Epromise)

- 其它代码实现
    - PacktPublishing/Go-Design-Patterns：[Chapter09](https://github.com/PacktPublishing/Go-Design-Patterns/tree/master/Chapter09)

# Changelog
- 2019/07/11：第一版