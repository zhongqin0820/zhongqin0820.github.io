---
title: Go语言入门总结：基础数据结构
date: 2018-06-22 11:47:10
updated: 2019-07-06 21:51:03
categories:
- Go语言入门

tags:
- Go
---
# 前言
梳理总结学习Go语言的笔记。关于内置基础数据结构（数组、切片、映射），以及补充关于`container`包的一些说明。

<!-- more -->
# 数组`array`
> 需要注意的是，**数组的长度属于数组类型的一部分。**具有相同元素类型，但长度不相等的数组属于不同的类型。
> 可对数组进行读/写，但不可对其进行增删操作

## 声明/定义与初始化
```go
// 不同的创建方式，注意初始化时候的数组长度必须一致
var a [LEN]type         //作用等同于var a [LEN]type = [LEN]type{}
var a [LEN]type = [LEN]type{}
var a [LEN]type = [...]type{ele1, ele2, ele3}
var a [LEN]type = [...]type{LEN-1: eleN}
// 其它声明并初始化方式
var a = [LEN]type{}     // 编译器自动推导出类型
a := [LEN]type{}        // 短变量声明，仅可作用于局部
```

## 操作
### `for range`表达式
- 用于数组`array`、切片`slice`、哈希结构`map`元素的读
    - `range`表达式遍历有两个返回值，第一个是索引下标/键值，第二个是元素的值
- 用于通道`chan`元素的读
    - `range`得到的内容是按照`FIFO`的元素
    - 使用`range`读通道元素前需要关闭通道，否则会阻塞/panic

### 索引下标
```go
for i:=0;i<len(s);i++ {
    // s[i]
}
```

### 值传递与引用传递
数组作为函数的参数仍然是值传递，虽然可以使用数组的指针来代替，但是改变不了数组长度。这时，我们通常使用切片`slice`来替代数组。
**数组是定长的，而切片则是不定长的。前者是值拷贝，后者通过传递地址实现引用拷贝**

# 切片`slice`
- 三种创建方式：
    - 基于底层数组创建：`aSlice := aArray[:]`
    - 直接创建：`aSlice := []Type{}`
    - 使用`make()` 函数创建：`aSlice := make([]Type,0,CAPSIZE)`

