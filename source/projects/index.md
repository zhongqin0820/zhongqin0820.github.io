---
title: projects
date: 2019-05-03 21:52:12
updated: 2019-09-22 12:27:26
comments: false
layout: projects
---
# 个人项目：AddMeDouban（Go）
- 时间：2019.8-至今

## 描述
基于豆瓣的交友平台，通过IP得到对应的地理信息，用户可以通过指定性别属性以及年龄段和地理信息来搜索用户

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
从豆瓣用户中过滤得到和自己具有一定匹配度（共同喜好）的用户。 

## 所用技术
- 网页解析相关的内容：BeautifulSoup以及正则表达式；
- 数据存储：SQLite
- 部署：Docker以及Shell脚本和项目配置相关模块configparser
- 项目地址：[zhongqin0820/2a-sieve-4db](https://github.com/zhongqin0820/2a-sieve-4db)

## 其它
- 为项目撰写较为完善的[文档](https://github.com/zhongqin0820/2a-sieve-4db)

# 实习项目：告警系统（Node.js）
- 时间：2018年7月-8月

## 所用技术
- 涉及硬件Raspberry Pi的环境搭建、告警生态系统搭建、以及核心业务系统实现。

### 外部系统搭建
#### 硬件Raspberry Pi的环境搭建
- Raspberry Pi（采集IoT数据）
- Python 3（采集IoT数据，并使用requests模块发送至influxdb）

#### 告警生态系统搭建
- Grafana v5（数据绘制面板）
- InfluxDB v1.7（时序型数据库）

### 核心业务系统实现
- ~~nginx~~：由于部署在云平台[heroku](https://www.heroku.com)，所以实际未使用。
- Node.js v8
- MongoDB v3
- Docker（自动化部署）

## 实现细节
下面是整个系统的示意图，点击查看大图。

<div style="width: 300px; margin: auto">![系统示意图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/site/project/projects_illustration.png)
</div>

### 外部系统
#### 硬件平台Raspberry Pi
[代码及相关说明地址](https://github.com/XiaoguTech/rta-simulator/tree/master/iot)。
搭建硬件数据采集平台，并使用Python处理数据后，利用requests模块发送至InfluxDB。功能包括：
- 硬件搭建：主要涉及使用组件的接线问题
- 数据采集：根据使用模块数据的传输位置读取数据（所用DS18B20通过总线传输数据，查询手册找到对应的地址）
- 数据发送：可以使用requests模块将数据发送给InfluxDB所在服务器

#### InfluxDB与Grafana
[代码及相关说明地址](https://github.com/XiaoguTech/rta-simulator)：包括一些配置修改。部署时，在修改部分配置下，使用其Docker file构建得到Docker image后运行容器实例。
- 主要根据InfluxDB中的数据，拉取数据进行绘制，为了能够公开访问图表，使用匿名组织画图

### 告警业务系统
- 基于[OpenKB](https://github.com/mrvautin/openKB)进行再开发得到。项目代码地址：[GitHub](https://github.com/XiaoguTech/rta-monitor)。
- 主要由两部分组成：简单的监控系统展示+简单的后台管理。
    - 告警展示与推送： 可以查看不同分类下的数据监控信息以及自动获取报警推送和查看对应报警的解决方案文章。
    - 添加告警平台组织的管理
    - 添加告警平台组织下用户的管理
    - 添加告警面板与告警帮助文章的管理：主要是Grafana对应panel链接和OpenKB中帮助文档的关联管理

## 其它
在完成上述系统搭建之余，完成关于了90%的[Grafana汉化工作](https://github.com/XiaoguTech/grafana)，很可惜没能坚持完成。

## 不足
- 本次项目涉及内容庞杂，因此对所用技术原理了解不深入，停留在表面。但是收获还是很多的，学习到了很多新技术。
- 另外，由于项目没有投入生产，对代码质量不够关注，整个构建过程代码质量不高。

# 实训项目：图片分类分割(半个月)
- 时间：2017年7月
- 源码地址：无（项目主要侧重概念的理解）

## 所用技术
- 涉及：网络爬虫、图像分类、图像分割
    - Python 2，Matlab
    - Redis
    - TensorFlow

### 网络爬虫实现
- 使用requests模块，爬取Bing图片中的图片
    - 请求头字段设置
    - 涉及代理IP的使用
- 使用Redis存储数据信息，由于数据量较小，实际中意义不大
- 对数据进行预处理：包括切割和删选图片，最终形成10个类别，每个类别200张图片的小数据集（类似cifar10）。

### 图像分类
- 使用特征工程提取图像特征（颜色、边缘信息）
- 使用简单线性分类模型对图像进行分类

### 图像分割
- 使用经典的图像分割算法对图像进行分割
- 利用图像分类阶段得到模型提取得到语义掩膜得到显著图像

### Challenge阶段
- 使用LeNet 5直接进行图像分类实验
- 使用Grabcut方法分离前背景

## 不足
- Challenge阶段，没有用到图像分类部分的内容

# 课程设计：一个简单类C文法解释器的实现(一个月)
- 时间：2016年12月
- 源码地址：[GitHub](https://github.com/Zokingo/compiler)

## 所用技术
- Java
- 编译原理知识

## 实现细节
该项目为编译原理课程设计项目，涉及内容大多与其相关(词法、语法、语义分析、优化等)。篇幅较大，更多细节请看[项目文档](https://raw.githubusercontent.com/Zokingo/compiler/master/documents/reports.pdf)。

## 不足
- 未完成编译成机器码部分内容，只是一个解释器
- 测试方法不够自动化

# 大创项目：抠图系统(一年)
- 开始时间：2016年9月
- 源码地址：[GitHub](https://github.com/zhongqin0820/GrabcutQt)
    - 在完成抠图的基础之上，加入图片合成的小功能。
    - 点击查看[系统演示](https://raw.githubusercontent.com/zhongqin0820/GrabcutQt/master/res/Jietu20170919-180439-HD.gif)。

## 所用技术
- Qt 5
- C++

## 实现细节
### 主要挑战
- 抠图算法[GrabCut](https://cvg.ethz.ch/teaching/cvl/2012/grabcut-siggraph04.pdf)的理解与实现
- 可视化界面实现
    - 抠图，抠取显著前景图像
    - 合成，从背景图像列表中选取背景与抠取的前景图像合成新的图像。

### GrabCut实现
- 基于OpenCV中的实现，进行扩展可视化界面。由于没有改进算法，所以大创项目终期答辩没有通过。
- 项目涉及的概念：
    - 显著性计算：传统的显著性计算策略，主要是颜色空间以及边缘线的特征。
    - GMM：根据显著性计算结果，生成高斯混合模型，计算像素点间的关系
    - GrabCut：利用生成的混合模型对前背景像素点（super pixel graph）进行分类（涉及最大流最小割算法），涉及到一些预处理和后处理的策略，如生物腐蚀操作等。

## 不足
- 美化系统UI
- 改进算法性能，显著性的计算部分可以替换成利用深度网络提取的图片特征（显著性）。
- 真正意义的自动抠图，主要是在预处理和后处理方面的改进。
- 优化图片合成部分的功能

# 单机游戏：消除类小游戏(一个半月)
- 开始时间: 2016年4月（独立完成）
- 源码地址：[GitHub](https://github.com/zhongqin0820/linkgameplus)
    - 项目共有两个版本，一个是消除麻将素材，一个是消除糖果素材（含有关卡模式）。

## 所用技术
- Cocos2d-X：v3.6
- C++

## 实现细节
### 核心概念
- 移动：控制素材向空白区域的目标方向作直线移动，遇到障碍或到达指定空格则停止
- 消除：每次消除3个符合**消除条件**的素材，或者消除1个奖励素材（获得一次额外的移动机会，奖励只累积一次）
- 胜利条件：在规定时间內，完成所有素材的消除
- 失败条件：反之

### 核心玩法
- 如果选中3个素材，检查选中的素材是否符合消除条件：当它们中的一个最多向一个方向上移动若干步可以使得选中的素材连成一条直线，且这三个素材ID符合取模3的结果相等。如下图所示。
- 如果选中的素材是加成素材：则添加一次移动机会
- 每次消除完之后必须移动一个素材。

<div style="width: 300px; margin: auto">![消除规则示意图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/site/project/help_tip2.png)
</div>

### 挑战
#### Cocos2d-X 框架及相关生态的学习曲线
- 在实现过程中，主要涉及到关于Cocos2d-X的框架的学习与理解（包括用到的一些设计模式：单例模式、工厂模式、观察者模式等，以及一些游戏实现方面的术语：场景、精灵、地图等）。
- 因为，这是第一次接触到关于游戏引擎等相关概念，所以这个过程大概花费了两周的时间。后续编码，由于本身游戏实现相对简单，所以用的时间反而较少。而由于缺少游戏静态资源，因此准备的过程也花费了一些时间。

#### 游戏资源准备
- 主要包括：图片资源、字体资源以及声音及音效资源的获取。这个过程大致花费了两周时间。

#### 主要内容
- 游戏加载界面：游戏进入页，显示加载字样以及随机产生游戏技巧说明。
- 游戏主场景：加载关卡信息
- 关卡选择界面：显示当前关卡信息以及关卡选择
- 设置界面：系统、游戏內的设置界面
- 帮助界面：显示游戏帮助信息
- 关于界面：介绍游戏信息

以下是页面跳转示意图：
<div style="width: 300px; margin: auto">![游戏页面跳转示意图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/site/project/page_goto.png)
</div>


## 项目收获
- 对Cocos2d-X游戏引擎框架有了一定的了解
- 相比传统的消除游戏（如：连连看），该游戏在玩法上有一定的创新。
- 第一次作出一个基本完整的小项目，并将其发布在应用商店（已下架）。如果你感兴趣，这里有一份[可执行文件](https://pan.baidu.com/s/1053E-eDhjeaRVCtPOfcvuA)可供尝试，提取码: u454。

## 不足
- 使用XML进行本地的数据存储。
- 玩法涉及移动，与传统消除游戏区别较大，一开始玩家可能不习惯。
