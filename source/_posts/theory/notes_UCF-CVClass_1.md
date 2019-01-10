---
title: UCF-CV课程总结:概述、滤波及Matlab使用
date: 2017-09-01 17:20:40
updated: 2017-09-01 17:20:40
categories:
- 计算机视觉

tags:
- 课程总结
---
### 前言
&emsp;&emsp;从八月开始，断断续续学习这门课程。从一开始Coursera上乔治亚理工的CV课和CP课继而到UCF的CV课。算是入门了这门课程，为以后的研究工作打下了一些基础。虽然，我的数学基础在很大程度上限制着我的吸收消化。但是，始终相信，有志者事竟成这个道理。
<br/>&emsp;&emsp;课程资源地址：[UCF-CAP 5415 - Computer Vision](http://crcv.ucf.edu/courses/CAP5415/Fall2012/)学习过程中，可以参照课程日程表安排学习时间。适当的加强强度。(由于时间问题，本人只完成了小部分实验...最好还是要做做实验的。)

<!--more-->

<br/>&emsp;&emsp;虽然在onenote上针对每节课都已经做了笔记，但是发现，自己只是将教授PPT中的内容摘抄了出来。这里借助博客<u>简单梳理一下这门课中学习到的内容</u>(事实上，其中的每一课都值得另起一篇去详细介绍)，顺便提升自己的写作能力总结能力。
<br/>&emsp;&emsp;我发现，本文的长度大大超过了预期。因此，基于"2-8准则"(实则我实在无法做到一小时完成这个工作)，计划每篇此类总结文章以两到三课为一篇文章去总结课程内容。
### 课程内容
&emsp;&emsp;整门课内容设计较为合理，作为重实现轻理论(少数学细节推导，乔治亚理工的课程较侧重于数学细节)大军中的一员，接受度良好。在课程主页上还有参考资料(免费，开放下载)以及CV领域的顶刊顶会的官网链接等信息。

#### Lecture1 CVIntroduction

&emsp;&emsp;第一课是关于计算机视觉中基础知识的介绍以及整个课程会接触到的内容的学习。了解计算机视觉所研究的是从二维图像中得到三维世界的语意信息。
<br/>![The Ability of Computer to See](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/1%E8%AE%A1%E7%AE%97%E6%9C%BA%E8%A7%86%E8%A7%89%E9%A2%86%E5%9F%9F.png)
<br/>&emsp;&emsp;在这个过程中，常常将图像当作一个函数$f(x,y)$,它是一个离散的关于像素点的函数。常用一个矩阵表示一张图像，其元素是0(黑)～255(白)的灰度值。分辨率则是由图像的像素长宽确定(屏幕的分辨率通常是固定的，但是可以向下调整，分辨率越高图像越细腻)。还介绍了图片的格式(不同的格式只是压缩编码的方式不同)。也介绍了视频的基本属性。依照人眼的构造，视频帧数每秒30帧就可以达到流畅的基本要求。
<br/>&emsp;&emsp;现今的射影设备都是基于孔洞模型(Pin Hole)进行射影变换。
<br/>![Pin Hole Model](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/2Pin%20Hole%20Model.png)
<br/>&emsp;&emsp;三维空间中的点$(x,y,z)$通过镜片投影到图像平面上的点$(x,y)$
<br/>&emsp;&emsp;由以上这个模型，我们就很容易想到：应该如何通过二维图像恢复被摄物体在三维世界中的几何信息呢？
<br/>![三维重构理论](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/3%E4%BB%8E%E4%BA%8C%E7%BB%B4%E5%9B%BE%E5%83%8F%E6%81%A2%E5%A4%8D%E4%B8%89%E7%BB%B4%E4%BF%A1%E6%81%AF.png)
<br/>&emsp;&emsp;这里不再细展开。以及，CV的应用有哪些：
<br/>![CV Application](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/4CV%20Applicatioin.png)
<br/>&emsp;&emsp;作为基础知识的简单入门，第一节课还是很不错的。至少罗列的那些应用光是看那些演示视频，想到自己也有机会去实现。光想想就能振奋人心。

#### Lecture2 Filtering
&emsp;&emsp;第二节课是CV领域最最基础的一个概念--滤波(Filtering)。我的[第一篇博文](https://cvblogs.cn/2017/08/11/image-processing-operation1/)整理的就是关于滤波操作的知识(虽然大部分是copy的别人的总结...)。
<br/>&emsp;&emsp;首先，当你拿到一张图像，一定要区分清楚它究竟是:1.二值的(Binary)、2.灰度的(Gray Scale)还是3.彩色的(Color)。
<br/>&emsp;&emsp;1.二值图像矩阵元素要么是0要么是1
<br/>&emsp;&emsp;2.灰度图像，Scale范围通常(0~255)
<br/>&emsp;&emsp;3.彩色图像较为复杂，通常在RGB空间下分为3个通道(R-Red、G-Green、B-Blue)
<br/>&emsp;&emsp;因为这些值元素的存在，引入一个图像直方图去直观描述它的比重信息。
<br/>&emsp;&emsp;![直方图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/5%E7%9B%B4%E6%96%B9%E5%9B%BE.png)
<br/>&emsp;&emsp;由于各种内外界不稳定因素的干扰，图像并不总是"干净"的,总是有这样那样的噪声(通常高频信息是噪声-变化明显，变现的特征就是高频信息)。噪声叠加原图像得到被污染后的图像。(高斯噪声即噪声的分布符合高斯分布！)
<br/>&emsp;&emsp;![噪声来源](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/6%E5%99%AA%E5%A3%B0%E6%9D%A5%E6%BA%90.png)
<br/>&emsp;&emsp;接着为引入滤波的概念，介绍了基本的数学运算：求导(Derivative，包括连续的和离散的即查分：向前、向后、中心，接着从一维引申到二维（图像是一个二维矩阵），接着提到了相关(Correlation)以及卷积(Convolution))以及求均值(Average/Mean)。
<br/>&emsp;&emsp;![求导(梯度)](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/7%E4%BA%8C%E7%BB%B4%E6%B1%82%E5%AF%BC%28%E6%A2%AF%E5%BA%A6%29.png)并且，还提到了掩膜(Mask)的概念。
<br/>&emsp;&emsp;高斯分布的重要意义：
<br/>&emsp;&emsp;![高斯属性](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/8%E9%AB%98%E6%96%AF%E6%A8%A1%E5%9E%8B%E5%B1%9E%E6%80%A7.png)
<br/>&emsp;&emsp;由于之前的文章已经梳理过，这里不再赘述。第二节结束。目前，我们已经了解了一些基本知识和基本运算(Matlab由库函数实现，推荐使用。)

#### Matlab Tutorial
&emsp;&emsp;作为课程的辅助章节，本节内容介绍Matlab的基本使用，作为Matlab的简单入门还是很有必要学习的。助教教授的内容都很实用。(对于没有接触过Matlab的人来说，通过学习本节可以完成基本编程任务。推荐观看。另外，斯坦福的吴恩达教授也在其机器学习视频中教授过Matlab的简单操作使用。也可以参考学习。)


### 结束语
&emsp;&emsp;写博客真是一项锻炼能力的活，花了一个半小时才写完两节课的简单梳理总结。这进度我也是醉了！看来，以后应该利用碎片化的闲暇时间一点点完成。否则太占用学习时间了！