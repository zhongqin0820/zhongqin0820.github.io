---
title: 图像处理基础-滤波(filtering)
date: 2017-08-11 19:39:53
updated: 2017-08-11 19:39:53

category:
- 图像处理

tags:
- 数学基础
---

### 前言
&emsp;&emsp;本文介绍图像处理基础知识滤波(filtering)相关知识。在[图像处理](https://www.wikiwand.com/zh/%E5%9B%BE%E5%83%8F%E5%A4%84%E7%90%86)中，你经常能够看到它的身影。对于学习图像处理，理解滤波背后的数学知识非常重要。

&emsp;&emsp;滤波分为<u>基于空间域滤波</u>(spatial filtering)与<u>基于频率域滤波</u>(frequency filtering)。二者的目的都是为了改善图像质量。篇幅有限，此篇文章先介绍空间滤波。

### 空间滤波(Spatial Filtering)
&emsp;&emsp;空间滤波操作有时候也被叫做<font color=#0099ff>卷积滤波</font>，或者干脆叫卷积(convolution)(离散的卷积，不是微积分里连续的卷积);常常需要用一个<font color=#0099ff>滤波器</font>(filter)做空间滤波操作;滤波器也有很多名字:<font color=#0099ff>卷积模版(template)</font>、<font color=#0099ff>卷积核(kernel)</font>、<font color=#0099ff>掩模(mask)</font>、<font color=#0099ff>窗口(window)</font>等。

&emsp;&emsp;空间滤波可以分为<font color=#0099ff>线性滤波(linear filter)</font>和<font color=#0099ff>非线性滤波(non-linear filter)</font>。非线性滤波常见的有中值滤波、最大值滤波等，相当于自定义一个函数，在数学上由于不满足线性变换因此叫做非线性滤波。这里不细研究它。

&emsp;&emsp;<font color=#0099ff>线性滤波</font>通常是：将模版覆盖区域内的元素，以模版中对应位置元素为权值，进行累加。与卷积操作类似的，还有一种数学操作叫做协相关(cross-correlatioin)，<u>它们都可以看作是基于矩阵内积的操作</u>。具有<font color=#0099ff>平移不变性(shift-invariant)</font>。

![Cross-Correlation](http://ou5rpeic9.bkt.clouddn.com/cross-correlation%E6%96%87%E5%AD%97%E8%A7%A3%E9%87%8A.png)

&emsp;&emsp;<font color=#0099ff>二者的区别：</font><u>卷积和协相关的差别是，卷积需要先对滤波矩阵进行<font color=#0099ff>180度的翻转</font>，但如果矩阵是对称的，那么两者就没有什么差别了</u>。实际上，在信号处理领域，卷积有广泛的意义，而且有其严格的数学定义，但在这里不关注这个。2D卷积需要4个嵌套循环(4-double loop)，所以它并不快，除非我们使用很小的卷积核。这里一般使用3x3或者5x5。而且，<u>对于滤波器，也有一定的规则要求</u>：

> 1）滤波器的大小应该是奇数，这样它才有一个中心，例如3x3，5x5或者7x7。有中心了，也有了半径的称呼，例如5x5大小的核的半径就是2。

> 2）滤波器矩阵所有的元素之和应该要等于1，这是为了保证滤波前后图像的亮度保持不变。当然了，这不是硬性要求了。

> 3）如果滤波器矩阵所有元素之和大于1，那么滤波后的图像就会比原图像更亮，反之，如果小于1，那么得到的图像就会变暗。如果和为0，图像不会变黑，但也会非常暗。

> 4）对于滤波后的结构，可能会出现负数或者大于255的数值。对这种情况，我们将他们直接截断到0和255之间即可。对于负数，也可以取绝对值。

### 卷积核(Convolution Kernel)
&emsp;&emsp;定义不同的卷积核与原图像做卷积操作，可以产生不同的效果。滤波与模糊的概念区分。模糊是人为感受到的现象，通过滤波操作可以呈现。
#### 什么都不做
&emsp;&emsp;滤波器矩阵所有元素之和为1，保证滤波前后图像的亮度保持不变。这个滤波器啥也没有做，得到的图像和原图是一样的。因为只有中心点的值是1。邻域点的权值都是0，对滤波后的取值没有任何影响。

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2014.50.25.png)
#### 图像锐化滤波器(Sharpness Filter)
&emsp;&emsp;图像的锐化和边缘检测很像，首先找到边缘，然后把边缘加到原来的图像上面，这样就强化了图像的边缘，使图像看起来更加锐利了。这两者操作统一起来就是锐化滤波器了，<u>也就是在边缘检测滤波器的基础上，再在中心的位置加1，这样滤波后的图像就会和原始的图像具有同样的亮度了，但是会更加锐利。</u>

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2014.50.44.png)

