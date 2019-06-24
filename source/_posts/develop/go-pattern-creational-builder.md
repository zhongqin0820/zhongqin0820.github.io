---
title: Go实现创建型模式：建造者模式
date: 2019-06-14 15:19:58
updated: 2019-06-24 12:42:23
categories:
- 计算机基础

tags:
- Go
- 设计模式
---
# 前言
建造者/生成器模式，封装一个类型的构造过程，并允许按多个步骤构造。主要将其区别于工厂模式的一步构造出一族实例。本文中样例代码来自[tmrts/go-patterns](https://github.com/tmrts/go-patterns/blob/master/creational/builder.md)的补充修改完善。同时提供对样例代码的UML描述。

<!-- more -->
# 场景分析
建造者模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。**用户只需要指定复杂对象的类型就可以得到该对象，而无须知道其内部的具体构造细节。**（例子中是将构造的过程推迟到了使用部分）。
这类具体的建造者对象通常聚合/组合了一系列成员属性，这些成员属性中有些是引用类型的成员对象。而且在这些复杂对象中，还可能存在一些限制条件，如某些属性没有赋值则复杂对象不能作为一个完整的产品使用；有些属性的赋值必须按照某个顺序，一个属性没有赋值之前，另一个属性可能无法赋值等。例如：复杂对象相当于一辆有待建造的汽车，而对象的属性相当于汽车的部件，建造产品的过程就相当于组合部件的过程。
利用建造者模式，用户只需通过指定复杂对象的类型和内容就可以构建它们，无须关心该对象所包含的属性以及它们的组装方式而得到一个已经建造完毕的完整产品对象。

## 区别抽象工厂模式
- 与抽象工厂模式相比， 建造者模式返回一个组装好的完整产品 ，而抽象工厂模式返回一系列相关的产品，这些产品位于不同的产品等级结构，构成了一个产品族。
- 在抽象工厂模式中，客户端实例化工厂类，然后调用工厂方法获取所需产品对象，而在建造者模式中，客户端可以不直接调用建造者的相关方法，而是通过指挥者类来指导如何生成对象，包括对象的组装过程和建造步骤，它侧重于**一步步构造**一个复杂对象，返回一个完整的对象。
- 如果将抽象工厂模式看成汽车**配件生产**工厂，生产一个产品族的产品，那么建造者模式就是一个**汽车组装**工厂 ，通过对部件的组装可以返回一辆完整的汽车。

# Go源码涉及该模式的内容
- [Design Patterns in Golang: The Good, the Bad and the Ugly](http://blog.ralch.com/tutorial/design-patterns/golang-design-patterns/)
    - [The Go image/draw package](https://blog.golang.org/go-imagedraw-package)
    - [database/sql](https://golang.org/src/database/sql/sql.go?s=805:853#L468)

# UML描述
- 【抽象建造者类】中定义了产品的创建方法和返回方法
- 建造者模式的结构中还引入了一个【指挥者类Director】，该类的作用主要有两个：
    - 一方面它隔离了客户与生产过程；另一方面它负责控制产品的生成过程。
    - 指挥者针对抽象建造者编程，客户端只需要知道具体建造者的类型，即可通过指挥者类调用建造者的相关方法，【返回一个完整的产品对象】
- 在客户端代码中，无须关心产品对象的具体组装过程，只需【确定具体建造者的类型】即可，建造者模式将复杂对象的构建与对象的表现分离开来，这样使得同样的构建过程可以创建出不同的表现。

<div style="width: 300px; margin: auto">
![UML](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/creational_builder.png)
</div>

# 代码实现
本例为[tmrts/go-patterns](https://github.com/tmrts/go-patterns/blob/master/creational/builder.md)中创建者模式的细节实现。直接利用`func NewBuilder(b Builder) Builder`方法代替了上面的`type Manufactor struct`的封装。完整代码请访问：[creational/builder/car*](https://github.com/zhongqin0820/coding-playground/tree/master/go/pattern/creational/builder)部分内容。

## 接口部分
```go
// 定义了建造者接口
type Builder interface {
    // 一个步骤
    Paint(Color) Builder
    Wheels(Wheels) Builder
    TopSpeed(Speed) Builder
    // 组装过程，返回产品接口
    Build() Interface
}

// 定义了产品接口
type Interface interface {
    Drive() error
    Stop() error
}

// 相当于指挥者，返回建造者接口变量
func NewBuilder(b Builder) Builder {
    return b
}
```

## 测试用例/客户端代码

```go
// Car是具体的建造者
type Car struct {
    color  Color
    wheels Wheels
    speed  Speed
}

func (c Car) Paint(color Color) Builder {
    c.color = color
    return c
}
func (c Car) Wheels(w Wheels) Builder {
    c.wheels = w
    return c
}
func (c Car) TopSpeed(s Speed) Builder {
    c.speed = s
    return c
}
func (c Car) Build() Interface {
    switch c.wheels {
    case SportsWheels:
        return SportsCar{}
    case SteelWheels:
        return FamilyCar{}
    }
    return nil
}

// 通过NewBuilder创建建造者实例
assembly := NewBuilder(Car{})

// 使用建造者实例构建具体产品，也可以将其封装在指挥者的代码中而不是在客户端调用
familyCar := assembly.Paint(RedColor).Wheels(SportsWheels).TopSpeed(50 * MPH).Build()

// 具体产品方法调用
familyCar.Drive()
familyCar.Stop()
```

# 参考资料
- [《图说设计模式》：建造者模式](https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/builder.html)
- [tmrts/go-patterns](https://github.com/tmrts/go-patterns/blob/master/creational/builder.md)
- [Design Patterns in Golang: The Good, the Bad and the Ugly](http://blog.ralch.com/tutorial/design-patterns/golang-design-patterns/)
- [GO DESIGNPATTERNS](http://dinus.ac.id/private_lib/fahri/GO_DESIGN_PATTERNS.pdf)
