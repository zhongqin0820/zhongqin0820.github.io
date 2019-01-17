---
title: 深度学习框架: PyTorch
date: 2017-12-05 10:57:10
updated: 2017-12-05 10:57:10
categories:
- 深度学习

tags:
- 基础概念
---

# PyTorch

`PyTorch`准备数据有很多灵活的方式，在最后能够将数据转为`Numpy`数组即可，通过`Numpy`数据可以转化为`PyTorch`所需的`Tensor`。

- 对于图像数据，可以直接通过opencv，Pillow等做处理；
- 语音数据可以通过scipy和librosa来处理成numpy；
- 文本数据可以通过CNTK之类的自然语言处理库处理成numpy数组。

<!-- more -->
# SKImage

| 子模块名称        | 主要实现功能                           |
| ------------ | -------------------------------- |
| io           | 读取、保存和显示图片或视频                    |
| data         | 提供一些测试图片和样本数据                    |
| color        | 颜色空间变换                           |
| filters      | 图像增强、边缘检测、排序滤波器、自动阈值等            |
| draw         | 操作于numpy数组上的基本图形绘制，包括线条、矩形、圆和文本等 |
| transform    | 几何变换或其它变换，如旋转、拉伸和拉东变换等           |
| morphology   | 形态学操作，如开闭运算、骨架提取等                |
| exposure     | 图片强度调整，如亮度调整、直方图均衡等              |
| feature      | 特征检测与提取等                         |
| measure      | 图像属性的测量，如相似性或等高线等                |
| segmentation | 图像分割                             |
| restoration  | 图像恢复                             |
| util         | 通用函数                             |