&emsp;&emsp;我们把核加大，就可以得到更加精细的锐化效果

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2014.50.52.png)

&emsp;&emsp;下面的滤波器会更强调边缘。

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2014.56.05.png)

&emsp;&emsp;主要是<u>强调图像的细节</u>。最简单的3x3的锐化滤波器如下：

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2014.53.58.png)

&emsp;&emsp;实际上是计算当前点和周围点的差别，然后将这个差别加到原来的位置上。另外，<u>中间点的权值要比所有的权值和大于1，意味着这个像素要保持原来的值。</u>

#### 边缘检测(Edge Detection)

&emsp;&emsp;我们要找水平的边缘：<u>需要注意的是，这里矩阵的元素和是0，所以滤波后的图像会很暗</u>，只有边缘的地方是有亮度的。

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2014.59.54.png)

&emsp;&emsp;相当于做差分(differencing)，求导的离散版本，有向前差分(forward differencing)，向后差分(backward differencing)，中心差分(central differencing)。

&emsp;&emsp;下面的滤波器可以找到垂直方向的边缘，这里像素上和下的像素值都使用

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.08.09.png)

&emsp;&emsp;再下面这个滤波器可以找到45度的边缘：取-2不为了什么，只是为了让矩阵的元素和为0而已。

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.10.30.png)

&emsp;&emsp;那下面这个滤波器就可以检测所有方向的边缘。

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.17.38.png)

&emsp;&emsp;为了检测边缘，我们需要在图像对应的方向计算梯度。用下面的卷积核来卷积图像，就可以了。但在实际中，这种简单的方法<u>会把噪声也放大</u>了。另外，需要注意的是，<u>矩阵所有的值加起来要是0</u>。

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.21.07.png)

#### 浮雕(Embossing Filter)

&emsp;&emsp;浮雕滤波器可以给图像一种3D阴影的效果。只要将中心一边的像素减去另一边的像素就可以了。这时候，像素值有可能是负数，我们将负数当成阴影，将正数当成光，然后我们<u>对结果图像加上128的偏移</u>。这时候，图像大部分就变成灰色了。

&emsp;&emsp;下面是45度的浮雕滤波器

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.25.28.png)

&emsp;&emsp;只要加大滤波器，就可以得到更加夸张的效果了

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.27.12.png)

#### 均值模糊(Box Filter/Averaging)

&emsp;&emsp;我们可以将当前像素和它的四邻域的像素一起取平均，然后再除以5，或者直接在滤波器的5个地方取0.2的值即可，如下图

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.29.23.png)

&emsp;&emsp;可以看到，这个模糊还是比较温柔的，我们可以把滤波器变大，这样就会变得粗暴了：<u>注意要将和再除以13.</u>

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.33.28.png)

&emsp;&emsp;如果你想要更模糊的效果，加大滤波器的大小即可。或者对图像应用<u>多次模糊</u>也可以。

#### 高斯模糊(Gaussian Filter)

