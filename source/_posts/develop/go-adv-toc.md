---
title: Go语言进阶总结：源码学习资料总结
date: 2019-06-01 13:54:23
updated: 2019-07-23 21:59:23
categories:
- Go语言进阶

tags:
- Go
---
# 前言
Go语言进阶学习资料总结，包括：如何学习源码、如何获取最新动态使用最新特性、源码的目录说明、源码编译安装Go的过程、调试工具gdb/[dlv](https://github.com/go-delve/delve/)的安装与使用以及性能分析工具的使用、主要相关学习资料链接。

<!-- more -->
# 学习清单
## 源码相关
### [如何获取Go最新动态和使用最新特性](http://blog.studygolang.com/2014/08/go_news_and_feature/)
1. 自己编译[tip版本源码](http://go.pkgdoc.org/)
2. 使用新特性

- [源码安装GO的过程](http://blog.studygolang.com/2013/01/%E5%88%86%E6%9E%90%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85go%E7%9A%84%E8%BF%87%E7%A8%8B/)
- [Go如何测试标准库](https://mzh.io/go-test-standrad-library-changes-zh-CN)

### [如何学习使用标准库](http://blog.studygolang.com/2012/11/%E5%AD%A6%E4%B9%A0go%E5%8C%85/)
1. 文档Overview了解大致信息
2. Index，关注常量和变量以及**函数**和类型（不需要关注类型的方法）
3. Index，关注类型及其方法
4. 例子练习
5. 第三方包实现

### [源码目录说明](http://blog.studygolang.com/2012/12/%E6%8E%A2%E7%A9%B6go%E4%B8%AD%E5%90%84%E4%B8%AA%E7%9B%AE%E5%BD%95%E7%9A%84%E5%8A%9F%E8%83%BD/)
- `api/`
- `doc/`
- `lib/time/`
- `misc/`
- **`src/`：Go源码**
    - `cmd`：工具源码
- `test/`

## 工具使用
### 调试工具
- [GDB调试GO程序](http://blog.studygolang.com/2012/12/gdb%E8%B0%83%E8%AF%95go%E7%A8%8B%E5%BA%8F/)
- overnote/golang：[09-Go工程管理/04-GDB调试.md](https://github.com/overnote/golang/blob/master/09-Go%E5%B7%A5%E7%A8%8B%E7%AE%A1%E7%90%86/04-GDB%E8%B0%83%E8%AF%95.md)
- [Mac 调试 golang 程序](http://www.do1618.com/archives/771/)
- [理解Golang多重赋值](https://studygolang.com/articles/20485)
- **[go-delve](https://github.com/go-delve/delve/)**：推荐纯Go代码使用
- [Debugging what you deploy in Go 1.12](https://blog.golang.org/debugging-what-you-deploy)：提到了Delve的使用

### 性能分析工具
- [go tool pprof](https://www.kancloud.cn/cattong/go_command_tutorial/261357)
- [go pprof 性能分析](https://wudaijun.com/2018/04/go-pprof/)
- [golang 内存分析/动态追踪](https://lrita.github.io/2017/05/26/golang-memory-pprof/)
- [9.1 Go 大杀器之性能剖析 PProf](https://book.eddycjy.com/golang/tools/go-tool-pprof.html)

# 学习资料
- [《浅谈 Go 语言实现原理》](https://draveness.me/golang/)：推荐学习
- [《Go语言标准库》](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example)
- [《 刻意学习 Golang - 标准库源码分析 》](https://learnku.com/articles/25470)
- 文档镜像开启：`godoc -http :8080`
    - [https://golang.google.cn/pkg/](https://golang.google.cn/pkg/)
    - [https://studygolang.com/pkgdoc](https://studygolang.com/pkgdoc)
    - [https://gowalker.org/go](https://gowalker.org/go)

# Changelog
- 2019/06/30：整理格式
- 2019/07/23：添加性能分析工具，调整部分内容