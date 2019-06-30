---
title: Go语言入门总结：面向对象
date: 2019-05-27 14:58:20
updated: 2019-06-24 11:44:44
categories:
- Go语言入门

tags:
- Go
- 面向对象
---
# 前言
梳理总结学习Go语言的笔记。关于Go语言中的面向对象编程，包括：OOP基本概念，UML类图以及在Go中如何进行OOP。添加关于类型查询、类型断言内容的梳理。

<!-- more -->
# OOP基本概念
- 具备可复用、可扩充、可维护
- 设计模式不是代码，而是针对特定问题的解决方案，你可以将其应用到特定的应用中。
- 设计过程中，应该着眼于系统中变化的部分，将起抽出来封装

## 抽象
- Go使用接口对问题进行抽象，接口是一组方法的签名，结构体类型只需实现该组方法即可说明该结构体实现了这个接口（在运行时通过枚举方法签名，判断是否赋值给接口变量的结构体实例类型实现了该接口的方法）

### 设计原则
- 鸭子类型的设计原则是面向接口编程，而不是针对实现编程。
- 因此，接口应该是对**行为**的抽象。具体的行为类型实现该行为接口。
- 完整的类型通过组合/聚合/关联【行为接口变量】，该行为接口变量会在运行时持有具体的行为类型实例的引用。
- 完整的类型通过新增执行对应操作的方法，在其中利用行为接口变量执行行为接口中的抽象行为方法。

## 封装
- 在Go中使用结构体定义类型，可以对一组属性及方法进行封装。
- 以包为级别的操作权限，大写表示外部可以访问，小写则仅包内可以访问。
- 使用`NewT(parameters)`函数新建对象实例代替类的构造函数

### 设计原则
- 封装系统中变化的部分

## 继承
在Go中没有类的概念，但是结构体可以通过匿名嵌套的方式扩展方法集合以及属性。

> 需要注意的是，存在同名方法时，直接调用会覆盖被嵌套结构体的方法。

### 设计原则
- 应该多用组合，少用继承

## 多态
静态类型语言，如C++的多态分为：
- 编译期
    - 函数重载
    - 模板

- 运行时
    - 虚函数：虚函数表、虚函数指针

在Go中利用**接口**定义的方法集合，结构体对方法集合进行实现。在运行时，对使用了接口的函数绑定对应的结构体实例，从而达到实现运行时多态的目的。这也被叫做鸭子类型（duck typing）。

### 设计原则
- 针对接口编程，不针对实现编程：即抽象行为接口

## UML类图
- 继承关系
    - 泛化(Generalization)：是一种继承关系，表示一般与特殊的关系，它指定了子类如何继承父类的所有特征和行为。如：鸟类泛化动物类
    - 实现(Realization)：类与接口的关系，表示类是接口所有特征和行为的实现，如大雁、鸭子、企鹅类实现飞翔接口
- 包含关系：【一般为成员属性】
    - 聚合(Aggregation)：弱相关，是整体与部分的关系，且部分可以离开整体而单独存在。如：大雁类聚合雁群类
    - 组合(Composition)：强相关，是整体与部分的关系，但部分不能离开整体而单独存在。如：鸟类组合翅膀类
    - 关联(Association)：是一种拥有的关系，它使一个类知道另一个类的属性和方法。如：企鹅关联气候；注：在最终代码中，关联对象通常是以成员变量的形式实现的
- 使用关系
    - 依赖(Dependency)：是一种使用的关系，即一个类的实现需要另一个类的协助，所以要尽量不使用双向的互相依赖。如：动物类的`新陈代谢(氧气 x, 水 y)`依赖氧气类与水类

![类图关系实例](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/%E7%B1%BB%E5%9B%BE%E5%85%B3%E7%B3%BB.png)

# Go中的OOP实例
最简单的就是实现一个[工厂方法](https://github.com/tmrts/go-patterns/blob/master/creational/factory.md)：

## 接口定义
```go
package data

import "io"

type Store interface {
    Open(string) (io.ReadWriteCloser, error)
}
```

## 类型定义
```go
package data

type StorageType int

const (
    DiskStorage StorageType = 1 << iota
    TempStorage
    MemoryStorage
)

func NewStore(t StorageType) Store {
    switch t {
    case MemoryStorage:
        return newMemoryStorage( /*...*/ )
    case DiskStorage:
        return newDiskStorage( /*...*/ )
    default:
        return newTempStorage( /*...*/ )
    }
}
```

## 使用
```go
s, _ := data.NewStore(data.MemoryStorage)
f, _ := s.Open("file")

n, _ := f.Write([]byte("data"))
defer f.Close()
```

### 类型查询、类型断言
- 类型查询

```go
type instanceType struct {
    fisrtname string
    lastname string
}
// type 关键字 查询接口变量的具体类型
var instance interface{} = instanceType{}
// 注意case之间的条件不可以重复
switch instance.(type) {
    case int:
        // to do something
    case string:
        // to do something
    case instanceType:
        // to do something
    default:
        // to do something
}

// reflect 包 查询接口变量的具体类型
var getType reflect.Type = reflect.TypeOf(instance)
var getValue reflect.Value = reflect.ValueOf(instance)
concreteType := getValue.Type()
```

- 类型断言

```go
type instanceType struct {
    fisrtname string
    lastname string
}

// 已知数据类型进行断言

// 定义及初始化接口实例：
var instance interface{} = instanceType{"xx","yy"}
// 接口实例进行类型断言得到接口中的类型实例：
var value instanceType = instance.(instanceType)
// 类型实例调用（获取具体值）：
value

// 未知数据类型进行断言(reflect包)

// 获取接口中的实例的类型
reflect.ValueOf(instance).Type()//或 reflect.TypeOf(instance)
// 获取接口中的实例内容：
reflect.ValueOf(instance).Interface()

// 针对结构体内属性的具体类型需要利用迭代得到
for i := 0; i < reflect.TypeOf(instance).NumField(); i++ {
    log.Print(reflect.TypeOf(instance).Field(i).Name)
    log.Print(reflect.TypeOf(instance).Field(i).Type) //或reflect.ValueOf(instance).Field(i).Type()
    log.Print(reflect.ValueOf(instance).Field(i))
}
```

# 学习资料
- [《Go语言编程》第3章 面向对象编程](https://book.douban.com/subject/11577300/)
- ['Golang tutorials series': Object Oriented Programming](https://golangbot.com/learn-golang-series/)
- [《Head First 设计模式（中文版）》](https://bookset.me/5123.html)

# Changelog
- 2019/06/10：更新UML类图关系及前言描述
- 2019/06/13：add [Golang tutorials series: Object Oriented Programming](https://golangbot.com/learn-golang-series/)
- 2019/06/18：修改针对接口变量进行类型查询与类型断言部分内容
- 2019/06/24：添加《head first设计模式》第一章学习笔记
