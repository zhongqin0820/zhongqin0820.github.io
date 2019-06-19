---
title: Go语言入门总结:基础数据类型
date: 2019-03-19 11:47:10
updated: 2019-06-18 15:46:12
categories:
- 网页开发

tags:
- Go
---
# 前言
梳理总结学习Go语言的笔记中关于类型系统部分内容，包括：基础数据类型布尔型，数值型，字符型，`error`及`errors`包及复合数据类型：数组，结构体以及指针。

<!-- more -->
# 基础数据类型
基础数据类型是构建更复杂数据结构的基础。go语言的基本数据类型共18个（不包括`error`类型），有必要对其熟练掌握。

## 布尔型`bool`：`1 byte`
### 取值
- `true`
- `false`（零值）

## 数值型
`1 byte` = `8 bits`

### 整型
- `int`/`uint`：`- byte`（宽度与平台相关） | `0`
- `int8`/`uint8`：`1 byte` | `0`
- `int16`/`uint16`：`2 byte` | `0`
- `int32`/`uint32`：`4 byte` | `0`
- `int64`/`uint64`：`8 byte` | `0`

### 浮点型
- `float32`：`4 byte` | `0.0`
- `float64`：`8 byte` | `0.0`

### 复数
- `complex64`：`8 byte` | `0.0 + 0.0i`
- `complex128`：`16 byte` | `0.0 + 0.0i`

## 字符型
### 字符
- `rune`（`int32`）：`4 byte` | `0` (专门用于存储Unicode字符)
- `byte` (`uint8`) ：`1 byte` | `0`

> 注意，使用`for range`对字符串迭代得到的元素是`rune`类型，而使用`for i:=0;i<len(s); i++ {}`遍历得到的元素是`byte`类型。

### 字符串
- `string`：`-`（可变） | `""`
- 或使用反引号括起来定义，即为raw格式，没有字符串转义等，原样输出。

> 注意，字符串属于读类型数据，不可对其修改，但是由于其底层由`[...]byte{}`数组实现，因此可以对其进行`+`操作拼接字符串，从而产生一个新的字符串。
> 或者先转换为`c:=[]byte(s)`在转换为`s2=string(c)`

### 字符串处理相关源码包
#### strings
- 提供了很多操作字符串的简单函数，通常一般的字符串操作需求都可以在这个包中找到。

#### bytes
- 定义了一些操作 byte slice 的便利操作。因为字符串可以表示为 []byte，因此，bytes 包定义的函数、方法等和 strings 包很类似。

#### strconv
- 提供了基本数据类型和字符串之间的转换。在 Go 中，没有隐式类型转换，一般的类型转换可以这么做：int32(i)，将 i （比如为 int 类型）转换为 int32，然而，字符串类型和 int、float、bool 等类型之间的转换却没有这么简单。

#### unicode
- Go 代码使用 UTF-8 编码（且不能带 BOM），同时标识符支持 Unicode 字符。在标准库 unicode 包及其子包 utf8、utf16中，提供了对 Unicode 相关编码、解码的支持，同时提供了测试 Unicode 码点（Unicode code points）属性的功能。
- 常用，其中ch是rune类型的字符
    - 判断是否为字母： `unicode.IsLetter(ch)`
    - 判断是否为数字： `unicode.IsDigit(ch)`
    - 判断是否为空白符号： `unicode.IsSpace(ch)`

#### regexp
- 提供了正则表达式功能，它的语法基于 RE2 ，regexp/syntax 子包进行正则表达式解析。

## 错误类型
`error`类型可以理解为是`string`的类型定义。底层定义：
```go
type error string
```

### 产生`error`
#### 使用`errors.New()`方法
- 需要引入`errors`包。

```go
var err = errors.New("this is an Error!")
```

