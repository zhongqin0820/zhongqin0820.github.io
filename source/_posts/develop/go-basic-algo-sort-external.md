---
title: Go实现经典算法问题：外部排序（单机版与网络版）
date: 2019-07-14 17:23:56
updated: 2019-07-15 22:49:17
categories:
- 计算机基础

tags:
- 经典算法
- Go
---
# 前言
在之前的[博文](https://cvblogs.cn/2019/03/08/develop/go_basic_algo_sort/)中曾经梳理过与经典的内部排序算法相关的内容，如：冒泡、选择、插入、希尔、归并、快排、堆排序等。顾名思义，其需要将数据一次性载入内存中进行排序。
本文将要讨论的是基于分治法的单机版本与网络版本的**并发外部排序算法**，即数据无需一次性载入内存。完整代码参见[Github](https://github.com/zhongqin0820/coding-playground/tree/master/go/concurrent/classic/sorts)。

<!-- more -->
# 场景描述
- 已知有一个大小为8m byte的二进制数据文件`large.in`，内含数据类型为`int`（64位机器上为8byte），即共有1m个数据待排
- 已知内存无法一次性加载全部数据，现要求对其进行排序，并将结果输出到文件`large.out`

## 关键点
- 单个节点排序时使用内部排序算法，使用`pipeline`模式，通过通道传输数据
- 多个节点合并时，使用`barrier`模式收集结果

# 示意图
<div style="width: 300px; margin: auto">

![Pipeline](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/develop/go/algo-external-sort.png)
</div>

# 样例代码
## 代码实现
完整代码参见[Github](https://github.com/zhongqin0820/coding-playground/tree/master/go/concurrent/classic/sorts)
### 数据生成
- 随机生成指定需求的数据，并写入二进制文件`large.in`中

```go
// filename是文件名，count是数据个数，此处为1m，即：8m / 8
func GenData(filename string, count int) {
    // 打开文件准备写入
    file, err := os.Create(filename)
    if err != nil {
        panic(err)
    }
    defer file.Close()
    // 并发生成count个随机数
    p := RandomSource(count)
    // 使用bufio优化写操作
    writer := bufio.NewWriter(file)
    // 写入文件largin.in
    WriterSink(writer, p)
    defer writer.Flush()
}

// 并发生成count个随机数
func RandomSource(count int) <-chan int {
    out := make(chan int, CHANSIZE)
    go func() {
        for i := 0; i < count; i++ {
            out <- rand.Int()
        }
        close(out)
    }()
    return out
}

// 将RandomSource中产生的数据，通过管道传输，写入二进制文件中
func WriterSink(writer io.Writer, in <-chan int) {
    for v := range in {
        buffer := make([]byte, 8)
        binary.BigEndian.PutUint64(buffer, uint64(v))
        writer.Write(buffer)
    }
}
```

### 数据读取
- 将大文件按照内存需求分成若干块读入。
- 传统的实现中，在对大数据文件进行分割成小文件进行内部排序后，需要再从各个结果小文件部分载入部分数据到内存中，并留出一定的空间对结果进行收集并写入最终结果文件。而利用管道技术，对内存的限制只在块的大小划分部分，归并部分的内存占用情况由管道的缓冲大小控制。

```go
// 按照chunkSize分块读取数据
func ReaderSource(reader io.Reader, chunkSize int) <-chan int {
    out := make(chan int, CHANSIZE)
    go func() {
        buffer := make([]byte, 8)
        bytesRead := 0
        // 读取chunkSize大小的数据到通道中
        for {
            n, err := reader.Read(buffer)
            bytesRead += n
            if n > 0 {
                v := int(binary.BigEndian.Uint64(buffer))
                out <- v
            }
            if err != nil || (chunkSize != -1 && bytesRead >= chunkSize) {
                break
            }
        }
        close(out)
    }()
    return out
}
```

### 单机版本
#### 单个节点排序
```go
// definition of pipeline
func InMemSort(in <-chan int) <-chan int {
    out := make(chan int, CHANSIZE)
    go func() {
        // Read into Memory
        a := []int{}
        for v := range in {
            a = append(a, v)
        }
        log.Println("Read done: ", time.Now().Sub(startTime))
        // Sort
        sort.Ints(a)
        log.Println("InMemSort done: ", time.Now().Sub(startTime))
        // Output
        for _, v := range a {
            out <- v
        }
        close(out)
    }()
    return out
}
```

#### 两个节点归并

```go
func Merge(in1, in2 <-chan int) <-chan int {
    out := make(chan int, CHANSIZE)
    go func() {
        v1, ok1 := <-in1
        v2, ok2 := <-in2
        for ok1 || ok2 {
            if !ok2 || (ok1 && v1 < v2) {
                out <- v1
                v1, ok1 = <-in1
            } else {
                out <- v2
                v2, ok2 = <-in2
            }
        }
        close(out)
        log.Println("Merge done: ", time.Now().Sub(startTime))
    }()
    return out
}
```

#### 多个节点分割
```go
func MergeN(inputs ...<-chan int) <-chan int {
    if len(inputs) == 1 {
        return inputs[0]
    }
    m := len(inputs) / 2
    return Merge(MergeN(inputs[:m]...), MergeN(inputs[m:]...))
}
```

### 网络版本
只需实现数据读与写。
```go
// addr为服务地址，根据地址读取数据到通道中
func NetworkSource(addr string) <-chan int {
    out := make(chan int, CHANSIZE)
    go func() {
        conn, err := net.Dial("tcp", addr)
        if err != nil {
            panic(err)
        }
        r := ReaderSource(conn, -1)
        for v := range r {
            out <- v
        }
        close(out)
    }()
    return out
}

// addr为服务地址，in为待排数据
func NetworkSink(addr string, in <-chan int) {
    listener, err := net.Listen("tcp", addr)
    if err != nil {
        panic(err)
    }

    go func() {
        defer listener.Close()

        conn, err := listener.Accept()
        if err != nil {
            panic(err)
        }
        defer conn.Close()

        writer := bufio.NewWriter(conn)
        defer writer.Flush()
        WriterSink(writer, in)
    }()
}
```

### 结果保存
```go
// 将排序结果通过管道并发写入结果文件中
func WriteToFile(p <-chan int, fileName string) {
    file, err := os.Create(fileName)
    if err != nil {
        panic(err)
    }
    defer file.Close()

    writer := bufio.NewWriter(file)
    defer writer.Flush()
    WriterSink(writer, p)
}
```

## 测试代码
### 小数据集
测试`func ReaderSource(reader io.Reader, chunkSize int) <-chan int `
```go
func TestReaderSource(t *testing.T) {
    const (
        fileName   = "small.in"
        outfile    = "small.out"
        n          = 64
        chunkCount = 2
        chunkSize  = n * 8 / chunkCount
    )
    // 生成数据
    GenData(fileName, n)
    // 初始化计时器
    Init()

    // 两个节点的合并
    sortResults := []<-chan int{}
    for i := 0; i < chunkCount; i++ {
        file, err := os.Open(fileName)
        if err != nil {
            panic(err)
        }

        // 分块读取数据
        file.Seek(int64(i*chunkSize), 0)
        source := ReaderSource(bufio.NewReader(file), chunkSize)
        // 单个节点排序
        sortResults = append(sortResults, InMemSort(source))
    }
    // 并发归并结果
    p := MergeN(sortResults...)
    // 写入结果文件
    WriteToFile(p, outfile)
    PrintFile(outfile)
}
```

### 大数据集
```go
const (
    fileName   = "large.in"
    fileSize   = 80000000
    chunkCount = 4
    outfile1   = "large1.out"
    outfile2   = "large2.out"
)
// 按照要求生成数据
pipeline.GenData(fileName, fileSize/8)

// 单机版本并发排序
p := external.LaunchPipeline(fileName, fileSize, chunkCount)
pipeline.WriteToFile(p, outfile1)
pipeline.PrintFile(outfile1)

// 网络版本并发排序
p = external.LaunchNetworkPipeline(fileName, fileSize, chunkCount)
pipeline.WriteToFile(p, outfile2)
pipeline.PrintFile(outfile2)
```

# 扩展
- 可以使用`workers pool`模式对资源进行管理

# 学习资料
- 面试中的大数据排序问题
    - [大数据算法：对5亿数据进行排序](https://blog.csdn.net/lemon_tree12138/article/details/48783535)介绍了分治法与字典树算法
    - [海量数据排序——如果有1TB的数据需要排序，但只有32GB的内存如何排序处理](https://blog.csdn.net/FX677588/article/details/72471357)
- 博客/出版物
    - ccmouse：[搭建并行处理管道，感受GO语言魅力](https://www.imooc.com/learn/927)
    - [Parallel Merge Sort in Go](https://hackernoon.com/parallel-merge-sort-in-go-fe14c1bc006)
- 其它代码实现
    - [oxtoacart/emsort](https://github.com/oxtoacart/emsort)
    - [azrle/go-external-sort](https://github.com/azrle/go-external-sort)
    - [siddhant2408/external-merge-sort](https://github.com/siddhant2408/external-merge-sort)

# Changelog
- 2019/07/15：第一版内容
