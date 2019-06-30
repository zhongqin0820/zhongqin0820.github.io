---
title: Go语言进阶总结：源码学习资料总结
date: 2019-06-01 13:54:23
updated: 2019-06-30 22:50:18
categories:
- Go语言进阶

tags:
- Go
---
# 前言
Go语言进阶学习资料总结，包括：如何学习源码、如何获取最新动态使用最新特性、源码的目录说明、调试工具gdb/[dlv](https://github.com/go-delve/delve/)的安装与使用、源码编译安装Go的过程、以及主要的学习资料[《Go语言标准库》](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example)和[官方包API文档国内镜像](https://studygolang.com/pkgdoc)的链接等。

<!-- more -->
# 学习清单
## [如何学习源码](http://blog.studygolang.com/2012/11/%E5%AD%A6%E4%B9%A0go%E5%8C%85/)
1. 文档Overview了解大致信息
2. Index，关注常量和变量以及**函数**和类型（不需要关注类型的方法）
3. Index，关注类型及其方法
4. 例子练习
5. 第三包实现：http://go.pkgdoc.org/

## [如何获取Go最新动态和使用最新特性](http://blog.studygolang.com/2014/08/go_news_and_feature/)
1. 自己编译[tip版本源码](http://go.pkgdoc.org/)
2. 使用新特性

> 利用Docker，可以自己写一个学习环境的镜像。

## [源码目录说明](http://blog.studygolang.com/2012/12/%E6%8E%A2%E7%A9%B6go%E4%B8%AD%E5%90%84%E4%B8%AA%E7%9B%AE%E5%BD%95%E7%9A%84%E5%8A%9F%E8%83%BD/)
- api/
- doc/
- lib/time/
- misc/
- **src/：Go源码**
    - cmd：工具源码
- test/

## gdb的使用
- [GDB调试GO程序](http://blog.studygolang.com/2012/12/gdb%E8%B0%83%E8%AF%95go%E7%A8%8B%E5%BA%8F/)
- [Mac 调试 golang 程序](http://www.do1618.com/archives/771/)
- [理解Golang多重赋值](https://studygolang.com/articles/20485)
- **[go-delve](https://github.com/go-delve/delve/)**：推荐使用
- [Debugging what you deploy in Go 1.12](https://blog.golang.org/debugging-what-you-deploy)：提到了Delve的使用

## 源码相关
- [源码安装GO的过程](http://blog.studygolang.com/2013/01/%E5%88%86%E6%9E%90%E6%BA%90%E7%A0%81%E5%AE%89%E8%A3%85go%E7%9A%84%E8%BF%87%E7%A8%8B/)
- [Go如何测试标准库](https://mzh.io/go-test-standrad-library-changes-zh-CN)

# 学习资料
- [《浅谈 Go 语言实现原理》](https://draveness.me/golang/)：推荐学习
- [《Go语言标准库》](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example)
- [《 刻意学习 Golang - 标准库源码分析 》](https://learnku.com/articles/25470)
- 文档镜像
    - [https://golang.google.cn/pkg/](https://golang.google.cn/pkg/)
    - [https://studygolang.com/pkgdoc](https://studygolang.com/pkgdoc)
    - [https://gowalker.org/go](https://gowalker.org/go)

# Changelog
- 2019/06/30：整理格式