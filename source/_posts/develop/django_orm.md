---
title: Django之数据库
date: 2017-12-08 17:56:10
categories:
- 网页开发

tags:
- Django
---
# 前言
关于在Django框架中如何使用数据库。

<!-- more -->
# 数据库的使用
## 定义模型
### 创建 APP
Django规定，如果要使用模型，必须要创建一个app。修改对应App目录下的`models.py`文件。
```py
# models.py
from django.db import models
 
class Test(models.Model):
    name = models.CharField(max_length=20)
```
> 注解：类名代表了数据库表名，继承了`models.Model`。
> 类里面的字段代表数据表中的字段(name)
> 数据类型则由CharField（相当于varchar）、DateField（相当于datetime）
> max_length 参数限定长度。

### 在命令行中运行
```bash
python manage.py migrate   # 创建表结构
python manage.py makemigrations TestModel  # 让 Django 知道我们在我们的模型有一些变更
python manage.py migrate TestModel   # 创建表结构
```
表名组成结构为：**应用名_类名**（如：`TestModel_test`）。

** 注意：尽管我们没有在models给表设置主键，但是Django会自动添加一个id作为主键。**
## 数据库操作
在 HelloWorld(所创建的App)目录中添加 testdb.py 文件（下面介绍），并修改 urls.py
> 注解：此处只要在对应App目录下创建好操作数据库的文件，以及在urls.py中处理好对应的路由转发规则即可。并不强制要求一定是这样。但是路由的映射一定要对应上。
 
### 添加数据
添加数据需要先创建对象，然后再执行 save 函数，相当于SQL中的`INSERT`
```py
# -*- coding: utf-8 -*-
 
from django.http import HttpResponse    #因为是处理客户端http请求，因此需要引入该模块
 
from TestModel.models import Test    #该模块中定义了我们的表结构（模型），可以理解为是一个类（自定义的数据类型）
 
# 数据库操作
def testdb(request):    #此处对应urls.py中的路由处理函数，一定要对应上
    test1 = Test(name='runoob')    #创建了一个实例对象
    test1.save()    #将实例对象保存（可以看作是SQL中的INSERT操作）
    return HttpResponse("<p>数据添加成功！</p>")    #如只是用HttpResponse则无需在参数中添加request。如果是render等则需要带上
```
### 获取数据
通过objects这个模型管理器来进行完成对应的操作
all()获得所有数据行，相当于SQL中的SELECT * FROM
filter相当于SQL中的WHERE，可设置条件过滤结果
objects.get(id=1)获取单个对象
objects.order_by('name')[0:2]限制返回的数据 相当于 SQL 中的 OFFSET 0 LIMIT 2;
objects.order_by("id")数据排序
上面的方法可以连锁使用

### 更新数据
修改数据可以使用 save() 或 update()
> save()和update都得针对已经有的数据进行操作
 
### 删除数据
删除数据库中的对象只需调用该对象的delete()方法即可


# 结束语
Django中的数据库操作还是挺简单的，但是封装感觉会是一个问题，单纯的利用框架提供的这个似乎解耦等不会很好，有待继续学习。

# 参考链接
[http://www.runoob.com/django/django-model.html](http://www.runoob.com/django/django-model.html)