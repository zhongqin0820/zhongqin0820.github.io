---
title: Go语言进阶总结：与并发编程相关标准库内容学习
date: 2019-06-27 16:45:56
updated: 2019-07-01 20:48:20
categories:
- Go语言进阶

tags:
- Go
---
# 前言
学习使用包括`os`，`os/exec`，`runtime`，`sync`，`sync/atmoic`，`os/signal`，`context`等在内的多个标准库。

<!-- more -->
# 进程管理
进程调度算法通过变更进程的状态，使CPU资源的利用率最大化。对进程的管理涉及进程从创建，执行，销毁的全过程。
涉及标准库，包括：
- [pkg/os](https://golang.google.cn/pkg/os/)：提供了不依赖平台的操作系统函数的接口.
- [pkg/os/exec](https://golang.google.cn/pkg/os/exec/)：执行外部命令.

## 创建进程
在Go语言中，Linux下创建进程使用的系统调用是`clone`。而与进程相关的结构体为：`os.Process`，通过如下方式可以创建一个进程。
- `os`：`func StartProcess(name string, argv []string, attr *ProcAttr) (*Process, error)`是一个低级别的接口
- `os/exec`：`type Cmd struct`的相关方法，是一个高级别的接口封装。

```go
// exec.Cmd
// 通过 exec.Command 函数产生 Cmd 实例
func Command(name string, arg ...string) *Cmd
// 调用 Run()，它内部会先调用 Start()，接着调用 Wait()；
func (c *Cmd) Run() error
// 调用 Start()，接着调用 Wait()，然后会阻塞直到命令执行完成；
func (c *Cmd) Start() error
func (c *Cmd) Wait() error
// Output() 是 Run() 的简便写法，外加获取外部命令的输出
func (c *Cmd) Output() ([]byte, error)
// CombinedOutput() 组合 Stdout 和 Stderr 的输出
func (c *Cmd) CombinedOutput() ([]byte, error)
func (c *Cmd) StderrPipe() (io.ReadCloser, error)
func (c *Cmd) StdinPipe() (io.WriteCloser, error)
func (c *Cmd) StdoutPipe() (io.ReadCloser, error)
```

> 一般应该尽量使用 `os/exec` 包

## 销毁进程
- `os.Exit(code int)`

# Goroutine
在并发编程中，同步是在最大化利用CPU资源时，同时保证执行单位（线程/进程）互斥访问临界资源的目的。线程/进程通信机制是达到同步的手段。
在Go的并发调度模型中，最小的执行单位是协程（`goroutine`），使用`go`关键字开启一个新的协程。`goroutine`是非常轻量的，除了给它分配**栈空间（4k）**，它所占用的内存空间是微乎其微的。
Go倾向使用基于**管道channel的消息通信模型**代替传统的基于**共享内存的锁机制**进行协同操作。

## sync包
- [pkg/sync](https://golang.google.cn/pkg/sync/)；提供了互斥锁这类的基本的同步原语

- 锁机制
```go
// Locker接口代表一个可以加锁和解锁的对象
type Locker interface

// 互斥锁
type Mutex struct
    func (m *Mutex) Lock()
    func (m *Mutex) Unlock()
// 读写锁
type RWMutex struct
    func (rw *RWMutex) Lock()           // 写锁
    func (rw *RWMutex) Unlock()
    func (rw *RWMutex) RLock()          // 读锁
    func (rw *RWMutex) RUnlock()
    func (rw *RWMutex) RLocker() Locker
```

- 其它操作
```go
// Cond实现了一个条件变量，一个线程集合地，供线程等待或者宣布某事件的发生。
// Cond需要配合锁机制使用
type Cond struct
    func NewCond(l Locker) *Cond
    func (c *Cond) Broadcast()          // 发送通知（广播）
    func (c *Cond) Signal()             // 发送通知（单发）
    func (c *Cond) Wait()               // 等待通知
// Once是只执行一次动作的对象
type Once struct
    func (o *Once) Do(f func())
// 等待组，对多个goroutine的管理
type WaitGroup struct
    func (wg *WaitGroup) Add(delta int)
    func (wg *WaitGroup) Done()
    func (wg *WaitGroup) Wait()
```

- 资源管理
```go
// 临时对象池，需要实现一个对象的New func() interface{}签名
type Pool struct
    func (p *Pool) Get() interface{}
    func (p *Pool) Put(x interface{})
// 线程安全Map
type Map
    func (m *Map) Delete(key interface{})
    func (m *Map) Load(key interface{}) (value interface{}, ok bool)
    func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
    func (m *Map) Range(f func(key, value interface{}) bool)
    func (m *Map) Store(key, value interface{})
```

- [pkg/sync/atomic](https://golang.google.cn/pkg/sync/atomic/)：提供了底层的原子性内存原语，这对于同步算法的实现很有用
- [pkg/os/signal](https://golang.google.cn/pkg/os/signal/)：实现了对输入信号的访问
    - `func Notify(c chan<- os.Signal, sig ...os.Signal)`
    - `func Stop(c chan<- os.Signal)`

# 学习资料
- 源码地址
    - [pkg/context](https://golang.google.cn/pkg/context/)
    - [pkg/sync](https://golang.google.cn/pkg/sync/)
    - [pkg/sync/atomic](https://golang.google.cn/pkg/sync/atomic/)
    - [pkg/os/exec](https://golang.google.cn/pkg/os/exec/)
    - [pkg/os/signal](https://golang.google.cn/pkg/os/signal/)
- 《浅谈 Go 语言实现原理》
    - [3.4 Channel](https://draveness.me/golang/datastructure/golang-channel.html)
    - [5.1 上下文 Context](https://draveness.me/golang/concurrency/golang-context.html)
    - [5.2 同步原语与锁](https://draveness.me/golang/concurrency/golang-sync-primitives.html)
- 《Go语言标准库》
    - [10.0 进程、线程和 goroutine](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter10/10.0.md)
    - [16.1 sync - 处理同步需求](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter16/16.01.md)
    - [16.2 sync/atomic - 原子操作](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter16/16.02.md)
    - [16.3 os/signal - 信号](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter16/16.03.md)

[//]: # (TODO:Zoking:2019/07/25)
[//]: # (待添加context和runtime包相关内容)
[//]: # (DUE:2019/08/10)

# Changlog
- 2019/07/01：基本完善内容格式
- 2019/07/08：添加针对channel于goroutine的底层模型的原理分析，《浅谈 Go 语言实现原理》[3.4 Channel](https://draveness.me/golang/datastructure/golang-channel.html)
- 2019/07/25：将channel部分内容移除，重新划分`sync`包内容
