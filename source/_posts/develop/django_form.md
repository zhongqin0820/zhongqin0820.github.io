---
title: Django之表单
date: 2017-12-08 18:15:18
categories:
- 网页开发

tags:
- Django
---
# 前言
HTML表单是网站交互性的经典方式。有必要区别与表格Table。这两个，本质上有很大的差别。

<!-- more -->

> 注解：
> 表格Table用于展示数据
> 表单Form则用于提交数据等
 
# HTTP请求
HTTP协议以"请求－回复"的方式工作。客户发送请求时，**可以在请求中附加数据**。服务器通过解析请求，就可以获得客户传来的数据，并根据URL来提供特定的服务。

## GET 方法
创建一个 search.py 文件，用于接收用户的请求。记得一定要将自己定义的路由处理函数在`urls.py`中定义的正则路由对应起来。
> 注解：此处将视图显示和请求处理分成两个函数处理。

## POST 方法
提交数据时更常用POST方法。
**用一个URL和处理函数，同时显示视图和处理请求。**
注意：csrf 全称是 Cross Site Request Forgery。这是Django提供的防止伪装提交请求的功能。POST 方法提交的表格，必须有此标签。

# Request 对象
**每个`view` 函数的第一个参数是一个 `HttpRequest` 对象**
注意：不能使用语句if request.POST来判断是否使用HTTP POST方法；应该使用if request.method == "POST" 。


# QueryDict对象
在HttpRequest对象中, GET和POST属性是django.http.QueryDict类的实例。
QueryDict类似字典的自定义类，用来处理单键对应多值的情况。