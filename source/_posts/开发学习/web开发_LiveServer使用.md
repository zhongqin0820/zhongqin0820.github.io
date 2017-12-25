---
title: web开发之LiveServer使用
date: 2017-11-12 22:15:10
categories:
- 网页开发

tags:
- Node.js
---
# live-server模块的作用
1. 实现热插拔`hot socketing`(修改文件之后，浏览器能够自动刷新)
2. 当服务启动时，自动打开所在的项目。(实现`opener`的功能)
3. 快速搭建临时的web服务。（实现`http-server`的功能）
<!-- more -->
我们主要使用1与2，3用express或者其他web框架。
# 安装
可以直接使用全局安装的方法
```bash
npm install -g live-server
```

# 使用
在`package.json`的`script`下添加一个`server`字段的`npm 脚本`。
```js
"scripts": {
  "server": "live-server ./ --port=9090"
}
```
使用`npm run server`执行。等待一会之后，浏览器会自动打开。
这样当你修改本地任何文件，浏览器都会自动立即同步（在内存层面，如果你想要发布代码，还是需要再次编译，否则修改之后的一些代码不会被改变！）。

# 继续学习
[官方链接https://www.npmjs.com/package/live-server](https://www.npmjs.com/package/live-server)