---
title: Go实现经典排序与查找算法
date: 2019-03-08 17:24:10
updated: 2019-03-08 17:24:10
categories:
- 网页开发

tags:
- Go
---
# 前言
使用Go语言实现常见排序与搜索算法。包括：冒泡、插入、选择、归并、快排以及顺序查找与二分查找。

<!-- more -->
# 排序
目的：使无序的序列有序。
基于此，会有对应算法特点的应用。如，对海量文件进行排序（归并排序）、海量数据的Top K个元素解（堆排序，或利用快排的partition部分）...

## $O(n^2)$
### 冒泡排序
可以将序列分为两个部分：无序|有序。按照某种约定，可以将大的放到后面，也可以将小的放到后面。但是一旦一轮之后，后面有序的部分都已经是最值了。**因此，对于有序的部分可以不需要再进行比较**。
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
### 选择排序
序列被分为两个部分：有序|无序。排序的过程即在无序的序列中找出最值放在有序部分末尾。类似，玩扑克时抓完所有牌（未排序）但是我们已经知道所有的牌，可以挑出自己想要的牌（无序中的最值），进行对牌的归档的过程。但是，在实现的过程中，我们可以考虑无序部分的第一个是key,在后续无序部分如果找到满足值大于（小于）它的，就交换这两个值。一轮之后，key就变为无序部分的最值了。
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

### 插入排序
序列被分为两个部分：有序|无序。排序的过程即总是选择无序的第一个元素插入有序部分的合适位置。类似，玩扑克时，边抓牌（无序第一张），边往手上排好序的牌中插入的过程。
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

## $O(nlogn)$

# TODO 需要分析为什么，时间复杂度是这个

### 归并排序

### 快速排序

### 希尔排序


### 堆排序



# 搜索
目的：在有序的序列內查找目标值对应下标索引，找不到时返回-1。
## $O(1)$

## $O(n)$

## $O(logn)$

# 结束语


# 参考资料
- [排序算法]()
- [搜索算法]()
- [时间复杂度分析：大O法]()
