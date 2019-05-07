---
title: Go语言入门总结:基础数据类型
date: 2019-03-19 11:47:10
updated: 2019-03-19 11:47:10
categories:
- 网页开发

tags:
- Go
---
## 前言
梳理总结学习Go语言的笔记，基础数据类型布尔型，数值型，字符型，`error`及`errors`包

<!-- more -->
## 基础数据类型
基础数据类型是构建更复杂数据结构的基础。go语言的基本数据类型共18个（不包括`error`类型），有必要对其熟练掌握。

### 布尔型`bool`：`1 byte`
#### 取值
- `true`
- `false`（零值）

### 数值型
#### 整型
- `int`/`uint`：`- byte`（宽度与平台相关） | `0`
- `int8`/`uint8`：`1 byte` | `0`
- `int16`/`uint16`：`2 byte` | `0`
- `int32`/`uint32`：`4 byte` | `0`
- `int64`/`uint64`：`8 byte` | `0`

#### 浮点型
- `float32`：`4 byte` | `0.0`
- `float64`：`8 byte` | `0.0`

#### 复数
- `complex64`：`8 byte` | `0.0 + 0.0i`
- `complex128`：`16 byte` | `0.0 + 0.0i`

### 字符型
#### 字符
- `rune`（`int32`）：`4 byte` | `0` (专门用于存储Unicode字符)
- `byte`：`1 byte` | `0`

#### 字符串
- `string`：`-`（可变） | `""`

### 错误类型
`error`类型可以理解为是`string`的别名。底层定义：
```go
type error string
```

#### 产生`error`
##### 使用`errors.New()`方法
- 需要引入`errors`包。

```go
var err = errors.New("this is an Error!")
```

###### `errors`包源码
```
// Package errors implements functions to manipulate errors.
package errors

// New returns an error that formats as the given text.
func New(text string) error {
    return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}
```

##### 使用`fmt.Errorf()`方法
- 需要引入`fmt`包
```
var err = fmt.Errorf("%s\n", "this is an another Error!") // 返回的是*errors.errorString 类型
err.Error() // 返回的是string类型
```

##### 自定义实现
- 需要自定义结构体，并实现接口`func Error() string`

```
type MyError struct {
    err string
}

// Error 返回错误说明
func (m MyError) Error() string {
    errinfo := fmt.Sprintf("this is an Error no.3: %s ", m.err)
    return errinfo
}

// 调用
m := MyError{err: "My Error!"}
fmt.Println(m.Error())

fmt.Println(reflect.TypeOf(m)) // main.MyError
fmt.Println(reflect.TypeOf(m.Error())) // string
```

#### 合理处理`error`
- 搭配`panic()`以及`recover()`
- 利用`defer()`确保程序能够在发生运行时恐慌`panic()`时，对其进行处理`recover()`

## 参考资料
- [《Go并发编程》](https://book.douban.com/subject/27016236/)
- [https://golang.org/pkg/errors/](https://golang.org/pkg/errors/)