- `slice`是对底层数组的抽象和控制。
    - 源码地址v1.12：[pkg/reflect/#SliceHeader](https://golang.org/pkg/reflect/#SliceHeader)
    - 它包含 `Go`需要对底层数组管理的三种元数据，分别是：

```go
type SliceHeader struct {
    Data uintptr //指向底层数组的指针
    Len  int     //slice中元素的长度：内置的len()函数获取长度
    Cap  int     //slice的容量(可供增长的最大值)：内置的cap()函数获取容量；当Len<1024时成倍增长，否则增长25%
}
```

## 声明/定义与初始化
### `make()`函数
函数`make()`可以用于灵活的创建数组切片。`make()`函数创建一个指定元素类型、长度和容量的`slice`。**容量部分可以省略，在这种情况下，容量将等于长度。**

```go
// 声明，需要对其初始化后才可使用，否则其值为nil
var s []type
// 声明并初始化，注意此处长度应置为0
var s = []type{}
var s = make([]type, 0, CAPSIZE)
// 短变量声明，仅可作用于局部
s := []type{}
s = make([]type, 0, CAPSIZE)
```

#### 与`new()`函数的区别
- `new()`：用来初始化一个对象，并且返回该对象的首地址．其自身是一个指针．可用于初始化任何类型
- `make()`：返回一个初始化的实例，返回的是一个实例，而不是指针，其只能用来初始化：slice, map和channel三种类型

## 操作
### 追加元素
内置的`append()`函数用于向`slice`追加元素。可以直接追加元素，也可以追加一个`slice`。**注意参数`slice`后有`...`。否则有语法错误。**
因为`append()`函数的语义是从第二个参数开始都应该是待附加的元素。`slice`后加`...`意味将`slice`的元素全部打散后传入。数组切片会自动处理存储空间不足的问题。如果追加的内容长度超过当前已分配的存储空间（即`cap()`调用返回的信息），**数组切片会自动分配一块足够大的内存**。
> 当`len<1024`时，成倍扩增；当`len>1024`，按25%扩增
> **注意，`append`操作是值拷贝，即产生的切片与原切片不共享底层数组。**另外，如果`make`操作分配了长度，则`append`是在末尾`len`后添加。

### 删除元素
- `s = append(s[:i],s[i+1:]...)`：指定删除切片`s[i]`
- `s = s[i:j]`：截取$ \left [ i,j \right ) $ 范围的元素，即删除$\left [ 0,i \right ) \bigcup \left [ j,len \left ( s \right ) \right )$

### 复制
内置的`func copy(dst, src []Type) int`函数用于数组切片的复制(深拷贝)。复制时使用者无需考虑目标数组和源数组的长度。

## 其它
### 高维切片的操作
针对高维切片的操作需要将其与一维操作进行区分，使用`make`对其进行初始化时，需要嵌套对其进行操作。

### 切片不能比较
**切片可遍历，可修改，不可比较**；与数组不同的是`slice`之间不能比较，因此我们不能使用`==`操作符来判断两个`slice`是否含有全部相等的元素。
不过标准库提供了高度优化的`bytes.Equal()`函数两个字节型`slice`是否相等，但是对于其它类型的`slice`，我们必须自己展开每个元素进行比较。

### 字符串和`byte`切片
标准库中提供了4个与字符串操作相关的包：

| 包       | 功能                                       |
| ------- | ---------------------------------------- |
| strings | 提供了字符串查询、替换、比较、截断、拆分和合并等功能。              |
| bytes   | 提供了很多与strings包类似的功能。因为字符串是只读的，逐步构建字符串会导致很多分配和复制，这种情况下，使用bytes.Buffer类型将会更有效。 |
| strconv | 提供了布尔类型、整数、浮点数和对应字符串的相互转换，还提供了双引号转义相关的转换。 |
| unicode | 提供了IsDigit、IsLetter、IsUpper和IsLower等功能，用于给字符分类。 |

# 映射`map`
映射通常被称为字典dict或者哈希表hash table。由于Go中字典的底层实现基于hashmap，因此，其是一个**无序的键值对集合**，其中所有的键都是不同的，然后通过给定的键可以在常数时间复杂度内检索、更新或删除对应的值。
- 源码地址v1.12：[pkg/runtime/map.go](https://golang.org/src/runtime/map.go)
- 扩展阅读
    - [理解 Golang 哈希表 Map 的原理](https://draveness.me/golang-hashmap)：推荐阅读
    - [golang map源码详解](https://juejin.im/entry/5a1e4bcd6fb9a045090942d8)

## 声明/定义与初始化
- map的键值类型不可以是`slice`，`chan`，`function` 和 包含这些类型 的 `struct` 不可以作为 `map` 的键，否则会编译错误，如当我们尝试定义`var a map[map[int]int]string`时，会报错：`invalid map key type map[int]int`。
- map的键可以是：**`bool`, 数字，`string`, 指针, `channel`** , 还可以是只 包含前面几个类型的 **接口, 结构体, 数组**
- map类型的变量在声明/定义后，还需要对其进行初始化才可以进行操作。否则报错：`assignment to entry in nil map`。**使用`make`或者字面值是创建`map`惯用的方法。**
- 需要注意，使用多行字面量对其进行初始化或者赋值时，最后一行的逗号不可省略。

## 操作
### 迭代遍历
由于`map`是无序的集合，底层是哈希表实现，所以每次遍历的顺序可能不一样。使用`for range`对其进行迭代遍历，返回`key,value`对。

### 查找元素
利用comma-ok法，判断元素是否在map中。**第二个值是一个布尔类型，用于表示元素是否存在。** 需要与`for range`是产生的`key,value`进行区分。

```go
v, ok := m[key]
if !ok {
    // do something if not found the key
}

// 结合起来使用
if v, ok := m[key]; !ok {
    // do something if not found the key
}

// m[key]法将产生两个值；布尔变量一般命名为ok。
```

### 删除操作
使用内建函数`delete(m, key)`删除元素，没有返回值。
```go
// If m is nil or there is no such element, delete is a no-op.
func delete(m map[Type]Type1, key Type)
```

# 其它
- Slice、Array、Map、Struct字面量的多行书写最后一行的逗号不可省

# container包
包含：heap（接口）、list（结构体）、ring（结构体）

## container/heap
> Package heap provides heap operations for any type that implements `heap.Interface`.
> 考虑到堆接口匿名嵌套sort.Interface接口（包含三个方法），因此，定义一个实现堆接口的结构体需要实现五个方法。
> **默认为最小堆**。

```go
type Interface interface {
    sort.Interface
    // Len() int
    // Less(i, j int) bool
    // Swap(i, j int)
    Push(x interface{}) // add x as element Len()
    Pop() interface{}   // remove and return element Len() - 1.
}
```

> 实现了`heap.Interface`的类型才可以调用下面的**函数**，注意函数的参数

```go
func Fix(h Interface, i int)                //调整第i个节点元素，需要自己手动指定
func Init(h Interface)                      //初始化堆
func Pop(h Interface) interface{}           //弹出堆
func Push(h Interface, x interface{})       //压入堆
func Remove(h Interface, i int) interface{} //删除第i个节点元素
```

## container/list
- list是双向链表，包内有两个结构体：List的节点是Element，真正的值Value（位于Element中）是一个接口，因此在取值的时候需要配合**类型断言**获得真实的值。
- 利用该数据结构可以模拟，常用线性数据结构：链表、栈、队列等

```go
type Element struct {
    next, prev *Element    // 上一个元素和下一个元素
    list       *List       // 元素所在链表
    Value      interface{} // 元素
}

func (e *Element) Next() *Element
func (e *Element) Prev() *Element

type List struct {
    root Element // 链表的根元素
    len  int     // 链表的长度
}

func New() *List
func (l *List) Back() *Element                                     // 最后一个元素
func (l *List) Front() *Element                                    // 第一个元素
func (l *List) Init() *List                                        // 链表初始化
func (l *List) InsertAfter(v interface{}, mark *Element) *Element  // 在某个元素后插入
func (l *List) InsertBefore(v interface{}, mark *Element) *Element // 在某个元素前插入
func (l *List) Len() int                                           // 在链表长度
func (l *List) MoveAfter(e, mark *Element)                         // 把e元素移动到mark之后
func (l *List) MoveBefore(e, mark *Element)                        // 把e元素移动到mark之前
func (l *List) MoveToBack(e *Element)                              // 把e元素移动到队列最后
func (l *List) MoveToFront(e *Element)                             // 把e元素移动到队列最头部
func (l *List) PushBack(v interface{}) *Element                    // 在队列最后插入元素
func (l *List) PushBackList(other *List)                           // 在队列最后插入接上新队列
func (l *List) PushFront(v interface{}) *Element                   // 在队列头部插入元素
func (l *List) PushFrontList(other *List)                          // 在队列头部插入接上新队列
func (l *List) Remove(e *Element) interface{}                      // 删除某个元素
```

## cotntainer/ring
- list.Ring是首尾相连的链表，注意其与list.List一样都是结构体。

```go
type Ring struct {
    next, prev *Ring
    Value      interface{}
}

func New(n int) *Ring                  // 初始化环，需要指定大小
func (r *Ring) Do(f func(interface{})) // 循环环进行操作
func (r *Ring) Len() int               // 环长度
func (r *Ring) Link(s *Ring) *Ring     // 连接两个环
func (r *Ring) Move(n int) *Ring       // 指针从当前元素开始向后移动或者向前（n可以为负数）
func (r *Ring) Next() *Ring            // 当前元素的下个元素
func (r *Ring) Prev() *Ring            // 当前元素的上个元素
func (r *Ring) Unlink(n int) *Ring     // 从当前元素开始，删除n个元素
```

# 结束语
对于引用类型的切片和映射(map)，需要注意对其进行`for range`迭代时，对其进行增删将会影响其遍历的迭代器指向。因此，该过程中应避免增删操作。应该通过关于标准包的学习使用对其进行掌握。

# 参考链接
- [基础数据结构](https://www.jb51.net/article/56828.htm)
- 《Go语言编程》

# 扩展阅读
- 《浅谈 Go 语言实现原理》：[3.1 数组和切片](https://draveness.me/golang/datastructure/golang-array-and-slice.html)
- 《浅谈 Go 语言实现原理》：[3.2 哈希表](https://draveness.me/golang/datastructure/golang-hashmap.html)
- 《浅谈 Go 语言实现原理》：[3.3 字符串](https://draveness.me/golang/datastructure/golang-string.html)

# Changelog
- 2019/06/25：修正格式，添加map部分的源码理解资料链接
- 2019/06/27：完善格式以及文章内容，使结构更加清晰
- 2019/07/06：添加《浅谈 Go 语言实现原理》扩展阅读
- 2019/07/24：修改map的键值的类型
