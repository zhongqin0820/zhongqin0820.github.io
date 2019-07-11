---
title: Go实现经典软件模式：目录索引
date: 2019-03-07 20:17:10
updated: 2019-07-11 17:57:56
categories:
- 软件模式

tags:
- Go
---
# 前言
从零开始的，使用Go语言实现经典软件模式，如：设计模式，并发模式。针对每个模式作简单分析，并在Go源码中找到对应的实现并作出简单分析，最后在样例代码部分，使用UML类图/示例图对其进行问题描述，贴出代码内容包括设计模式关键部分代码以及使用部分测试代码并附上分析，最后附上完整的代码链接提供参考。

<!-- more -->
# 参考资料
## Tools related
- ~~[分析go项目源码,采用plantuml自动生成uml类图](https://studygolang.com/articles/9719)~~
- ~~[plantuml.com/class-diagram](http://plantuml.com/class-diagram)~~
- [UML入门系列文章](https://www.cnblogs.com/wolf-sun/p/UML-collaboration-diagram.html)

## Tutorials
- [《Go design pattern》配套视频教程](https://www.bilibili.com/video/av10623920)
- [《Go design pattern》配套代码](https://github.com/PacktPublishing/Go-Design-Patterns)
- [《GO DESIGN PATTERNS》配套书籍](http://dinus.ac.id/private_lib/fahri/GO_DESIGN_PATTERNS.pdf)

## Blogs
- 设计模式
    - [Design Patterns by Svetlin Ralchev](http://blog.ralch.com/categories/design-patterns/)：10篇相关博文讲解
    - CSDN：[用golang实现设计模式](https://blog.csdn.net/qibin0506/column/info/godp)：8篇
    - 维基主页：[Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns)
- 并发模式
    - [go/wiki/LearnConcurrency](https://github.com/golang/go/wiki/LearnConcurrency)
    - [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)

## Publication
- [《Go语言编程》第3章 面向对象编程](https://book.douban.com/subject/11577300/)
- [《Head First 设计模式（中文版）》](https://bookset.me/5123.html)
- [《图说设计模式》](https://design-patterns.readthedocs.io/zh_CN/latest/)
- [《Evaluating the GO Programming Language with Design Patterns》](https://ecs.victoria.ac.nz/foswiki/pub/Main/TechnicalReportSeries/ECSTR11-01.pdf)

## Related Repos
- [tmrts/go-patterns](https://github.com/tmrts/go-patterns)：除了经典设计模式，还包括其它应用模式：Synchronization Patterns，Concurrency Patterns，Messaging Patterns，Stability Patterns，Profiling Patterns，Idioms，Anti-Patterns...
- [senghoo/golang-design-pattern](https://github.com/senghoo/golang-design-pattern)：实现了《研磨设计模式》中的23种设计模式
- [ismayilmalik/golang-design-patterns](https://github.com/ismayilmalik/golang-design-patterns)：Implementation of design patterns in Golang
- [HCLAC/DesignPattern](https://github.com/HCLAC/DesignPattern): 实现了《大话设计模式》中的23种设计模式
- [silsuer/golang-design-patterns](https://github.com/silsuer/golang-design-patterns)：带博文描述
- [zhenbianshu/DesignPattern](https://github.com/zhenbianshu/DesignPattern)：较为完整实现了23种设计模式

# 设计模式
## 设计基础
- [Go语言入门总结：面向对象与Go编程模型](https://cvblogs.cn/2019/05/27/develop/go_object_oriented_intro/)
- 抽象
- 封装
- 继承/实现
- 多态

## 设计原则
- 依赖倒置原则
- 开放封闭原则：对扩展开放，对更改封闭
- 单一职责原则：一个类应该仅有一个引起它变化的原因
- Liskov替换原则
- 接口隔离原则：接口应该小而完备
- 优先使用对象组合，而不是类继承
- 封装变化点
- 针对接口编程，而不是针对实现编程

## 模式内容
- 常见的创建型设计模型：单例模式、工厂模式（简单工厂、工厂方法、抽象工厂）、建造者模式以及原型模式。
- 常见的结构型设计模式：适配器、门面（外观）、代理等。
- 常见的行为型设计模式：迭代器、观察者、状态、策略等。

### 创建型
> - 创建型模式(Creational Pattern)对类的实例化过程进行了抽象，能够将软件模块中对象的创建和对象的使用分离。为了使软件的结构更加清晰，外界对于这些对象只需要知道它们共同的接口，而不清楚其具体的实现细节，使整个系统的设计更加符合单一职责原则。
> - 创建型模式在创建什么(What)，由谁创建(Who)，何时创建(When)等方面都为软件设计者提供了尽可能大的灵活性。创建型模式隐藏了类的实例的创建细节，通过隐藏对象如何被创建和组合在一起达到使整个系统独立的目的。
> **me115/design_patterns** --- <cite>[《图说设计模式》](https://design-patterns.readthedocs.io/zh_CN/latest/)</cite>

### 结构型
> - 结构型模式(Structural Pattern)描述如何将类或者对 象结合在一起形成更大的结构，就像搭积木，可以通过 简单积木的组合形成复杂的、功能更为强大的结构。
> - 结构型模式可以分为类结构型模式和对象结构型模式：
>    - **类结构型模式**关心类的组合，由多个类可以组合成一个更大的系统，在类结构型模式中一般只存在继承关系和实现关系。
>    - **对象结构型模式**关心类与对象的组合，通过关联关系使得在一个类中定义另一个类的实例对象，然后通过该对象调用其方法。
> 根据“合成复用原则”，在系统中尽量使用关联关系来替代继承关系，因此大部分结构型模式都是对象结构型模式。
> **me115/design_patterns** --- <cite>[《图说设计模式》](https://design-patterns.readthedocs.io/zh_CN/latest/)</cite>

### 行为型
> - 行为型模式(Behavioral Pattern)是对在不同的对象之间划分责任和算法的抽象化。
> - 行为型模式不仅仅关注类和对象的结构，而且重点关注它们之间的相互作用。
> - 通过行为型模式，可以更加清晰地划分类与对象的职责，并研究系统在运行时实例对象 之间的交互。在系统运行时，对象并不是孤立的，它们可以通过相互通信与协作完成某些复杂功能，一个对象在运行时也将影响到其他对象的运行。
> - 行为型模式分为类行为型模式和对象行为型模式两种：
>   - 类行为型模式：类的行为型模式使用继承关系在几个类之间分配行为，类行为型模式主要通过多态等方式来分配父类与子类的职责。
>   - 对象行为型模式：对象的行为型模式则使用对象的聚合关联关系来分配行为，对象行为型模式主要是通过对象关联等方式来分配两个或多个类的职责。根据“合成复用原则”，系统中要尽量使用关联关系来取代继承关系，因此大部分行为型设计模式都属于对象行为型设计模式。
> **me115/design_patterns** --- <cite>[《图说设计模式》](https://design-patterns.readthedocs.io/zh_CN/latest/)</cite>

# 并发模式
> With Creational patterns we were dealing with creating objects.
> With the Structural patterns we were learning how to build idiomatic structures
> and in Behavioral patterns we were managing mostly with algorithms.
> Now, **with Concurrency patterns, we will mostly manage the timing execution and order execution of applications that has more than one flow.**
> **[《GO DESIGN PATTERNS》](http://dinus.ac.id/private_lib/fahri/GO_DESIGN_PATTERNS.pdf)** --- <cite>Chapter 9</cite>

## 模式内容
- 同步屏障模式（Barrier）：a very common pattern, especially when we have to wait for more than one response from different Goroutines before letting the program continue
- 异步Future模式（Futures & Promises）：allows us to write an algorithm that will be executed eventually in time (or not) by the same Goroutine or a different one
- 流水线模式（Pipeline）：a powerful pattern to build complex synchronous flows of Goroutines that are connected with each other according to some logic
- 线程池模式（Workers Pool）：a useful tool to control the number of Goroutines in an execution.
- 发布/订阅模式（Publish/Subscriber）：a rewrite of the Observer pattern, written with a concurrent structure.

# Changelog
- 2019/06/09：Add awesome materials
- 2019/06/29：Add a related repo: [HCLAC/DesignPattern](https://github.com/HCLAC/DesignPattern)
- 2019/07/10：Add design basics & principles
- 2019/07/11：Refactor & Add concurrency patterns
