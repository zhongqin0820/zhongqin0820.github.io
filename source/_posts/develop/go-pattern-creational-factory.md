---
title: Go实现创建型模式：工厂模式
date: 2019-06-09 10:51:03
updated: 2019-06-14 12:35:48
categories:
- 软件模式

tags:
- 设计模式
- Go
---
# 前言
按照耦合的粒度分为：简单工厂模式/静态工厂模式，工厂方法模式，抽象工厂模式。使用UML对示例代码进行描述，同时提供对应模式的简单分析，完整示例代码：[creational/factory](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/creational/factory)。

<!-- more -->
# 模式概览
## 关键点
- 相较单例模式是为了提供全局唯一访问点，工厂模式是为了针对**接口**编程，不针对实现编程，做到“对扩展开放,对修改关闭”而提出（即：代码的可扩展性）。
- 通过利用接口减少应用程序和具体类之间的依赖促进松耦合。

## 侧重问题
- 工厂类型的模式侧重的是如何降低耦合性，从而达到“对扩展开放,对修改关闭”。

# 分类
## 简单工厂方法
- 在Go语言编程中是使用最多的一种创建实例的方法：形如`func NewT()*T`。
- “找出会变化的方面,把它们从不变的部分分离出来”。
- 简单工厂通过一个具体工厂使用依赖一个接口的方法，借助类型判断在运行时绑定特定的具体类型，从而处理创建不同类型实例的细节。

### UML描述
<div style="width: 300px; margin: auto">

![UML](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/creational_factory_simple.jpg)
</div>

### 样例代码
在Go可以通过重定义`int`进行语义化选项，通过利用`iota`定义类似枚举的语义化自定义类型常量。
```go
type StorageType int

// 定义选项，NotExistedStorage是为了测试用例的书写引入
const (
    DiskStorage StorageType = 1 << iota
    TempStorage
    MemoryStorage
    NotExistedStorage
)
```

- 简单工厂方法及接口定义

```go
// NewStore 即简单工厂方法，传入参数是定义的语义化选项，每个选项对应的返回类型是Store接口具体实例类型
func NewStore(t StorageType) Store {
    switch t {
    case DiskStorage:
        return newDiskStorage()
    case TempStorage:
        return newTempStorage()
    case MemoryStorage:
        return newMemoryStorage()
    default:
        return nil
    }
}

// Store 是一个接口
type Store interface {
    Save(s string)
}
```

- 使用/测试用例

```go
type testCase struct {
    name    string
    storage StorageType
}

func TestStorage(t *testing.T) {
    var ts []testCase = []testCase{
        {"DiskStorage", DiskStorage},
        {"TempStorage", TempStorage},
        {"MemoryStorage", MemoryStorage},
        {"NotExistedStorage", NotExistedStorage},
    }
    // table test
    for _, v := range ts {
        t.Run(v.name, func(t *testing.T) {
            storage := NewStore(v.storage)
            if storage == nil {
                t.Errorf("Type DiskStorage not implemented yet")
            } else {
                storage.Save(fmt.Sprint("Save from ", v.name))
            }
        })
    }
}
```

### 总结及分析
- [《图说设计模式》：简单工厂模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/simple_factory.html)篇总结的很到位。

## 工厂方法
- 抽象工厂类与产品类，特定工厂具体类生产特定具体类型产品
- 即定义了一个创建对象的接口，但由子类决定要实例化的产品类是哪一个。将类把实例化推迟到子类。

### UML描述
<div style="width: 300px; margin: auto">

![UML](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/creational_factory_method.jpg)
</div>

### 样例代码
- 对应UML图实现即可，注意的是，在具体工厂类中返回的是具体的产品类。

```go
func (r *RestfulFactory) CreateServer() Server {
    return &RestfulServer{}
}
```

- 测试用例：注意，这里的调用面向接口。

```go
func TestMethod(t *testing.T) {
    var factory Factory
    var server Server
    // call an implementation of Factory interface
    factory = &SoaFactory{}
    // get correspongding product
    server = factory.CreateServer()
    name := server.GetServerType()
    if name != "SOA" {
        t.Error("Wrong type!")
    }
}
```

### 总结及分析
- 参见[《图说设计模式》：工厂方法模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/factory_method.html)。

## 抽象工厂
- 依赖倒置原则（更强的抽象）：要依赖抽象，不要依赖具体类。
- 使用对象组合，对象的创建被实现在工厂接口所暴露出来的方法中。
- 提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类（调用者只依赖接口）。

### UML描述
<div style="width: 300px; margin: auto">

![UML](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/creational_factory_abstract.jpg)
</div>

### 样例代码
- 根据UML描述实现即可，注意抽象工厂方法是对一族对象进行创建。

### 总结及分析
- 参见[《图说设计模式》：抽象工厂模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/abstract_factory.html)

# Go源码涉及该模式的内容
> Well in order to connect to PostgreSQL server the sql package instaciate the registered driver via Factory Design Pattern. The driver is registered by this function.
> The Factory function is db.Open.
> **Svetlin Ralchev** --- <cite>[Design Patterns in Golang: The Good, the Bad and the Ugly](http://blog.ralch.com/tutorial/design-patterns/golang-design-patterns/)</cite>

# 学习资料
## 出版物
- [《Head First 设计模式（中文版）》：chap4 工厂模式](https://bookset.me/5123.html)
- [《图说设计模式》：创建型模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/creational.html)

## 博客
- [Design Patterns in Golang: The Good, the Bad and the Ugly](http://blog.ralch.com/tutorial/design-patterns/golang-design-patterns/)
- [Factory patterns in Go (Golang)](https://www.sohamkamani.com/blog/golang/2018-06-20-golang-factory-patterns/)
- [Desing Patterns in Golang: Factory Method](http://blog.ralch.com/articles/design-patterns/golang-factory-method/)

# Changelog
- 2019/06/14：添加三个模式的UML描述以及分析资料链接等。
