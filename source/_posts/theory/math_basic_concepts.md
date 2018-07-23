---
title: 数学基础：矩阵范数及latex公式编辑
date: 2017-09-25 16:22:10
categories:
- 图像处理

tags:
- 数学基础
---
### 前言
&emsp;&emsp;在学习过程中，发现似乎在最优化问题上常常通过范数进行计算。于是，这里简单梳理一下关于范数的基础知识。范数是数学中的一个基本概念(经常能看到它的身影，只不过它利用$\parallel \cdot \parallel $符号进行表示，使我们不易认出它。举个例子：我们常使用的，<font color=red>一维世界</font>中，衡量两点间的距离就是使用的1范数)，范数属于线性代数、泛函分析中的内容。掌握范数的基础知识对于以后的研究工作很有必要。<br/>
&emsp;&emsp;其常常被用来度量某个向量空间（或矩阵）中的每个向量的长度或大小。这里，我们将只讨论矩阵范数相关的内容。

<!--more-->
### 范数的基本性质
&emsp;&emsp;任何范数都必须满足以下三个性质:
1. 非负性
2. 齐次性
3. 三角不等式
&emsp;&emsp;矩阵范数还需满足相容性：$\parallel {XY} \parallel $$\leq $$\parallel X \parallel $$\parallel Y \parallel$。因此也称其为相容范数。<br/>

### 常用的三种$p$范数推导出的矩阵范数

#### 1-范数
&emsp;&emsp;也叫列和范数，A每一列元素绝对值之和的最大值。<br/>
<img src="http://latex.codecogs.com/gif.latex?\left&space;\|&space;A&space;\right&space;\|_1=&space;\max&space;\left&space;\{&space;\sum&space;\left&space;\|&space;a_{i1}&space;\right&space;\|,\sum&space;\left&space;\|&space;a_{i2}&space;\right&space;\|,\ldots,\sum&space;\left&space;\|&space;a_{in}&space;\right&space;\|&space;\right&space;\}" title="\left \| A \right \|_1= \max \left \{ \sum \left \| a_{i1} \right \|,\sum \left \| a_{i2} \right \|,\ldots,\sum \left \| a_{in} \right \| \right \}" />
&emsp;&emsp;其中，<img src="http://latex.codecogs.com/gif.latex?\sum&space;\left&space;\|&space;a_{i1}&space;\right&space;\|=\left&space;|&space;a_{11}&space;\right&space;|&plus;\left&space;|&space;a_{21}&space;\right&space;|&plus;\ldots&plus;\left&space;|&space;a_{m1}&space;\right&space;|" title="\sum \left \| a_{i1} \right \|=\left | a_{11} \right |+\left | a_{21} \right |+\ldots+\left | a_{m1} \right |" />&emsp;&emsp;其余类似.

#### 2-范数
&emsp;&emsp;谱范数，即$A^H$ * $A$特征值$\lambda_i$中最大者$\lambda_1$的平方根，其中$A^H$为$A$的转置共轭矩阵.<br/>

<img src="http://latex.codecogs.com/gif.latex?\left&space;\|&space;A&space;\right&space;\|_2&space;=&space;\left&space;(&space;\max&space;\left&space;\{&space;\lambda_i\left&space;(&space;A^{H}*A&space;\right&space;)\right&space;\}\right&space;)^\frac{1}{2}" title="\left \| A \right \|_2 = \left ( \max \left \{ \lambda_i\left ( A^{H}*A \right )\right \}\right )^\frac{1}{2}" />

#### ∞-范数：
&emsp;&emsp;（行和范数，A每一行元素绝对值之和的最大值）<br/>

<img src="http://latex.codecogs.com/gif.latex?\left&space;\|&space;A&space;\right&space;\|_\infty&space;=&space;\max&space;\left&space;(&space;\sum&space;\left&space;|&space;a_{1j}&space;\right&space;|&space;,\sum&space;\left&space;|&space;a_{2j}&space;\right&space;|,\ldots,\sum&space;\left&space;|&space;a_{mj}&space;\right&space;|&space;\right&space;)" title="\left \| A \right \|_\infty = \max \left ( \sum \left | a_{1j} \right | ,\sum \left | a_{2j} \right |,\ldots,\sum \left | a_{mj} \right | \right )" />
(其中$\sum \parallel a_{1j} \parallel $ 为第一行元素绝对值的和，其余类似);

#### 其它的p-范数
&emsp;&emsp;没有很简单的表达式。对于$p$-范数而言，可以证明:<img src="http://latex.codecogs.com/gif.latex?\left&space;\|&space;A&space;\right&space;\|_p&space;=&space;\left&space;\|&space;A^H&space;\right&space;\|_q" title="\left \| A \right \|_p = \left \| A^H \right \|_q" />
&emsp;&emsp;其中p和q是共轭指标。简单的情形可以直接验证：
<img src="http://latex.codecogs.com/gif.latex?\left&space;\|&space;A&space;\right&space;\|_1&space;=&space;\left&space;\|&space;A^H&space;\right&space;\|_{\infty}" title="\left \| A \right \|_1 = \left \| A^H \right \|_{\infty}" />
&emsp;&emsp;一般情形则需要利用:
<img src="http://latex.codecogs.com/gif.latex?\left&space;\|&space;A&space;\right&space;\|_p&space;=&space;\max&space;\left&space;\{&space;y^H*A*x:\left&space;\|&space;x&space;\right&space;\|_p&space;=&space;\left&space;\|&space;y&space;\right&space;\|_q&space;=&space;1&space;\right&space;\}" title="\left \| A \right \|_p = \max \left \{ y^H*A*x:\left \| x \right \|_p = \left \| y \right \|_q = 1 \right \}" />

### 引用
1. [百度百科](https://baike.baidu.com/item/%E8%8C%83%E6%95%B0/10856788?fr=aladdin)
2. [CSDN Markdown简明教程3-表格和公式](http://blog.csdn.net/whqet/article/details/44277965)
3. [机器学习中常见的字母解析及MarkDown代码](http://blog.csdn.net/sanqima/article/details/51275104?ref=myread)

### 结束语
&emsp;&emsp;才发现不总结真的不行，又把之前学过的知识忘记了。总之，在图像处理中，矩阵范数这个概念真的非常重要。
&emsp;&emsp;另外针对markdown中数学公式的书写真的是一大硬伤。需要花时间好好学习一下$latex$的语法。才发现百度百科里的公式都不是正规的$latex$书写的。
&emsp;&emsp;从四点开始写总结，八点才写完...$latex$公式真可怕。生产力杀手啊...最后还是靠的[在线编辑器](http://www.codecogs.com/latex/eqneditor.php)...