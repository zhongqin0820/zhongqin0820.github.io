---
title: Go实现经典算法问题:栈与队列
date: 2019-05-31 20:48:11
updated: 2019-06-03 20:23:22
categories:
- 计算机基础

tags:
- Go
---
# 前言
Go中线性数据结构没有现成的栈与队列。因此，当被问到使用队列实现栈，或者使用栈实现队列的问题时会变得很尴尬。如：[Leetcode 225. Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/)、[Leetcode 232. Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)、[两个栈实现双端队列](https://www.cnblogs.com/zmlctt/p/3985128.html)。

针对这个问题，简单做个梳理。法1：借助切片使用类型重定义后的类型添加新方法封装得到的栈/队列。法2：直接使用container/list中的双向链表的封装。这里最后使用后者解决问题。

<!-- more -->
# 使用切片实现栈与队列
通过对`[]int`进行类型定义并扩充方法集合得到。

## 栈
- 先进后出

```go
type MyStack []int
func NewMyStack() *MyStack { return &MyStack{} }
func (s *MyStack) Push(x int) { *s = append(*s, x) }
func (s *MyStack) Pop() int {
    if s.Empty() {return -1}
    res := (*s)[len(*s)-1]
    *s = (*s)[:len(*s)-1]
    return res
}
func (s *MyStack) Peek() int {
    if s.Empty() {return -1}
    return (*s)[len(*s)-1]
}
func (s *MyStack) Empty() bool { return len(*s)==0 }
```

## 队列
- 先进先出

```go
type MyQueue []int
func NewMyQueue() *MyQueue { return &MyQueue{} }
func (q *MyQueue) EnQueue(x int) { *q = append(*q,x) }
func (q *MyQueue) DeQueue() int {
    if q.Empty() { return -1 }
    res := (*q)[0]
    *q = (*q)[1:]
    return res
}
func (q *MyQueue) Front() int {
    if q.Empty() { return -1 }
    return (*q)[0]
}
func (q *MyQueue) Empty() bool { return len(*q) == 0 }
```

## 使用`container/list`
- list是一个双向链表ADT，可以使用这个特性，模拟为栈和队列。

```go
type Element struct {
    next, prev *Element  // 上一个元素和下一个元素
    list *List  // 元素所在链表
    Value interface{}  // 元素
}

func (e *Element) Next() *Element
func (e *Element) Prev() *Element

type List struct {
    root Element  // 链表的根元素
    len  int      // 链表的长度
}

func New() *List
func (l *List) Back() *Element   // 最后一个元素
func (l *List) Front() *Element  // 第一个元素
func (l *List) Init() *List  // 链表初始化
func (l *List) InsertAfter(v interface{}, mark *Element) *Element // 在某个元素后插入
func (l *List) InsertBefore(v interface{}, mark *Element) *Element  // 在某个元素前插入
func (l *List) Len() int // 在链表长度
func (l *List) MoveAfter(e, mark *Element)  // 把e元素移动到mark之后
func (l *List) MoveBefore(e, mark *Element)  // 把e元素移动到mark之前
func (l *List) MoveToBack(e *Element) // 把e元素移动到队列最后
func (l *List) MoveToFront(e *Element) // 把e元素移动到队列最头部
func (l *List) PushBack(v interface{}) *Element  // 在队列最后插入元素
func (l *List) PushBackList(other *List)  // 在队列最后插入接上新队列
func (l *List) PushFront(v interface{}) *Element  // 在队列头部插入元素
func (l *List) PushFrontList(other *List) // 在队列头部插入接上新队列
func (l *List) Remove(e *Element) interface{} // 删除某个元素
```

### 栈
- 单向进出（使用`PushBack`压栈，使用`Back`获取压栈的元素），**后进先出**。

```go
// 新建栈 *list.List
stack := list.New()

// 压栈
stack.PushBack(element)

// 取栈顶
stack.Back().Value.(ElementType)

// 弹出栈
stack.Remove(stack.Back())

// 判断是否为空
stack.Len() != 0

```

### 队列
- 尾进头出（使用`PushBack`进队列，使用`Front`获取队头的元素），先进先出

```go
// 新建队列 *list.List
queue := list.New()

// 进队列
queue.PushBack(element)

// 取队头元素
queue.Front().Value.(ElementType)

// 出队列
queue.Remove(queue.Front())

// 队列判空
queue.Len() != 0
```

# 面试问题
## 切片的特点与基本操作
- 声明/初始化
    - var s []int   // 声明/定义
    - var s []int = []int{} //声明并初始化
    - var s = []int{} //声明并初始化，自动类型推导
    - s := []int{}  //声明并初始化，自动类型推导，短变量声明**不可以用于全局变量**

- 切片操作
    - s = s[i:] //等价删除i之前的数据，不包括下标i，i=1删除开头，i=len(n)删除整个切片
    - s = s[:j] //等价删除j之后的数据，包括下标j，j=len(n)-1删除末尾，j=0删除整个切片

## 一个队列实现栈
- [Leetcode 225. Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues)
- 解决思路
    - 队列q1负责入栈
    - 队列q2负责辅助出栈，将队列q1倒到q2直到剩下最后一个元素，弹出q1中最后一个元素。将q2中的东西倒回q2。

```go
import "container/list"

type MyStack struct {
    q1 *list.List
    q2 *list.List
}

/** Initialize your data structure here. */
func Constructor() MyStack {
    return MyStack{ q1: list.New(), q2: list.New() }
}


/** Push element x onto stack. */
func (this *MyStack) Push(x int)  {
    this.q1.PushBack(x)
}


/** Removes the element on top of the stack and returns that element. */
func (this *MyStack) Pop() int {
    for ;this.q1.Len() > 1; {
        this.q2.PushBack(this.q1.Remove(this.q1.Front()))
    }
    res := this.q1.Front().Value.(int)
    this.q1.Remove(this.q1.Front())
    for ;this.q2.Len() > 0; {
        this.q1.PushBack(this.q2.Remove(this.q2.Front()))
    }
    return res
}


/** Get the top element. */
func (this *MyStack) Top() int {
    for ;this.q1.Len() > 1; {
        this.q2.PushBack(this.q1.Remove(this.q1.Front()))
    }
    res := this.q1.Front().Value.(int)
    this.q2.PushBack(this.q1.Remove(this.q1.Front()))
    for ;this.q2.Len() > 0; {
        this.q1.PushBack(this.q2.Remove(this.q2.Front()))
    }
    return res
}


/** Returns whether the stack is empty. */
func (this *MyStack) Empty() bool {
    return this.q1.Len() == 0 && this.q2.Len() == 0
}
```

## 两个栈实现一个队列
- [Leetcode 232. Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)
- 解决思路
    - 栈s1负责元素入队列
    - 栈s2负责出队列，当栈s2为空时将栈s1中的元素倒到栈s2中。

```go
import "container/list"
type MyQueue struct {
    s1 *list.List
    s2 *list.List
}


/** Initialize your data structure here. */
func Constructor() MyQueue {
    return MyQueue{ s1: list.New(), s2: list.New() }
}


/** Push element x to the back of queue. */
func (this *MyQueue) Push(x int)  {
    this.s1.PushBack(x)
}


/** Removes the element from in front of queue and returns that element. */
func (this *MyQueue) Pop() int {
    if this.s2.Len() == 0 {
        for ;this.s1.Len()!=0; {
            this.s2.PushBack(this.s1.Remove(this.s1.Back()))
        }
    }
    if this.s2.Len() == 0 {
        return -1
    }
    res := this.s2.Back().Value.(int)
    this.s2.Remove(this.s2.Back())
    return res
}


/** Get the front element. */
func (this *MyQueue) Peek() int {
    if this.s2.Len() == 0 {
        for ;this.s1.Len()!=0; {
            this.s2.PushBack(this.s1.Remove(this.s1.Back()))
        }
    }
    if this.s2.Len() == 0 {
        return -1
    }
    res := this.s2.Back().Value.(int)
    return res
}


/** Returns whether the queue is empty. */
func (this *MyQueue) Empty() bool {
    return this.s1.Len() == 0 && this.s2.Len() == 0
}
```

## 两个栈实现双端队列
- [问题描述](https://www.cnblogs.com/zmlctt/p/3985128.html)
- 解决思路
    - 类似两个栈实现队列
    - 栈s1：压入/弹出队尾
    - 栈s2：压入/弹出队头

```go
import "container/list"
type MyDeQueue struct {
    s1 *list.List
    s2 *list.List
}


/** Initialize your data structure here. */
func Constructor() MyDeQueue {
    return MyDeQueue{ s1: list.New(), s2: list.New() }
}


/** Push element x to the front of queue. */
func (this *MyDeQueue) PushFront(x int)  {
    this.s2.PushBack(x)
}


/** Removes the element from the front of queue and returns that element. */
func (this *MyDeQueue) PopFront() int {
    if this.s2.Len() == 0 {
        for ;this.s1.Len()!=0; {
            this.s2.PushBack(this.s1.Remove(this.s1.Back()))
        }
    }
    if this.s2.Len() == 0 {
        return -1
    }
    res := this.s2.Back().Value.(int)
    this.s2.Remove(this.s2.Back())
    return res
}


/** Push element x to the back of queue. */
func (this *MyDeQueue) PushBack(x int)  {
    this.s1.PushBack(x)
}


/** Removes the element from the back of queue and returns that element. */
func (this *MyDeQueue) PopBack() int {
    if this.s1.Len() == 0 {
        for;this.s2.Len()!=0; {
            this.s1.PushBack(this.s2.Remove(this.s2.Back()))
        }
    }
    if this.s1.Len() == 0 {
        return -1
    }
    res := this.s1.Back().Value.(int)
    this.s1.Remove(this.s1.Back())
    return res
}


/** Get the front element. */
func (this *MyDeQueue) Front() int {
    if this.s2.Len() == 0 {
        for ;this.s1.Len()!=0; {
            this.s2.PushBack(this.s1.Remove(this.s1.Back()))
        }
    }
    if this.s2.Len() == 0 {
        return -1
    }
    res := this.s2.Back().Value.(int)
    return res
}


/** Get the back element. */
func (this *MyDeQueue) Back() int {
    if this.s1.Len() == 0 {
        for;this.s2.Len()!=0; {
            this.s1.PushBack(this.s2.Remove(this.s2.Back()))
        }
    }
    if this.s1.Len() == 0 {
        return -1
    }
    res := this.s1.Back().Value.(int)
    return res
}


/** Returns whether the queue is empty. */
func (this *MyDeQueue) Empty() bool {
    return this.s1.Len() == 0 && this.s2.Len() == 0
}
```

# 结束语
掌握对数据结构的定义与封装实现。以及标准库的使用与熟悉。