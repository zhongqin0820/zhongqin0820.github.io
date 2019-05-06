---
title: Go实现10大经典排序算法
date: 2019-03-08 17:24:10
updated: 2019-05-05 15:38:10
categories:
- 计算机基础

tags:
- Go
- 经典算法
---
# 前言
使用Go语言实现常见10大经典排序算法。包括：冒泡、插入、选择、归并、快排...。

![不同排序算法时间复杂度对比图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/algo_sorting_time_comparision.png)


<!-- more -->
# 排序
目的：使无序的序列有序。在分析时间复杂度时，使用**平均时间复杂度**。
基于此，会有对应算法特点的应用。面试时会侧重"大"数据的考察：如，对海量文件进行排序（归并排序）、海量数据的Top K个元素解（堆排序，或利用快排的partition部分）...

## 比较排序
### 冒泡排序：$ O(n^2) $
可以将序列分为两个部分：无序|有序。按照某种约定，可以将大的放到后面，也可以将小的放到后面。但是一旦一轮之后，后面有序的部分都已经是最值了。**因此，对于有序的部分可以不需要再进行比较**。[动画解释](https://www.bilibili.com/video/av41042841/?p=7)

```go
// 冒泡排序
func BubleSort(a []int, flag bool) {
    if Defensive(a) {
        return a
    }
    n := len(a)
    for i:=0;i<n;i++{
        for j:=1;j<n-i;j++{
            if flag {//small -> big
                if a[j-1]>a[j] {
                    a[j],a[j-1]=a[j-1],a[j]
                }
            } else {//big->small
                if a[j-1]<a[j] {
                    a[j],a[j-1]=a[j-1],a[j]
                }
            }
        }
    }
    return a
}

// 边界判断
func Defensive(a []int) bool {
    if a==nil || len(a) == 0 || len(a) == 1 {
        return False
    }
    return True
}

```

### 选择排序：$ O(n^2) $
序列被分为两个部分：有序|无序。排序的过程即在无序的序列中找出最值放在有序部分末尾。类似，玩扑克时抓完所有牌（未排序）但是我们已经知道所有的牌，可以挑出自己想要的牌（无序中的最值），进行对牌的归档的过程。但是，在实现的过程中，我们可以考虑无序部分的第一个是key,在后续无序部分如果找到满足值大于（小于）它的，就交换这两个值。一轮之后，key就变为无序部分的最值了。[动画解释](https://www.bilibili.com/video/av41042841/?p=10)

```go
func SelectSort(a []int, flag bool) {
    if Defensive(a) {
        return a
    }
    n := len(a)
    for i:=0;i<n;i++ {
        for j:=i+1;j<n;j++ {
            if flag {// small -> big
                if a[i] > a[j] {
                    a[i],a[j]=a[j],a[i]
                }
            } else {// big -> small
                if a[i] < a[j] {
                    a[i],a[j]=a[j],a[i]
                }
            }
        }
    }
    return a
}
```

### 插入排序：$ O(n^2) $
序列被分为两个部分：有序|无序。排序的过程即总是选择无序的第一个元素插入有序部分的合适位置。类似，玩扑克时，边抓牌（无序第一张），边往手上排好序的牌中插入的过程。[动画解释](https://www.bilibili.com/video/av41042841/?p=1)

```go
func InsertSort( a []int, flag bool) {
    if Defensive(a) {
        return a
    }
    n := len(a)
    for i:=0;i<n;i++ {
        if flag {//small -> big
            for j:=i+1;j<n;j++ {
                if
            }
        }
    }
}

```

### 希尔排序：$O \left ( O^{1.3} \right )$

[动画解释](https://www.bilibili.com/video/av41042841/?p=9)

## $ O\left ( n log \left ( n \right ) \right ) $ 简单推导
这里的说明主要参考[这个视频教程](https://www.youtube.com/watch?v=4Q72kbwyEmk)。
对于对数关系，有：
$$ \because log_c a + log_c b = log_c \left ( {a \times b} \right ) $$

针对每个带排序元素，插入的时间复杂度有如下关系：
$$ \therefore log_c n + log_c \left ( {n-1}\right ) + \cdots + log_c \left( 1 \right ) = log_c \left ( n! \right ) $$

根据最优化原理(更多细节，可以查看这个问题[回答](https://math.stackexchange.com/questions/140961/why-is-logn-on-log-n))，又：

$$ \because n! \approx 2^{ n log \left ( n \right ) } $$

令$ c=2 $，则有：

$$ \therefore log_2 \left ( n! \right ) \approx n log \left ( n \right ) $$

即我们的时间复杂度：$ O\left ( n log \left ( n \right ) \right ) $

### 归并排序：$ O\left ( n log \left ( n \right ) \right ) $
利用分治（Divided-Conquer）的思想进行排序。[动画解释](https://www.bilibili.com/video/av41042841/?p=3)

### 快速排序：$ O\left ( n log \left ( n \right ) \right ) $
利用i，pivot，j之间的关系进行快速排序。[动画解释](https://www.bilibili.com/video/av41042841/?p=6)

### 堆排序：$ O\left ( n log \left ( n \right ) \right ) $
利用堆结构进行排序。调整堆的复杂度为$ n log n $，其中前面的$ n $为外部元素个数，后面的$ n $为堆的大小。[动画解释](https://www.bilibili.com/video/av41042841/?p=2)

## 非比较排序
以上排序算法属于比较类排序算法，即通过比较来决定元素间的相对次序，由于其时间复杂度不能突破$ O\left ( n log \left ( n \right ) \right ) $，因此也称为非线性时间比较类排序。

### 桶排序：$O \left ( n+k \right )$
[动画解释](https://www.bilibili.com/video/av41042841/?p=8)

### 计数排序：$O \left ( n+k \right )$
[动画解释](https://www.bilibili.com/video/av41042841/?p=5)

### 基数排序：$O \left ( n \times k \right )$
[动画解释](https://www.bilibili.com/video/av41042841/?p=4)


## 其它排序

### TimSort


### TreeSort


### CubeSort


# 结束语


# 参考资料
- [十大经典排序算法（动图演示）](https://www.cnblogs.com/onepixel/p/7674659.html)
- [bigocheatsheet](http://bigocheatsheet.com/)
