---
title: Go实现创建型模式：单例模式
date: 2019-06-07 17:04:15
updated: 2019-06-09 13:37:25
categories:
- 软件模式

tags:
- 设计模式
- Go
---
# 前言
单例模式（Singleton）提供一个全局的访问点，和全局变量一样方便，又没有全局变量的缺点（程序一开始就必须创建好对象）。
本文中为了验证单例模式的不同实现，主要利用了`sync`包的锁机制。同时在测试用例上分别设计了单线程与多线程的情况。因此，为了验证多线程环境，在初始化实例时使用`time.Sleep()`模拟阻塞。
并附上**[完整代码](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/creational/singleton)**链接。

<!-- more -->
# 场景分析
> 有一些对象只能有一个实例，如：线程池（threadpool）、缓存（cache）、对话框、处理偏好设置和注册表（registry）的对象、日志对象，充当打印机、显卡等设备的驱动程序的对象。
> 如果创造出多个实例，就会导致许多问题产生，如：程序的行为异常、资源使用过量，或者是不一致的结果。特别是在多线程的执行条件下。
> **弗里曼** --- *<cite>[《Head First 设计模式（中文版）》：chap5 单件模式] [1]</cite>*

## 关键点
- 确保一个类只有一个私有实例
- 提供一个全局访问点

## 线程安全问题
> 如果是多线程的程序，还需要考虑是否线程安全，如果当唯一实例尚未创建时，有两个线程同时调用创建方法，那么它们同时没有检测到唯一实例的存在，从而同时各自创建了一个实例，这样就有两个实例被构造出来，从而违反了单例模式中实例唯一的原则。 解决这个问题的办法是为指示类是否已经实例化的变量提供一个互斥锁(虽然这样会降低效率)。
> **维基百科** --- <cite>[单例模式] [2]</cite>

# Go源码涉及该模式的内容
> The `net/http` package has `http.DefaultClient` and `http.DefaultServeMux` objects that are alive during the application lifecycle. The `DefaultClient` is used by `Get`, `Head` and `Post` functions to send request to http server.
> Those variables contains a single instances that can be used accros the application. The implementation does not follow the same as most of the mainstream language. It’s still Golang idiomatic.
> **Svetlin Ralchev** --- <cite>[Design Patterns in Golang: The Good, the Bad and the Ugly](http://blog.ralch.com/tutorial/design-patterns/golang-design-patterns/)</cite>

# UML描述
注意，Go语言以包作为管理单位：包内的变量根据首字母大小写**只对包外执行访问控制**。

<div style="width: 300px; margin: auto">

![type simple struct](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/creational_singleton.jpg)
</div>

# 样例代码
代码实现了完整的过程，由于篇幅限制，这里只贴出`func GetInstance() *Singleton`的细节。同时，代码为每个实现都设计了单线程与多线程的测试用例以说明其问题。
**[完整代码](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/creational/singleton)**

## 懒汉模式
```go
func GetSimpleInstance() *simple {
    if simpleInstance == nil {
        // delay init
        time.Sleep(5e9)
        simpleInstance = new(simple)
    }
    return simpleInstance
}
```

## 线程安全
在对实例（临界资源）初始化时，仅仅对其进行检查是不够的，在多线程环境下，当线程1正在初始化而未初始化完毕时，线程2依旧可以进入该区域执行初始化操作，因此还需要通过对该临界区域加锁实现互斥访问。

### 饿汉模式
```go
func GetLockInstance() *lock {
    instanceMutex.Lock()//试锁定造成了资源的浪费
    defer instanceMutex.Unlock()
    if lockInstance == nil {
        // delay init
        time.Sleep(5e9)
        lockInstance = new(lock)
    }
    return lockInstance
}
```

### check-lock-check
饿汉模式使用锁的方式解决互斥访问问题，每次都需要进行试锁定造成了资源的浪费，使用`check-lock-check`模式提升性能。
```go
func GetCheckInstance() *check {
    if checkInstance == nil {//效率比试锁定高
        checkMutex.Lock()
        defer checkMutex.Unlock()
        if checkInstance == nil {
            // delay init
            time.Sleep(5e9)
            checkInstance = new(check)
        }
    }
    return checkInstance
}
```

### `sync.Once`
Go中提供了一个`sync.Once`的一次性锁，推荐使用这种方法。
```go
var once sync.Once

func GetOneInstance() *one {
    once.Do(func() {
        // delay init
        time.Sleep(5e9)
        oneInstance = new(one)
    })
    return oneInstance
}
```

## 测试用例
依旧以懒汉模式的单例模式为例贴出代码：
- 使用`go test -v -test.run TestSimpleSingleton`执行测试，会发现多线程的用例中产生了两个实例而不是一个。
- 同时，在利用`go test -race`时会检测到基于条件判断的方法会出现`race condition detected`的情况。
- 本文中，为了描述单例模式的不同实现进行了多种尝试，推荐使用`sync.Once`。

```go
func TestSimpleSingleton(t *testing.T) {
    var globalwg sync.WaitGroup
    globalwg.Add(2)
    // one thread
    go t.Run("one", func(t *testing.T) {
        s1 := GetSimpleInstance()
        s2 := GetSimpleInstance()
        if s2 == s1 {
            // s2.AddOne()
            t.Logf("instance1=%p,instance2=%p\n", s1, s2)
        } else {
            t.Errorf("instance1=%p,instance2=%p\n", s1, s2)
        }
        s2 = GetSimpleInstance()
        if s2 == s1 {
            // s2.AddOne()
            t.Logf("instance1=%p,instance2=%p\n", s1, s2)
        } else {
            t.Errorf("instance1=%p,instance2=%p\n", s1, s2)
        }
        t.Logf("value=%d\n", s1.GetValue())
        globalwg.Done()
    })

    // multi thread
    go t.Run("multi", func(t *testing.T) {
        s1 := GetSimpleInstance()
        var wg sync.WaitGroup
        for i := 0; i < 20; i++ {
            wg.Add(1)
            go func(i int) {
                defer wg.Done()
                s := GetSimpleInstance()
                if s != s1 {
                    t.Errorf("instance1=%p,instance%d=%p\n", s1, i, s)
                } else {
                    // s.AddOne()
                    t.Log(i)
                }
                time.Sleep(1e9)
            }(i)
        }
        wg.Wait()
        t.Logf("value=%d\n", s1.GetValue())
        globalwg.Done()
    })
    globalwg.Wait()
}
```

# 参考资料
[1]: https://bookset.me/5123.html "《Head First 设计模式（中文版）》：chap5 单件模式"
[2]: https://zh.wikipedia.org/wiki/单例模式 "维基百科：单例模式"
- [Design Patterns in Golang: Singleton](http://blog.ralch.com/articles/design-patterns/golang-singleton/)
- [《Head First 设计模式（中文版）》：chap5 单件模式](https://bookset.me/5123.html)
- [维基百科：单例模式](https://zh.wikipedia.org/wiki/单例模式)