##### `errors`包源码
```
// Package errors implements functions to manipulate errors.
package errors


// errorString is a trivial implementation of error.
type errorString struct {
    s string
}


// New returns an error that formats as the given text.
func New(text string) error {
    return &errorString{text}
}


func (e *errorString) Error() string {
    return e.s
}
```

#### 使用`fmt.Errorf()`方法
- 需要引入`fmt`包
```
var err = fmt.Errorf("%s\n", "this is an another Error!") // 返回的是*errors.errorString 类型
err.Error() // 返回的是string类型
```

#### 自定义实现
- 需要自定义结构体，并实现接口`func Error() string`。

```
type MyError struct {
    err string
}

// Error 返回错误说明
func (m *MyError) Error() string {
    errinfo := fmt.Sprintf("this is an Error no.3: %s ", m.err)
    return errinfo
}

// 调用
m := &MyError{err: "My Error!"}
fmt.Println(m.Error())

fmt.Println(reflect.TypeOf(m)) // *main.MyError
fmt.Println(reflect.TypeOf(m.Error())) // string
```

### 合理处理`error`
- 搭配`panic()`以及`recover()`
- 利用`defer()`确保程序能够在发生运行时恐慌`panic()`时，对其进行处理`recover()`，`defer()`也常常用于一些资源的释放。

```go
func(){
    // defer
    defer func(){
        if p:=recover();p!=nil {
            fmt.Println("recover this")
        }
    }()
    panic("this is a panic")
}()
// output:
// recover this
```

# 复合数据类型
## 数组
- 之前的博文，对数组与结构体定义进行了简单[梳理](https://cvblogs.cn/2018/06/22/develop/go_basic_intro_datastructs/)。

## 结构体
- 主要是理解结构体定义的自定义类型，与类型重定义得到的新类型的差别与联系，以及关于类型别名的分析。

```go
// 定义结构体类型，包含属性和方法的结构
type XXX struct {

}

// 类型重定义，相当于继承YYY类型的所有属性和方法，可以进行强制类型转换
// 二者定义的变量的具体类型不同，但是静态类型相同都是YYY类型
type XXX YYY

// 类型别名，扩展XXX的方法集合会影响YYY类型
// 二者定义的静态类型相同
type XXX=YYY
```

## 指针
- 任何类型都有其对应的指针类型，指针类型的零值是`nil`，所有指针类型变量所占内存大小相同，指针的所占内存大小由机器位数决定，64位机器占8字节。
- Go不支持指针运算
- Go中函数传参基于值拷贝，对于引用类型如`slice`，`map`，`chan`的元数据实际是指针，因此可以将其理解为相当于传引用。
- 指针使用的时机：
    - 需要共享数据，避免大数据的拷贝带来的内存资源的占用
    - 需要对数据进行修改，
- `*`与`&`符号
    - 指针变量声明/定义时的`*`指示这是一个指针类型的变量，对指针变量进行操作时的`*`是对指针变量的解引用。
    - 对指针变量的赋值需要与指针的解引用进行区别。前者操作的指针变量自身，后者是对指针变量指向的内容进行操作。
    - `&`作为地址符号时，相当于说明共享该地址指向的内存区域。

# 类型查询与反射
- 类型查询使用`type`关键字对接口的类型进行查询，通常需要预知该接口可能被实现的具体类型。通过`switch case`进行判断执行对应操作。
- 当需要查询的接口对应的具体变量值不可知时，通常使用reflect 包提供的反射机制无需预知接口可能被实现的具体类型。

## 更多对比
|内容|type关键字|reflect包|
|:---:|:---:|:---:|
|类型查询|`instance.(type)`必须配合`switch case`进行判断执行对应操作。|`reflect.TypeOf(instance)`返回Type结构体类型|
|类型断言|`instance.(instanceType)`返回具体值|`reflect.ValueOf(instance)`返回Value结构体类型|

# 参考资料
- [《Go并发编程》](https://book.douban.com/subject/27016236/)
- [https://golang.org/pkg/errors/](https://golang.org/pkg/errors/)