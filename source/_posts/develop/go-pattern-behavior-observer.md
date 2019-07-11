---
title: Go实现行为型模式：观察者模式
date: 2019-07-03 16:33:27
updated: 2019-07-03 16:33:27
categories:
- 设计模式

tags:
- Go
---
# 前言
观察者模式是一种对象行为型模式。当一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。

<!-- more -->
# 场景分析
观察者模式也叫发布/订阅模式（Publish/Subscribe）或模型/视图模式（Model/View）或源/监听器模式（Source/Listener）或从属者模式（Dependents），它通过建立一种对象与多个对象之间的依赖关系（`1:n`），实现当**一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应**。

发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，这就是观察者模式的模式动机。

## 关键点
- Theme: 主题内容（观察目标）
- Publisher: 发布者
- Subscriber: 订阅者（观察者）
- ConcretePublisher: 具体发布者
- ConcreteSubscriber: 具体订阅者

# UML描述

<div style="width: 300px; margin: auto">
![UML](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/pattern/behavior_observer.png)
</div>

# 代码实现
## 定义部分
- [接口定义部分完整代码](https://github.com/zhongqin0820/coding-playground/blob/master/go/pattern/behaviroal/observer/observer.go)

```go
type Theme string

type (
    Publisher interface {
        Register(Subscriber)
        Deregister(Subscriber)
        Notify(Theme)
    }

    Subscriber interface {
        OnNotify(Theme)
    }
)
```

- [具体实现部分完整代码](https://github.com/zhongqin0820/coding-playground/blob/master/go/pattern/behaviroal/observer/client.go)

```go
// 使用map来存储订阅者方便添加与删除
type pub struct {
    subList map[Subscriber]struct{}
}

// Notify 实现当主题Theme修改时，通知所有订阅了该发布者内容的所有订阅者
func (p *pub) Notify(t Theme) {
    for s, _ := range p.subList {
        s.OnNotify(t)
    }
}

// OnNotify订阅者对自己订阅的主题内容进行修改
func (s *sub) OnNotify(t Theme) {
    s.theme = t
}
```

## 测试代码
实现包括三个单元测试案例：注册订阅者，取消注册订阅者，主题更新通知所有订阅者。
- [具体实现部分完整代码](https://github.com/zhongqin0820/coding-playground/blob/master/go/pattern/behaviroal/observer/observer_test.go)

```go
// 测试通知机制
t.Run("Notify", func(t *testing.T) {
    if len(p.subList) == 0 {
        t.Errorf("publist is empty")
    }
    for s, _ := range p.subList {
        c, ok := s.(*sub)
        if !ok {
            t.Fail()
        }
        if c.theme != "default" {
            t.Errorf("expected default, get %s\n", c.theme)
        }
    }
    // update theme
    var newTheme Theme = "hello"
    p.Notify(newTheme)
    for s, _ := range p.subList {
        c, ok := s.(*sub)
        if !ok {
            t.Fail()
        }
        if c.theme != newTheme {
            t.Errorf("expected %s, get %s\n", newTheme, c.theme)
        }
    }
})
```

# 学习资料
- 出版物/视频教程
    - 《图说设计模式》之行为型模式：[观察者模式](https://design-patterns.readthedocs.io/zh_CN/latest/behavioral_patterns/observer.html)，推荐阅读优缺点分析及总结部分内容
    - GeekBand：23种设计模式之[观察者模式](https://www.bilibili.com/video/av24176315/?p=5)
    - silsuer/golang-design-patterns：[观察者模式](https://github.com/silsuer/golang-design-patterns/tree/master/observer-pattern)
- 其它实现参考
    - PacktPublishing/Go-Design-Patterns：[observer](https://github.com/PacktPublishing/Go-Design-Patterns/tree/master/Chapter07/observer)
    - tmrts/go-patterns：[behavioral/observer](https://github.com/tmrts/go-patterns/blob/master/behavioral/observer.md)
    - HCLAC/DesignPattern：[Observer](https://github.com/HCLAC/DesignPattern/tree/master/Observer)
    - senghoo/golang-design-pattern：[10_observer](https://github.com/senghoo/golang-design-pattern/tree/master/10_observer)
    - zhenbianshu/DesignPattern：[src/observer](https://github.com/zhenbianshu/DesignPattern/tree/master/src/observer)

# Changelog
- 2019/07/03：第一版内容