---
title: UCF-CV课程总结:边缘检测
date: 2017-09-04 19:50:56
categories:
- 计算机视觉

tags:
- 课程总结
---
### 前言
&emsp;&emsp;这篇博文是本人学习UCF计算机视觉公开课的课程知识梳理系列文章的第二篇。本篇博文主要介绍<u>边缘检测</u>章节的内容。


<!--more-->


### 课程内容

#### 相关先导知识

&emsp;&emsp;边缘一般是指图像在某一局部强度剧烈变化的区域。强度变化一般有两种情况(横轴表示空间变化，纵轴表示灰度变化，虚线表示边缘)：
<br/>![阶跃变化：向亮渐变的过程](http://ou5rpeic9.bkt.clouddn.com/%E4%B8%80%E9%98%B6%E5%AF%BC%E6%95%B0)&emsp;&emsp;![屋顶变化：由暗到亮再变暗的过程](http://ou5rpeic9.bkt.clouddn.com/%E4%BA%8C%E9%98%B6%E5%AF%BC%E6%95%B0)
<br/>&emsp;&emsp;边缘检测的任务：找到具有阶跃变化或者屋顶变化的像素点的集合。
<br/>&emsp;&emsp;最直观的想法就是<b>求微分</b>。
- 对于第一种情况：一阶微分的峰值为边缘点，二阶微分的零点为边缘点。
- 对于第二种情况：一阶微分的零点为边缘点，二阶微分的峰值为边缘点。

<br/>&emsp;&emsp;一般可将边缘分为水平边缘、垂直边缘和对角线边缘。
<br/>&emsp;&emsp;由于边缘检测的算法主要是<b>基于图像强度</b>(采用差分的方法来进行计算)的一阶和二阶导数。
<br/>&emsp;&emsp;但导数通常对噪声敏感，因此必须采用滤波器来改善与噪声有关的边缘检测器的性能。常见的滤波方法主要有<b>高斯滤波</b>，即采用离散化的高斯函数产生一组<b>归一化</b>的高斯核。
#### 一般边缘检测过程
&emsp;&emsp;1. 滤波：改善与噪声有关的边缘检测器的性能
<br/>&emsp;&emsp;2. 增强：将图像灰度点邻域强度值有显著变化的点凸显出来(可通过计算梯度幅值来确定)
<br/>&emsp;&emsp;3. 检测：常用的方法是通过阈值化方法来检测

#### 边缘检测器介绍(Edge Detector)
&emsp;&emsp;课程中讲述到的检测器包括：
<br/>&emsp;&emsp;![边缘检测器分类](http://ou5rpeic9.bkt.clouddn.com/%E8%BE%B9%E7%BC%98%E6%A3%80%E6%B5%8B%E5%99%A8%E5%88%86%E7%B1%BB.png)
##### Prewitt and Sobel Edge Detector
&emsp;&emsp;属于离散微分算子 (discrete differentiation operator)。利用差分求出$x,y$方向上的梯度，求解梯度的大小，阈值化梯度的大小找到边缘。
###### Prewitt算子
&emsp;&emsp;结合了差分运算与邻域平均的方法。其卷积模板如下：
<br/>![Prewitt Operator](http://ou5rpeic9.bkt.clouddn.com/Prewitt%E6%A3%80%E6%B5%8B%E6%B5%81%E7%A8%8B%E5%9B%BE.png)
###### Sobel算子
&emsp;&emsp;与prewitt算子类似，但考虑到了相邻不同像素点的影响程度是不同的，所以采用加权平均。
<br/>![Sobel Operator](http://ou5rpeic9.bkt.clouddn.com/Sobel%E6%A3%80%E6%B5%8B%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

##### Marr Hildreth Edge Detector-LoG
&emsp;&emsp;基本步骤：
1. 用高斯器滤波原图像去除噪声
2. 拉普拉斯滤波后的图像(求$x,y$二阶导)
3. 找到零点(边缘点)，重复步骤2直到全部找到

<br/>![求梯度与拉普拉斯符号的区别](http://ou5rpeic9.bkt.clouddn.com/%E6%A2%AF%E5%BA%A6%E7%AC%A6%E5%8F%B7%E5%92%8C%E6%8B%89%E6%99%AE%E6%8B%89%E6%96%AF%E7%AC%A6%E5%8F%B7%E7%9A%84%E5%8C%BA%E5%88%AB.png)
<br/>&emsp;&emsp;laplacian算子是一些使用二阶微分的算子，实际上就是梯度的散度(可以利用DoG近似求解LoG降低算法时间复杂度的数量级?可以增强对比度！)
<br/>LoG计算：不同计算方法，时间复杂度不同
 1. 一个2D laplacian算子(n{2})
 2. 四个1D的算子组合而成(4n)

<br/>![LoG算子](http://ou5rpeic9.bkt.clouddn.com/LoG%20Operator)
<br/>&emsp;&emsp;由于 Laplacian使用了图像梯度，它内部的代码其实是调用了 Sobel 算子的
##### Canny
&emsp;&emsp;是一种求最优边缘检测的一套方法。是一种<b>先平滑再求导</b>的方法。
<br/>&emsp;&emsp;最优边缘检测的三个主要评价标准：
1. <b>低错误率</b>: 标识出尽可能多的实际边缘，同时尽可能的减少噪声产生的误报。
2. <b>高定位性</b>: 标识出的边缘要与图像中的实际边缘尽可能接近。
3. <b>最小响应</b>: 图像中的边缘只能标识一次，并且可能存在的图像噪声不应标识为边缘。

<br/>![Canny算法求解步骤](http://ou5rpeic9.bkt.clouddn.com/Canny%E7%AE%97%E6%B3%95%E6%B1%82%E8%A7%A3%E6%AD%A5%E9%AA%A4.png)
<br/>&emsp;&emsp;特别注意第4步！排除非边缘像素， 仅仅保留了一些细线条(候选边缘).

#### 程序实现
```matlab
    i = imread('lena.png');%读入原图像
    A = fspecial('gaussian');%高斯滤波器
    i = filter2(A, i) / 255;%高斯滤波并归一化(/255)
    figure;
    imshow(i);%显示图像
    imwrite(i,'lena_0.jpg');%保存图像
    %prewitt方法
    a = edge(i, 'prewitt');
    figure;
    imshow(a);
    imwrite(a,'lena_1.jpg');
    %sobel方法
    a = edge(i, 'sobel');
    figure;
    imshow(a);
    imwrite(a,'lena_2.jpg');
    %marr hildreth方法(LoG)
    a = edge(i, 'log');
    figure;
    imshow(a);
    imwrite(a,'lena_3.jpg');
    %canny方法
    a = edge(i, 'canny');
    figure;
    imshow(a);
    imwrite(a,'lena_4.jpg');
```
### 引用
1. [边缘检测算法](http://blog.csdn.net/xiahn1a/article/details/42141429)
2. [【OpenCV入门教程之十二】OpenCV边缘检测：Canny算子,Sobel算子,Laplace算子,Scharr滤波器合辑]()

### 结束语

<br/>&emsp;&emsp;一如既往，写文章对于我真的是一件万分困难的事情，基本上只能达到通顺的要求。翻来覆去也就只能用那几个markdown语法。排版也不甚美观。但我相信，坚持下去，总有一天，我能写出自己的内容。