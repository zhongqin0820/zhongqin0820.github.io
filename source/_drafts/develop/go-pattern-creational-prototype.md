---
title: Go实现创建型模式：原型模式
date: 2019-06-26 14:18:24
updated: 2019-06-26 22:28:01
categories:
- 软件模式

tags:
- 设计模式
- Go
---
# 前言
原型模式配合原型管理器使用，使得客户端在不知道具体类的情况下，通过接口管理器得到新的实例，并且包含部分预设定配置。其使对象能复制自身，并且暴露到接口中，使客户端面向接口编程时，不知道接口实际对象的情况下生成新的对象。

<!-- more -->
# 场景分析
- 原型模式是为了避免重复的大量/复杂对象的创建，只需：
    - 维护一组可能被用户拷贝复用的对象
    - 为这些对象提供一个默认的初始值
    - 避免在运行时进行复杂的对象初始化工作

- 原型模式用于在运行时拷贝那些在编译期时创建的一个或一族对象。例如：
    - 针对系统用户，提供一个默认的用户初始模版/默认的服务套餐，之后用户可以针对自己的需求进行任意的修改

- 原型模式需要区别与创建者模式在运行时创建对象，原型模式是在运行时拷贝在编译期创建的对象。

# 关键点
通常该模式需要和其它模式组合使用才能发挥更大的价值，这里给出的示例简单起见，只有一个具体类型，还可以对其进行扩展，然后针对多个类型扩展用一个工厂方法对其进行管理。
- 一个抽象的可克隆接口，即：其利用`Clone() Role`返回接口自身实例
- 具体的类型根据特定需求对抽象接口进行实现
- 在客户端使用时，创建一个接口变量，将具体类型变量赋值给接口变量，创建新的接口变量时直接利用`Clone() Role`返回接口变量实例，并在其上进行修改

# UML描述

<div style="width: 300px; margin: auto">

![UML](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/develop/pattern/creational_prototype.png)
</div>

# 样例代码
## 实现部分

```go
// 抽象模板
type Role interface {
    Clone() Role
    SetHeadColor(string)
    SetEyesColor(string)
    SetTall(int)
    Show()
}

// 针对具体类型的工厂方法
func NewRole() *Role {
    return &Chinese{HeadColor: "black", EyesColor: "yellow", Tall: 190}
}

// 具体类型
type Chinese struct {
    HeadColor string
    EyesColor string
    Tall      int
}

func (c *Chinese) Clone() Role {
    return &Chinese{HeadColor: c.HeadColor, EyesColor: c.EyesColor, Tall: c.Tall}
}

func (c *Chinese) SetHeadColor(s string) {
    c.HeadColor = s
}

func (c *Chinese) SetEyesColor(s string) {
    c.EyesColor = s
}

func (c *Chinese) SetTall(s int) {
    c.Tall = s
}

func (c *Chinese) Show() {
    log.Printf("%s,%s,%d\n", c.HeadColor, c.EyesColor, c.Tall)
}

```

## 客户端/测试
- 针对该模式，对两个变量进行比较时可利用`reflect.DeepEqual(a1, a2 interface{}) bool`进行[深度相等判断](https://studygolang.com/static/pkgdoc/pkg/reflect.htm#DeepEqual)。

```go
func TestPrototype(t *testing.T) {
    var role Role
    role = NewRole()
    role.Show()
    // 新建实例基于模板拷贝
    var another Role = role.Clone()
    another.SetTall(160)
    another.Show()
    role.Show()
    t.Run("same type", func(t *testing.T) {
        // 判断
        if reflect.DeepEqual(role, another) {
            t.Logf("same: %p, %p\n", role, another)
        } else {
            t.Logf("different: %p, %p\n", &role, &another)
        }
    })
}
```

# 模式分析
## 模式缺点
> One of the disadvantages of this pattern is that the process of copying an object can be complicated. In addition, structs that have circular references to other classes are difficult to clone. Its overuse could affect performance, as the prototype object itself would need to be instantiated if you use a registry of prototypes.
> In the context of Golang, I don’t see any particular reason to adopt it. Golang already provides builtin mechanism for cloning objects.
> **Svetlin Ralchev** --- <cite>[Design Patterns in Golang: Prototype](http://blog.ralch.com/articles/design-patterns/golang-prototype/)</cite>

# 参考资料
- Video: [Design Patterns in Golang: Prototype](https://www.bilibili.com/video/av10623920/?p=15) & its implementation in [GitHub.com](https://github.com/ismayilmalik/golang-design-patterns/blob/master/creational/prototype/prototype.go) & another implementation in [GitHub.com](https://github.com/senghoo/golang-design-pattern/blob/master/07_prototype/README.md)
- Blog: [Design Patterns in Golang: Prototype](http://blog.ralch.com/articles/design-patterns/golang-prototype/) & its implementation in [GitHub.com](https://github.com/svett/golang-design-patterns/blob/master/creational-patterns/prototype/README.md)
- 类型判断相关
    - [浅谈Golang Reflection (二)](https://zhuanlan.zhihu.com/p/40840214)
    - [13.3. 示例: 深度相等判断](http://shouce.jb51.net/gopl-zh/ch13/ch13-03.html)
    - [reflect.htm#DeepEqual](https://studygolang.com/static/pkgdoc/pkg/reflect.htm#DeepEqual)