&emsp;&emsp;<u>均值模糊很简单，但不是很平滑。高斯模糊就有这个优点，所以被广泛用在图像降噪上。特别是在边缘检测之前，都会用来移除细节。高斯滤波器是一个低通滤波器。</u>

#### 运动模糊(Motion Blur)

&emsp;&emsp;运动模糊可以通过只在一个方向模糊达到，例如下面9x9的运动模糊滤波器。注意，<u>求和结果要除以9。</u>

&emsp;&emsp;这个效果就好像，摄像机是从左上角移动的右下角。

![](http://ou5rpeic9.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-08-13%2015.41.32.png)

### 自己手写的卷积函数
虽然Matlab已经封装了一个函数con2()作卷积操作，但是还是要自己手写一下加深理解。下面是[UCF计算机视觉网课](http://crcv.ucf.edu/courses/CAP5415/Fall2012/)的作业。4层循环。因为<u>边缘效应</u>通常情况下需要补0。还是Matlab新手，代码一点都不elegant，没有借鉴意义。
```matlab
    function [] = pa1()
    m_img = imread('PA1/balloon.jpg');
    m_kernel = rand(5,5);
    [r1, c1, cc1] = size(m_img);
    [r2, c2] = size(m_kernel);
    for i = r2+1:r1
        for j = c2+1:c1
            sump = 0;
            for k = 1:r2
                for l = 1:c2
                    sump = m_img(i-k, j-l) * m_kernel(k,l);
                end;
            end;
            outQ(i,j) = sump;
        end;
    end
    imshow(outQ);wait(0);
```
### Matlab中用imfilter()实现线性空间滤波
&emsp;&emsp;最常用的是这句：
```matlab
    imfilter(f, w, 'replicate') %相关滤波，默认边界外围填充0，造成黑边，使用'replicate'复制边缘像素，消除边缘效应，输出大小与输入图像f相同.
```
&emsp;&emsp;imfilter()默认是相关算子，在做真正的卷积之前可以先将核旋转180°(除非你指定了滤波模式为'conv')，rotated_filter=flipud(fliplr(filter))
```
    imfilter(f, w, filtering_mode, boundary_options, size_options)
    f:图像
    w:滤波模版
    filtering_mode:滤波模式
        'corr':相关滤波。[默认值]
        'conv':卷积滤波。
    boundary_options:边界选项
        P:(没有引号)边界外围补充0。[默认值]
        'replicate':边界外围复制边界值
        'symmetric':边界外围使用边界镜像
        'circular':图像的大小通过讲图像处理为二维周期函数的一个周期来扩展（这是什么？）
    size_options:大小选项
        'same':输出大小与输入图像f大小相同。[默认值]
        'full':输出与扩展（填充）后的图像大小相同。
```

### 总结
1. 规范用语，在<font color=#0099ff>滑窗操作</font>、<font color=#0099ff>计算图像梯度</font>等场合，不要使用“卷积”，而要使用“滤波”或者“协相关”。因为，我们通常讲的卷积，在这几个操作中其实是协相关，那就不要用卷积这个词以<u>避免引起混淆</u>。

2. 不同的滤波器产生的效果不同，滤波器的定义有讲究。经常会看到一些人名命名的算子。比如用作边缘检测，拉普拉斯(Laplacian)算子、Canny算子。观察它们的算子内容，结合卷积核的定义规则。你会发现其实就那么回事。不要被复杂的学术名词给弄懵逼了。

3. GPU实现代替CPU实现，CPU实现需要四重循环，时间代价太大，性能不好。GPU使用特定的数据结构直接进行计算降低时间代价。

4. 这是我的第一篇博文，本来是想好好写的，发现写博文真的是一项技术活。话到嘴边，憋不出一个字。

### 参考引用
1. [图像卷积与滤波的一些知识点](http://blog.csdn.net/zouxy09/article/details/49080029)

2. [图像卷积、相关以及在MATLAB中的操作](http://www.cnblogs.com/zjutzz/p/5661543.html)