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
学习使用包括`context`，`os`，`os/exec`，`sync`，`sync/atmoic`，`os/signal`等在内的多个标准库。

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

# 同步
在并发编程中，同步是在最大化利用CPU资源时，同时保证执行单位（线程/进程）互斥访问临街资源的目的。线程/进程通信是达到同步的手段。
在Go的并发模型中，最小的执行单位是`goroutine`，使用`go`关键字开启一个新的协程。`goroutine`是非常轻量的，除了给它分配**栈空间（4k）**，它所占用的内存空间是微乎其微的。Go倾向使用基于**管道channel的消息通信模型**代替传统的基于**共享内存的锁机制**进行协同操作。

## sync包
- [pkg/sync](https://golang.google.cn/pkg/sync/)；提供了互斥锁这类的基本的同步原语

```go
// Locker接口代表一个可以加锁和解锁的对象
type Locker interface
// Once是只执行一次动作的对象
type Once struct
func (o *Once) Do(f func())
// 互斥锁
type Mutex struct
func (m *Mutex) Lock()
func (m *Mutex) Unlock()
// 读写锁
type RWMutex struct
func (rw *RWMutex) Lock()
func (rw *RWMutex) Unlock()
func (rw *RWMutex) RLock()
func (rw *RWMutex) RUnlock()
func (rw *RWMutex) RLocker() Locker
// Cond实现了一个条件变量，一个线程集合地，供线程等待或者宣布某事件的发生。
type Cond struct
func NewCond(l Locker) *Cond
func (c *Cond) Broadcast()
func (c *Cond) Signal()
func (c *Cond) Wait()
// 等待组，对多个goroutine的管理
type WaitGroup struct
func (wg *WaitGroup) Add(delta int)
func (wg *WaitGroup) Done()
func (wg *WaitGroup) Wait()
// 临时对象池，需要实现一个对象的New func() interface{}签名
type Pool struct
func (p *Pool) Get() interface{}
func (p *Pool) Put(x interface{})
```

- [pkg/sync/atomic](https://golang.google.cn/pkg/sync/atomic/)：提供了底层的原子性内存原语，这对于同步算法的实现很有用
- [pkg/os/signal](https://golang.google.cn/pkg/os/signal/)：实现了对输入信号的访问
    - `func Notify(c chan<- os.Signal, sig ...os.Signal)`
    - `func Stop(c chan<- os.Signal)`

## channel
channel是基于消息通信的同步方式。
### channel的原理
- 在对channel原理进行讲解前，应该需要对Go的并发协程模型（MPG模型/两级线程模型）有一定的认识，channel的数据传递优化依赖于该模型
- 源码地址v1.12：[src/runtime/chan.go](https://golang.org/src/runtime/chan.go)
- [GopherCon 2017: Kavya Joshi - Understanding Channels](https://www.youtube.com/watch?v=KBZlN0izeiY)
    - hchan：buf（环形队列），lock，recvx，sendx，【recvq，sendq】与并发操作相关
    - sudoq：双向链表结构，元素是goroutine
    - 每次读/写操作都需要使用`lock`加锁
    - `goroutine阻塞`：向满的channel写/从空的channel读
    - `goroutine唤醒`：空的channel中被写入数据/满的channel中被取走数据【利用sudog数据结构，直接互相读/写值】

### channel的坑
- channel必须带数据类型，可以先声明channel类型的变量，但在使用前，需要使用`make()`对其进行赋值，同时指定缓冲长度，即通道中允许排队的元素个数
- 单向通道通常作为函数的参数，在变量声明中是不应该出现单向通道的，因为通道本来就是为了通信而生，只能接收或者只能发送数据的通道是没有意义的。
- 程序中必须同时有不同的 goroutine 对非缓冲通道进行发送和接收操作，否则会造成死锁。
- 如果写完需使用`close()`关闭channel，否则利用`for range`形式迭代会死锁
- 如果想要获取通道内元素的个数，可以直接使用内置函数`len()`
- `select` 专门用于通道发送和接收操作，看起来和 `switch` 很相似，但是进行选择和判断的方法完全不同，`select`是随机选择就绪的通道，`switch`是根据条件判断进行选择。
- 使用非缓冲通道代替传统同步机制（互斥锁等），从而保证`goroutine`在`main.goroutine`退出前执行完成。

### channel常用代码思想
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

# 学习资料
- 源码地址
    - [pkg/context](https://golang.google.cn/pkg/context/)
    - [pkg/sync](https://golang.google.cn/pkg/sync/)
    - [pkg/sync/atomic](https://golang.google.cn/pkg/sync/atomic/)
    - [pkg/os/exec](https://golang.google.cn/pkg/os/exec/)
    - [pkg/os/signal](https://golang.google.cn/pkg/os/signal/)
- 《浅谈 Go 语言实现原理》
    - [5.1 上下文 Context](https://draveness.me/golang/concurrency/golang-context.html)
    - [5.2 同步原语与锁](https://draveness.me/golang/concurrency/golang-sync-primitives.html)
- 《Go语言标准库》
    - [10.0 进程、线程和 goroutine](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter10/10.0.md)
    - [16.1 sync - 处理同步需求](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter16/16.01.md)
    - [16.2 sync/atomic - 原子操作](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter16/16.02.md)
    - [16.3 os/signal - 信号](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/blob/master/chapter16/16.03.md)

# Changlog
- 2019/07/01：基本完善内容格式

# TODO
- [ ] 2019/07/01：添加针对channel于goroutine的底层模型（MPG）的原理分析