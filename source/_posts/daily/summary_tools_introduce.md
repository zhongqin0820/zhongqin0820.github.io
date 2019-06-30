---
title: 工作开发中的一些工具推荐
date: 2017-11-04 22:08:10
updated: 2019-06-30 11:06:49
categories:
- 其它

tags:
- 开发总结
---
# 前言
内容包括浏览器相关的插件推荐，搜索引擎使用技巧以及几个不错的资源网站。以及阅读生态搭建相关工具，一些提升效率的工具和编码工具。

<!-- more -->
# 浏览器
## 插件推荐
浏览器的插件使浏览器的性价比得到提高。因此，当我们想要使用一个浏览器。我们应当要善于利用搜索引擎查询自己的需要对应的各家浏览器的插件都有哪些。或者直接搜索那种别人整理的插件推荐列表进行安装。好的插件大大提升我们的用户体验和工作效率。

| 名字 | 说明 |
| :--------: | -------- |
| AdBlock | 拦截网页上的广告 |
|Octotree|GitHub源码树|
|OneTab|减轻标签页混乱现象|
|RSS Feed Reader|配合[rsshub](https://docs.rsshub.app/)使用|
|沙拉查词|专业划词翻译扩展|

## 搜索引擎使用技巧
- `filetype`：指定文件类型；如：`a tale of two cities filetype:pdf`
- `site`：指定搜索的网站；如：`a tale of two cities site:goodreads.com`
- `inurl`：指定网址中带有的关键字，相较`site`更方便一些；如：`a tale of two cities inurl:goodreads`
- `intitle`：指定标题中带有的关键字；如：`a tale of two cities intitle:dickens`

## 实用网站
- [thatseed](https://www.thatseed.org/)：VPS服务提供商
- [gen.lib.rus.ec](http://gen.lib.rus.ec/)：英文电子书资源
- [bookset](https://bookset.me/)：中文电子书资源

# 阅读生态搭建相关工具
- ~~Calibre：本地图书管理，可以管理由其它阅读媒介导出的图书笔记等。弃，偶尔使用~~
- [Goodreads](https://www.goodreads.com/)或[豆瓣](https://www.douban.com/)：在线图书管理（均有对应的APP支持），前者主要是英文，后者侧重中文；
- 不论是Kindle/iBooks的阅读生态都做的非常好，择一即可，贵在坚持。（推荐设置：一次只保留一本书在设备上）

## iBooks
- 阅读媒介
    - 移动端APP
    - PC端
- 打开iCloud同步设置后，媒介间所有内容无障碍同步
- 缺点：
    - 依赖苹果生态，对导出笔记的支持不够友好

## Kindle
- ~~弃，偶尔使用~~
- 阅读媒介：需要记录下对应设备的`Kindle Email Address`配合进行资源推送
    - 移动端APP/Kindle设备
    - PC端
- 推送对应的内容到相应的设备邮箱即可收到对应的内容
- 缺点：
    - 用户友好性较差：一开始的生态建立的复杂度较高，不同区的用户的资源访问权限不同，中国区的用户可能找不到一些只有其他区用户可以访问的资源；
    - 格式支持不够友好：不支持`epub`格式；如果`amz3`格式的书籍资源不是官方的，则同步性不好。

# 其他工具
- [英语学习](https://cvblogs.cn/2019/04/14/daily/summary_english_recommendation/)：听说读写相关
- 文档/笔记
    - Office办公软件：文档编辑
    - OneNote笔记：三端同步笔记
    - XMind：思维导图应用
    - ~~百度脑图：免费在线思维导图应用。弃~~
- 科学上网
    - Shadowsocks：需要配合VPS使用，推荐[thatseed](https://www.thatseed.org/)。
    - ~~lantern：翻墙工具！GitHub下载。弃~~
- 快捷/文件查找
    - Alfred/Launchy：前者是Mac平台下的可配合工作流使用，后者是Win平台下的
    - EasyFind/Everything：前者是Mac平台下的，后者是Win平台下的
- 终端
    - Go2Shell：快速从Finder进入终端，可以自定义终端类型，默认系统自带终端
    - iTerm2：代替系统自带终端
    - tmux：终端神器，划分终端窗口
    - z：终端神器，代替cd命令
- 其它
    - CheatSheet：长按Command键查看当前软件快捷键
    - PhotoShop：抠图练习

# 编码工具：IDE/编辑器
无论是使用IDE还是编辑器都应该花心思熟悉快捷键以及搭配插件使用。
- 用过的IDE
    - IDEA：Node.js项目
    - PyCharm：Python项目
    - ~~Goland：Go项目 弃~~
    - ~~VS2013->VS2017：C、C++ （主要是拿Cocos-2dx写的游戏）、 C# 弃~~
    - ~~Eclipse：纯粹的Java开发，只用一个Java Swig...弃~~
    - ~~XCode：C、C++、iOS开发弃~~

- 用过的编辑器
    - Sublime Text 3：搭配插件使用
    - Vim：终端下，搭配插件使用，推荐NerdTree + TagBarToggle
    - Notepad++：win上使用，兼容Matlab文件最好
    - ~~Atom：弃~~
    - ~~VS Code：弃~~

# Changelog
- 2019/06/05：重新排列整理排版
- 2019/06/30：重新排版，添加英语学习的索引以及对阅读生态搭建相关工具方面的整理