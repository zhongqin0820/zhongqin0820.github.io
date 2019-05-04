---
title: 数据库底层原理：MySQL、MongoDB、InfluxDB
date: 2019-04-16 16:16:34
updated: 2019-04-16 17:50:12
categories:
- 网页开发

tags:
- 基础概念
---
# 前言
几种典型数据库产品底层原理简单分析整理。

<!-- more -->
# 关系型数据库：MySQL
## 索引与底层数据结构
索引是一种数据结构，它完成字段对结果的查询映射，好的索引实现能够大大提升数据查询的效率。索引的实现通常基于平衡树（红黑树、B树、B+树都属于平衡树）。其**在存储引擎层被实现**。
查找操作的时间复杂度与树高h相关。O($h$)=O($log_d N$)，其中d为树节点的出度，出度越大时间复杂度越小效率越高。
完成插入、删除操作后，树的平衡会被破坏，因此需要在这些操作后对树进行调整使之恢复平衡。

### 底层数据结构
B+树由：二叉查找树(BS Tree) -> 平衡二叉树(AVL Tree) -> 平衡多路查找树(B-Tree)逐步优化而来。

#### 红黑树
红黑树（Red-Black Tree）是二叉查找树（Binary Search Tree）的一种改进。属于平衡二叉树。红黑树在每一次插入或删除节点之后都会花O(log N)的时间来对树的结构作修改，以保持树的平衡。

#### B Tree(B树)
B Tree即B-Tree，也可以被叫做平衡多路查找树。是为磁盘等外存储设备设计的一种平衡查找树。

#### B+Tree(B+树)
是应文件系统所需而产生的一种B树的变形树。数据按照左闭右开原则根据非叶子结点的索引区间存放在叶子结点。
拥有更低的磁盘读写代价以及更加稳定的查询效率。

### 索引
B+ Tree索引是大多数MySQL存储引擎的默认索引类型。不过不同引擎实现的具体方式不同。InnoDB在数据域中保存完整的数据记录。MyISAM引擎仅保存数据记录的地址。

其它索引还包括：哈希索引、全文索引、空间数据索引。可以参考[这里](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/MySQL.md#mysql-%E7%B4%A2%E5%BC%95)进一步了解。

## 存储引擎
### InnoDB
MySQL 默认的事务型存储引擎，只有在需要它不支持的特性时，才考虑使用其它存储引擎。

### MyISAM
考虑到其在写入与查询方面的优势，可以用于系统日志的收集。

### 二者的比较

|          |   InnoDB   |     MyISAM     |
| :------: | :--------: | :------------: |
|   事务   | 事务安全型 |  非事务安全型  |
|    锁    |   行级锁   |     表级锁     |
| 全文索引 |   不支持   |      支持      |
| 查询效率 |    较低    |      较高      |
|   外键   |    支持    |     不支持     |
|  count   |  遍历得到  | 存储表的总行数 |
| 保存格式 |     -      |    文件格式    |
|  安全性  |   更安全   |       -        |
|   索引   |  聚簇索引  |   非聚簇索引   |


# 非关系型数据库
主要讨论几种经典非关系型数据库产品的底层实现及特点，并与关系型数据库作简单的对比分析。
## 文档型数据库：MongoDB
文档的数据库，即可以存放xml、json、bson类型的结构化数据。
### 存储引擎及底层数据结构

### 与关系型数据库区别

## KV数据库：Redis
### 存储引擎及底层数据结构

### 与关系型数据库区别

## 时序型数据库：InfluxDB
### 存储引擎及底层数据结构
由于需要满足其高写入低读取的特点，使用LSM作为底层数据实现。

### 与关系型数据库区别

# 结束语
选择何种数据库产品用于数据存储应当基于特地的业务场景考虑。

# 参考资料
- [二叉查找树、平衡二叉树（AVLTree）和平衡多路查找树（B-Tree），B+树](https://blog.csdn.net/qq_21993785/article/details/80576642)
- [B+Tree在数据库索引上拥有独特优势的原因（为什么比红黑树更合适）](https://blog.csdn.net/qq_21993785/article/details/80580679)
- [MyISAM与InnoDB索引原理剖析](https://blog.csdn.net/qq_21993785/article/details/80582373)
- [MySQL面试知识要点](https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/MySQL.md)
- [数据库——索引、事务、存储引擎](https://blog.csdn.net/l_x_y_hh/article/details/81774316)

# TODO
- [ ] [MySQL、MongoDB、Redis 数据库之间的区别](https://blog.csdn.net/CatStarXcode/article/details/79513425)
- [ ] [Influxdb原理详解](https://www.cnblogs.com/gaoguangjun/p/8513054.html)
- [ ] [Go夜读：#25 OpenTSDB 时序数据引擎介绍](https://www.bilibili.com/video/av41250856)
