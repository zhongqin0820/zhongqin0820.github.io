---
title: Go语言入门总结：接口与反射
date: 2019-06-18 10:15:50
updated: 2019-06-27 11:22:39
categories:
- Go语言入门

tags:
- Go
---
# 前言
梳理总结学习Go语言的笔记。关于结构体的基础知识，以及关于接口与反射的进一步知识的梳理。

<!-- more -->
# 结构体
## 定义
结构体是一组属性对`name type`的集合，使用`type XXX struct`进行定义。
```go
// 一般结构体
type person struct {
    firstName string
    lastName  string
    age       int
}
// 成员属性是切片
type animal struct {
    name string
    characteristics []string
}
// herbivore匿名嵌套结构体animal
type herbivore struct {
    animal//匿名结构体，相当于OOP中的继承
    eatHuman bool
}
// 匿名结构体的定义与初始化是一起的，需要将其区分于匿名嵌套；
// 前者是定义了一个匿名结构体，后者是嵌套了一个结构体不对其命名而使用其结构体类型名
bio := struct {
    firstName string
    friends   map[string]int
    favDrinks []string
}{
    firstName: "Steven",
    friends: map[string]int{
        "Tim":   12345678,
        "Abdul": 34789657,
        "Kally": 28993332,
    },
    favDrinks: []string{
        "Pepsi",
        "Cocacola",
    },
}
```

## 声明与初始化结构体变量
对结构体变量的声明与初始化符合Go类型系统的方式；需要注意的是：字面量的多行书写最后一行的逗号不可省
```go
// 一般结构体
p1 := person{
    firstName: "Mark",
    lastName:  "Kedu",
    age:       30,//此处的逗号不可以省略,否则：syntax error: unexpected newline, expecting comma or }
}
// 成员属性是切片时的初始化方式
animal1 := animal{
    name: "Lion",
    characteristics: []string{"Eats human",
        "Wild Animal",
        "King of the jungle",//此处的逗号不可以省略
    },//此处的逗号不可以省略
}
// 带有匿名嵌套结构体的初始化方式
herb := herbivore{
    animal: animal{
        name: "Goat",
        characteristics: []string{"Lacks sense",
            "Lazy",
            "Eat grass",
        },
    },
    eatHuman: false,
}
```

### 类型提升
发生在匿名嵌套结构体时，为了避免被嵌套的结构体与其存在`name type`冲突。需要通过指定对应的具体所属结构体进行区分。
当不存在冲突时，可以直接利用结构体的变量实例访问匿名嵌套结构体中的属性，如本例中的`herb.characteristics`等同于`herb.animal.characteristics`

## 方法定义
方法是特殊的带有接收者的函数，接收者可以是普通类型也可以是对应的指针类型。
- 当我们需要对接收者的值进行修改时，应该使用对应的指针类型。

# 接口
接口是一组方法签名的集合。结构体通过实现接口定义的方法，配合接口可以实现OOP中的运行时多态。通过定义一个空接口（相当于any type）可以一定程度实现范型编程，不过接口的实际值只有在运行时才知道。

## 类型系统
Go的类型包括编译期的静态类型与运行时的具体类型。使用`type value`对表示。
- 一个接口变量值由两个指针组成：
    - 第一个指向一个interface table，叫 itable。itable开头是一些描述结构体类型的元字段，后面是一串方法。注意这个方法是接口本身的方法，并非其运行时绑定的结构体对象的方法。但其实现由具体的结构体实现。当这个interface无方法时，itable可以省略，直接指向一个type即可。
    - 另一个指针data指向运行时具体结构体类型变量的一个拷贝。也就是，给结构体变量赋值时，**会在堆上分配内存，用于存放拷贝的值。**同样，当值本身只有一个字长时，这个指针也可以省略。
- 一个接口变量的初始值是两个nil。接口变量是否为nil取决于itable字段。所以不一定data为nil就是nil，判断时要额外注意。

> 注意：计算接口变量对象的itable时，需要对两种类型的方法列表进行遍历对比。itable的计算不需要到函数调用时进行，只需要在接口变量赋值时进行即可。

## 源码分析
- [golang的interface剖析](https://www.cnblogs.com/qqmomery/p/6298771.html)：总结的十分到位。

```go
type iface struct {
    tab  *itab              // 指向itable
    data unsafe.Pointer     // 指向真实数据
}

type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab            //interface类型与具体类型之间的信息结合起来做一个hash的过程，itab中的link就是hash中的拉链。
    bad    int32
    unused int32
    fun    [1]uintptr       // variable sized
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
- `reflect.Value`：可以通过`reflect.ValueOf(接口变量)`得到；
    - `reflect.Value.Type()`可以返回对应接口变量实例的类型`reflect.Type`

## 反射对象到接口变量值
- 可以通过`reflect.ValueOf(接口变量).Interface()`得到

## 对内容进行修改
- 想要对接口变量值进行修改，需要传入**接口变量的地址**
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
func (v Value) Type() Type
```

# 参考资料
- [golang的interface剖析](https://www.cnblogs.com/qqmomery/p/6298771.html)
- [Part 34: Reflection](https://golangbot.com/reflection/)
- [Golang的反射reflect深入理解和示例](https://juejin.im/post/5a75a4fb5188257a82110544)
- [【翻译】反射的规则](https://studygolang.com/articles/1010)
- [【原文】The Laws of Reflection](https://blog.golang.org/laws-of-reflection)

# 扩展阅读
- 《浅谈 Go 语言实现原理》：[2.2 接口](https://draveness.me/golang/basic/golang-interface.html)

# Changelog
- 2019/06/27：添加结构体部分的内容
- 2019/07/04：添加`reflect.Value.Type()`
- 2019/07/06：添加《浅谈 Go 语言实现原理》扩展阅读
