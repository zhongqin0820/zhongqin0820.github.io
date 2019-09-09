---
title: 做一个优雅的Markdown工程师
date: 2019-07-27 18:29:46
updated: 2019-07-27 20:58:11
categories:
- 其它

tags:
- 修复问题
---
# 前言
静态博客生成器通常通过解析用户的Markdown来生成博客内容，如果希望像软件开发中那样对文章进行小版本更新。一种做法是在源码中显式声明`TODO`之类的字样，但是这些内容会与正文一起被解析成网页正文影响文章阅读。同时，随着本地博文数量的增多，这种做法会造成管理困难从而失去便利性。本篇文章介绍的方法将有效解决上诉问题。

<!-- more -->
# 解决方案
在写工程代码时，可以使用注释以及TODO之类的功能来组织管理我们的代码，对Markdown的编辑同样也支持这种做法。通过在Markdown源码中配合使用注释，同时引入TODO，FIXME之类的提醒功能，从而达到只发表我们认为足够完善的部分内容而忽略尚未或希望之后完善的内容。

本文内容基于Intellij公司IDE，如IDEA中的Live Templates，TODO功能以及Markdown本身的注释语法对如何搭建一个静态博客内容创作过程的管理流程进行说明。

# 搭建流程
## 相关环境
- IDE：Intellij公司IDE均可，下文中以IDEA为例说明
- 静态博客生成器：Hexo

## 搭建细节
### Markdown
Markdown利用隐藏URL的特性，本身只要是形容`[...]: # (...)`格式的代码都会被解析为注释，即不会被渲染；但是，IDEA与Hexo（默认Github Flavored Markdown）中支持的Markdown注释方式如下所示，并且只允许单行注释。
```markdown
[//]: # (this is a comment)
```

### Live Templates
使用IDEA支持的Live Templates生成自己的注释风格代码片段，注意**注释需与上下正文间隔一行**，否则不会被识别为注释，一个正确的样例如下：
```markdown
正文A

[//]: # (关键字:注释者:$date$)

正文B
```
其中，各字段对应说明如下：
- `关键字`：对应IDEA TODO中的`Patterns`正则，配合自动识别检测
- `注释者`：不是必须，注释者姓名
- `$date$`：不是必须，Live Templates中的自定义变量，生成当前时间。关于Live Templates，可以参考W3C的[IntelliJ IDEA实时模板怎么创建](https://www.w3cschool.cn/intellij_idea_doc/intellij_idea_doc-hnk72eaw.html)

### TODO
在IDEA的TODO中添加正则Pattern：`\b关键字\b`，`\b`表示匹配字段的开头与结尾。篇幅有限，这里不对IDEA的TODO特性展开进一步介绍，可以参考W3C的[IntelliJ IDEA如何使用TODO](https://www.w3cschool.cn/intellij_idea_doc/intellij_idea_doc-nx4b2dto.html)

[//]: # (TODO:Zoking:2019/07/27)
[//]: # (this is a hidden comment)
[//]: # (DUE:Zoking:2019/07/27)

# 效果展示
网页渲染效果如你所见，此处展示本地Markdown原文，以及IDEA中的TODO窗口。

## Markdown
通过预览源码，我们可以看到只显示了对应注释中的TODO而不会显示标题中的TODO，这里我额外定义了一个`DUE`字段对`TODO`进行期限声明，方便后续管理。
<div style="width: 300px; margin: auto">

![Markdown源码](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/tools/source.png)
</div>

## TODO
TODO窗口中则可以配合过滤操作等合理管理源码中的内容。
<div style="width: 300px; margin: auto">

![TODO窗口](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/tools/todo.png)
</div>

# Changelog
- 2019/07/27：第一版内容