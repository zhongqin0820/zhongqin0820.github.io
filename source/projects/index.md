---
title: projects
date: 2019-05-03 21:52:12
updated: 2020-04-23 05:52:13
comments: false
layout: projects
---
# 个人项目：AddMeDouban（Go）
- 时间：2019.8-至今

## 描述
- 基于豆瓣的交友平台，通过IP得到对应的地理信息，用户可以通过指定性别属性以及年龄段和地理信息来搜索用户。

## 所用技术
- 后端：Web框架Gin，鉴权使用JWT，数据库使用MongoDB，设置的数据保留策略为一天
- 前端：Bootstrap，JQuery，HTML，CSS，ECharts.js
- 部署：利用Gitlab-CI结合Heroku进行自动化部署
- 在线地址：[https://lezen1.herokuapp.com](https://lezen1.herokuapp.com)

## 其它
- 为该项目封装了一个提升免费IP库内容的包：[zhongqin0820/ipdb-upgrader](https://github.com/zhongqin0820/ipdb-upgrader) 

# 个人项目：2a-sieve-4db（Python）
- 时间：2019.8.21-2019.8.24

## 描述
- 从豆瓣用户中过滤得到和自己具有一定匹配度（共同喜好）的用户。 

## 所用技术
- 网页解析相关的内容：BeautifulSoup以及正则表达式
- 数据存储：SQLite
- 部署：Docker以及Shell脚本和项目配置相关模块configparser
- 项目地址：[zhongqin0820/2a-sieve-4db](https://github.com/zhongqin0820/2a-sieve-4db)

## 其它
- 为项目撰写较为完善的[文档](https://github.com/zhongqin0820/2a-sieve-4db)
