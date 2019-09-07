---
title: UCF-CV课程总结:Harris Corner Detector
date: 2017-09-10 16:15:26
updated: 2017-09-10 16:15:26
categories:
- 计算机视觉

tags:
- 课程总结
---
# 前言
这篇博文是本人学习UCF计算机视觉公开课的课程知识梳理系列文章的第三篇。本篇博文主要介绍[哈里斯角点检测算法](https://senitco.github.io/2017/06/18/image-feature-harris/)章节的内容。

<!--more-->
# 兴趣点(特征点)检测及其应用
在做机器学习训练模型时或者视觉中的一些应用：人脸识别、物体检测等，常常将一张图像提取出一组代表性的特征更好的表示这张图像。特征分为以下两种：

- 全局特征：颜色特征(直方图)、纹理特征、形状特征(全局特征容易受到环境的干扰，光照，旋转，噪声等不利因素都会影响全局特征)
- 局部特征：Harris、SIFT、SURF、HOG(是图像特征的局部表达，它只能反正图像上具有的局部特殊性，所以它只适合于对图像进行匹配，检索等应用)，接下来的几篇介绍的都是关于局部特征的内容。

局部特征在全局特征中目标被遮挡或者背景过于复杂等情况下能够发挥作用，使算法对输入图像的要求降低。

# Harris Corner Detector
具有旋转不变性，不具有尺度不变性。以下，再说几个关于harris算法的概念：<b>角点</b>是图像中一边物体的拐角或者线条之间的交叉部分。
可以用作人脸关键点检测。

# 程序实现
- [Harris Corner Detector version 1.0.0.0 (344 KB) by Ali Ganoun](https://www.mathworks.com/matlabcentral/fileexchange/9272-harris-corner-detector)

# 引用
0. [图像处理特征不变算子系列之Moravec算子（一）](http://blog.csdn.net/kezunhai/article/details/11176065)
1. [Harris角点检测原理分析](http://blog.csdn.net/newthinker_wei/article/details/45603583)
2. [图像处理之角点检测算法(Harris Corner Detection)](http://blog.csdn.net/jia20003/article/details/16908661)
3. [Harris角点算法](http://www.cnblogs.com/polly333/p/5416172.html)
4. [图像算法之二：特征提取算法系列之Harris](http://blog.csdn.net/SoaringLee_fighting/article/details/52670035)
5. [Harris角点检测算法 1](http://www.cnblogs.com/ztfei/archive/2012/05/07/2487123.html)
6. [Harris角点](http://www.cnblogs.com/ronny/p/4009425.html)
