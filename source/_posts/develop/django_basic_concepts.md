---
title: Django之基础概念学习
date: 2017-12-07 12:47:10
categories:
- 网页开发

tags:
- Django
---
# 前言
最近需要学习Django框架。笔记备忘。其实学习一个框架，莫过于自己动手从零开始搭建。亲身去体会这个框架。这样学习才能事半功倍。神奇的发现hexo的文件里不可以有那个跨站点的token标签= =

<!-- more -->
# 引用链接
[https://www.cnblogs.com/feixuelove1009/p/5823135.html](https://www.cnblogs.com/feixuelove1009/p/5823135.html)
[http://www.runoob.com/django/django-first-app.html](http://www.runoob.com/django/django-first-app.html)

# Web框架介绍
![一般的web框架](http://images2015.cnblogs.com/blog/948404/201608/948404-20160830212049246-1153317231.jpg)
其它基于python的web框架，如tornado、flask、webpy都是在这个范围内进行增删裁剪的。例如tornado用的是自己的异步非阻塞“wsgi”，flask则只提供了最精简和基本的框架。Django则是直接使用了WSGI，并实现了大部分功能

# MVC/MTV介绍
模型(model)：定义数据库相关的内容，一般放在models.py文件中。
视图(view)：定义HTML等静态网页文件相关，也就是那些html、css、js等前端的东西。
控制器(controller)：定义业务逻辑相关，就是你的主要代码。
MTV: 有些WEB框架觉得MVC的字面意思很别扭，就给它改了一下。view不再是HTML相关，而是主业务逻辑了，相当于控制器。html被放在Templates中，称作模板，于是MVC就变成了MTV。这其实就是一个文字游戏，和MVC本质上是一样的，换了个名字和叫法而已，换汤不换药。
# Django模型组织
![Django模型组织](http://images2015.cnblogs.com/blog/948404/201609/948404-20160903111840215-2065765780.jpg)
## 我们学Django学的是什么？

1. 目录结构规范

2. urls路由方式

3. settings配置

4. ORM操作

5. 模板渲染

6. 其它


# Django项目目录结构
![xiaogu_django项目结构.png](./xiaogu_django项目结构.png)
![目录结构](http://images2015.cnblogs.com/blog/948404/201608/948404-20160830222155746-982749621.png)
**所有的APP共享项目资源**
在每个django项目中可以包含多个APP，相当于一个大型项目中的分系统、子模块、功能部件等等，相互之间比较独立，但也有联系。
**编写路由**
路由都在urls.py文件里，它将浏览器输入的url映射到相应的业务处理逻辑
> 此处尚未理解透彻
 
**业务处理逻辑**
业务处理逻辑都在views.py文件里。通过上面两个步骤，我们将urls.py中的index这个url指向了views.py里的index（）函数，它接收用户请求request，并返回一个“hello world”字符串。实际上这肯定不行，通常我们都是将html文件返回给用户。

**返回HTML文件**
写这么一个index.html文件,再修改一下views文件。为了让django知道我们的html文件在哪里，需要修改settings.py文件的相应内容。但默认情况下，它正好适用，你无需修改。
> 注：这里有个小技巧，在多次频繁重启服务时，由于端口未释放的原因，容易启动不了服务，修改一下端口就OK了。
 
**使用静态文件**
我们已经可以将html文件返还给用户了，但是还不够，前端三大块，html、css、js还有各种插件，它们齐全才是一个完整的页面。
在django中，一般将静态文件放在static目录中。你的CSS,JS和各种插件都可以放置在这个目录里。为了让django找到这个目录，依然需要对settings进行配置。
同样，在index.html文件中，可以引入js文件了。

**接收用户发送的数据**
我们将一个要素齐全的html文件返还给了用户浏览器。但这还不够，因为web服务器和用户之间没有动态交互。

下面我们设计一个表单，让用户输入用户名和密码，提交给index这个url，服务器将接收到这些数据。
**注意：跨站保护机制**
django有一个csrf跨站请求保护机制，我们暂时在settings文件中将它关闭，或者在form表单里添加一个标签。

**返回动态页面**
我们收到了用户的数据，但返回给用户的依然是个静态页面，通常我们会根据用户的数据，进行处理后在返回给用户。

这时候，django采用自己的模板语言，类似jinja2，根据提供的数据，替换掉html中的相应部分，详细语法入门后再深入学习

**使用数据库**
流程走到这里，django的MTV框架基本已经浮出水面了，只剩下最后的数据库部分了。
django通过自带的ORM框架操作数据库，并且自带轻量级的sqlite3数据库。
1. 在settings.py里注册你的app(不注册它，你的数据库就不知道该给哪个app创建表)
2. 然后我们在settings中，配置数据库相关的参数，如果使用自带的sqlite，不需要修改。
3. 再编辑models.py文件，也就是MTV中的M。
4. 接下来要在pycharm的teminal中通过命令创建数据库的表
```bash
python manage.py makemigrations
python manage.py migrate
```
5. 修改views.py中的业务逻辑
6. 重启web服务后，刷新浏览器页面，之后和用户交互的数据都能保存到数据库中。任何时候都可以从数据库中读取数据，展示到页面上。

# 结束语
关于学习方法的建议：学习任何东西，不要直接扎入细节，应该先了解它的外围知识，看看它的整体架构，再学习它的基本内容，然后才是深入学习，打磨技巧！