---
title: Go语言入门总结:面向对象之接口与反射
date: 2019-06-18 10:15:50
updated: 2019-06-19 18:26:46
categories:
- 计算机基础

tags:
- Go
- 面向对象
---
# 前言
梳理总结Go中的类型系统，以及关于接口与反射的进一步知识的梳理。

<!-- more -->
# Go的类型系统
Go的类型包括编译期的静态类型与运行时的具体类型。使用<type value>对表示。

# 接口
- 一个interface值由两个指针组成：
    - 第一个指向一个interface table，叫 itable。itable开头是一些描述类型的元字段，后面是一串方法。注意这个方法是interface本身的方法，并非其dynamic value（Binary）的方法。即这里只有String()方法，而没有Get方法。但这个方法的实现肯定是具体类的方法，这里就是Binary的方法。当这个interface无方法时，itable可以省略，直接指向一个type即可。
    - 另一个指针data指向dynamic value的一个拷贝，这里则是b的一份拷贝。也就是，给interface赋值时，**会在堆上分配内存，用于存放拷贝的值。**同样，当值本身只有一个字长时，这个指针也可以省略。
- 一个interface的初始值是两个nil。interface是否为nil取决于itable字段。所以不一定data为nil就是nil，判断时要额外注意。
- 计算interface对象的itable时，需要对两种类型的方法列表进行遍历对比。itable的计算不需要到函数调用时进行，只需要在interface赋值时进行即可。

## 源码分析
- [golang的interface剖析](https://www.cnblogs.com/qqmomery/p/6298771.html)：总结的十分到位。
```go
type iface struct {
    tab  *itab     // 指向itable
    data unsafe.Pointer     // 指向真实数据
}

type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab//interface类型与具体类型之间的信息结合起来做一个hash的过程，itab中的link就是hash中的拉链。
    bad    int32
    unused int32
    fun    [1]uintptr // variable sized
}
// 如果具体类型的方法签名匹配上该接口，则将<接口类型，具体类型>:<*interfacetype,*_type>对作为键值插入到一个全局hash table中。即：
// hash  [hashSize]*itab
// 使用时，查表即可
```

# reflect
Golang语言实现了反射，反射机制建立在类型系统之上，在运行时动态的调用对象的方法和属性，官方自带的reflect包就是反射相关的，只要包含这个包就可以使用。

## 接口变量到反射对象
反射对象包括：
- `reflect.Type`：可以通过`reflect.TypeOf(接口变量)`得到
- `reflect.Value`：可以通过`reflect.ValueOf(接口变量)`得到

## 反射对象到接口变量值
- 可以通过`reflect.ValueOf(接口变量).Interface()`得到

## 对内容进行修改
- 想要对接口变量值进行修改，需要传入接口变量的地址
- 可以通过`reflect.ValueOf(&接口变量).Elem().Canset()`进行判断

## 针对结构体内容
- 可以通过获取结构体的属性以及方法进行对应操作

```go
type Value struct {}
func (v Value) Field(i int) Value
func (v Value) FieldByIndex(index []int) Value
func (v Value) FieldByName(name string) Value
func (v Value) FieldByNameFunc(match func(string) bool) Value
func (v Value) Method(i int) Value
func (v Value) MethodByName(name string) Value
func (v Value) NumField() int
func (v Value) NumMethod() int
```

# 参考资料
- [golang的interface剖析](https://www.cnblogs.com/qqmomery/p/6298771.html)
- [Part 34: Reflection](https://golangbot.com/reflection/)
- [Golang的反射reflect深入理解和示例](https://juejin.im/post/5a75a4fb5188257a82110544)
- [【翻译】反射的规则](https://studygolang.com/articles/1010)
- [【原文】The Laws of Reflection](https://blog.golang.org/laws-of-reflection)
