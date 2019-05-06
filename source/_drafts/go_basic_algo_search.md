---
title: Go实现7大经典查找算法
date: 2019-03-08 17:26:11
updated: 2019-03-08 17:26:11
categories:
- 计算机基础

tags:
- Go
- 经典算法
---
# 前言
使用Go语言实现常见查找算法。包括：顺序查找与二分查找...。
![时间复杂度对比图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/big_O_cheat.png)

<!-- more -->
# 搜索
目的：通常要求在有序的序列內查找目标值对应下标索引，找不到时返回`-1`。

## 线性时间（$ O \left ( n \right ) $）
含义：该算法完成任务所需要的步骤直接和n相关。
这是查找算法在未优化情况下的时间复杂度，典型的查找算法是顺序查找。在对其进行优化时，可以**通过使用特殊的数据结构**，以空间换时间，或者通过序列本身的特点优化算法的查找过程，从而降低时间复杂度。后续我们将进一步展开介绍，下面先给出关于顺序查找的说明。

### 顺序查找
可以要求数据是无序的，依次遍历。

```go
// SeqSearch 返回目标值在序列中的下标索引
// a 是查找序列，target 是查找目标
func SeqSearch(a []int, target int) int {
    for i, v := range a {
        if v == target {
            return i
        }
    }
    return -1
}
```

## 对数时间（$ O \left ( log \left ( n \right ) \right ) $）
含义：该算法每执行一步，它完成任务所需要的步骤数目会以一定的因子减少。

### 二分查找
需要利用数据已经排好序的特点。

```go
func BinSearch(a []int, target int) int {
    i, mid, j := 0, 0, len(a)-1
    for i<=j {
        mid = i + (j - i)/2
        if a[mid] == target {
            return mid
        } else if a[mid] < target {
            i = mid + 1
        } else {
            j = mid - 1
        }
    }
    return -1
}
```

## 常数时间（$ O \left ( 1 \right ) $）
含义：该算法仅仅使用一步就可以完成任务。

### hashmap
利用数据结构的索引特性，`target`做`key`，引射到对应的`value`（即下标：`index`）。需要额外的预处理时间。

```go
func HashSearch(a []int, target int) int {
    m := make(map[int][]int, len(a))
    // 预处理
    for i,v := a {
        m[v]=append(m[v], v)
    }
    // 索引查找
    i, ok := m[target]
    if ok {
        return i[0]
    }
    return -1
}
```

# 结束语
如果问题要求找极值，那么还可以利用其他特殊的数据结构。
> 遇到算法题，首先考虑应该使用何种ADT，再接着考虑编码问题以及最终的结果。

# 参考资料
- [[Data Structure & Algorithm] 七大查找算法](https://www.cnblogs.com/maybe2030/p/4715035.html)
- [bigocheatsheet](http://bigocheatsheet.com/)
