---
title: Go实现经典算法问题：查找
date: 2019-03-08 17:26:11
updated: 2019-06-16 19:45:16
categories:
- 计算机基础

tags:
- 经典算法
- Go
---
# 前言
使用Go语言实现常见查找算法。包括：顺序查找，插值查找（二分查找、插值查找、斐波那契查找），树表查找（二叉树、红黑树，B树，B+树）以及哈希查找。
完整代码参阅[Github repo](https://github.com/zhongqin0820/coding-playground/tree/master/go/test/sorts)。
<div style="width: 300px; margin: auto">

![时间复杂度对比图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/big_O_cheat.png)
</div>

<!-- more -->
# 查找/搜索
目的：通常要求在有序的序列內查找目标值对应下标索引，找不到时通常返回`-1`。

## 线性时间（$ O \left ( n \right ) $）
含义：该算法完成任务所需要的步骤直接和n相关。
这是查找算法在未优化情况下的时间复杂度，典型的查找算法是顺序查找。在对其进行优化时，可以**通过使用特殊的数据结构**，以空间换时间，或者通过序列本身的特点优化算法的查找过程，从而降低时间复杂度。后续我们将进一步展开介绍，下面先给出关于顺序查找的说明。

### 顺序查找
数据可以是无序的，依次遍历元素。

```go
// SeqSearch 返回目标值在序列中的下标索引，找不到返回-1
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

### 改进：分块查找
将n个数据元素"按块有序"划分为m块（m ≤ n）。每一块中的结点不必有序，但块与块之间必须"按块有序"；即：第1块中任一元素的关键字都必须小于第2块中任一元素的关键字；而第2块中任一元素又都必须小于第3块中的任一元素，以此类推。

## 对数时间（$ O \left ( log \left ( n \right ) \right ) $）
含义：该算法每执行一步，它完成任务所需要的步骤数目会以一定的因子减少。

### 插值查找
#### 二分查找
数据需要已经排好序。计算中间索引下标时，应使用`mid = i + (j-i)/2`，防止溢出。

```go
func BinarySearch(a []int, target int) int {
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

#### 插值查找/自适应二分查找
基于二分查找的改进，使用自适应的比例参数代替`1/2`，即：`mid = i + (j-i) * (target-a[i]) / (a[j]-a[i]) `，但是需要注意的是，这种方法如果查找的元素为负且不在列表中会出错。需要对其进行边界判断。

```go
func InterpolationSearch(a []int, target int) int {
    i, mid, j, n := 0, 0, len(a)-1, len(a)-1
    for i <= j {
        mid = i + (j-i)*(target-a[i])/(a[j]-a[i])
        if mid < 0 || mid > n {
            break
        }
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

#### 斐波那契查找
基于二分查找，利用斐波那契数列。
- 需要预先生成斐波那契数列，通过运用黄金比例的概念在数列中选择查找点进行查找，提高查找效率。同样地，斐波那契查找也属于一种有序查找算法。
- 要求开始表中记录的个数为某个斐波那契数小1，即：$ n=F \left ( k \right ) - 1 $;
    - 开始将k值与第F(k-1)位置的记录进行比较(及mid=low+F(k-1)-1),比较结果也分为三种
    - 1）相等，mid位置的元素即为所求
    - 2）大于，low=mid+1,k-=2;
    - 3）小于，high=mid-1,k-=1。

```go
func FibSearch(a []int, target int) int {
    var n int = len(a)
    // Auto generate Fibonacci seq & Find the first k meet n<F[k]-1
    F, k := func(n int) ([]int, int) {
        var dp []int = []int{}
        dp = append(dp, 1) //dp[0]=1
        dp = append(dp, 1) //dp[1]=1
        var i int = 2
        for ; dp[i-1]-1 < n; i++ {
            dp = append(dp, dp[i-1]+dp[i-2]) //dp[i]=dp[i-1]+dp[i-2]
        }
        return dp, i - 1
    }(n)
    // Extend the original a to meet up F[k]-1
    for i := n; i < F[k]-1; i++ {
        a = append(a, a[i-1])
    }
    // do the Search
    var i, j, mid int = 0, len(a) - 1, 0
    for i <= j {
        mid = i + F[k-1] - 1
        switch {
        case target == a[mid]:
            if mid < n {
                return mid
            } else {
                return n - 1
            }
        case target < a[mid]:
            j = mid - 1
            k -= 1
        default: //target > a[mid]
            i = mid + 1
            k -= 2
        }
    }
    return -1
}
```

### 树表查找
对**待查找的数据**进行生成二叉查找树，利用该树的性质，通过**中序遍历**得到有序序列。

#### 二叉查找树
- 参考资料参阅[浅谈算法和数据结构: 七 二叉查找树](https://www.cnblogs.com/yangecnu/p/Introduce-Binary-Search-Tree.html)
- 不要求平衡，最坏情况下的时间复杂度退化为：$ O \left ( n \right ) $
- 基于此的**改进是要求二叉查找树在进行对应增删操作后保持平衡**。
- 平衡查找树：2-3树
- 2-3查找树的扩展：
    - 红黑树
        - 红黑树的思想就是对2-3查找树进行编码，尤其是对2-3查找树中的3-nodes节点添加额外的信息。
        - 使用颜色标记来替代2-3树中比较难处理的3-node节点问题。
    - B树、B+树
        - 在文件系统和数据库系统中有着广泛的应用。

## 常数时间（$ O \left ( 1 \right ) $）
- 含义：该算法在查找时通过提供的键值索引仅仅使用一步就可以完成查找任务。(对于无冲突的Hash表而言，查找复杂度为O(1)（注意，在查找之前我们需要构建相应的Hash表）。)
- 特点："直接定址"与"解决冲突"
- 哈希函数：通过某种转换关系，使关键字适度的分散到指定大小的的顺序结构中，越分散，则以后查找的时间复杂度越小，空间复杂度越高。
- 算法流出
    - 用给定的哈希函数构造哈希表；
    - 根据选择的冲突处理方法解决地址冲突；参阅[浅谈算法和数据结构: 十一 哈希表](https://www.cnblogs.com/yangecnu/p/Introduce-Hashtable.html)
        - 拉链法（separate chaining）
        - 线性探测（linear probing）
    - 在哈希表的基础上执行哈希查找。

### hashmap
利用数据结构的索引特性，`target`做`key`，映射到对应的`value`（即下标：`index`）。需要额外的预处理时间。

```go
func HashSearch(a []int, target int) int {
    m := make(map[int][]int, len(a))
    // 预处理
    for i, v := range a {
        m[v] = append(m[v], i)
    }
    // 索引查找
    i, ok := m[target]
    if ok {
        return i[0]
    }
    return -1
}
```

#### 哈希表基础理论
- [理解 Golang 哈希表 Map 的原理](https://draveness.me/golang-hashmap)：推荐阅读

哈希表名称的来源是因为它使用了哈希函数将一个键映射到一个桶中，这个桶中就可能包含该键对应的值。
在理想情况下，哈希函数应该能够将不同键映射到唯一的索引上，但是**键集合的数量会远远大于映射的范围**，不同的键经过哈希函数的处理应该会返回唯一的值，这要求哈希函数的输出范围大于输入范围，所以在实际使用时，这个理想状态是不可能实现的。
比较实际的方式是让哈希函数的结果能够**均匀分布**，这样虽然哈希会发生**碰撞**和**冲突**，但是我们只需要解决哈希碰撞的问题就可以在工程上去使用这种更实际的方案，结果不均匀的哈希函数会造成更多的冲并带来更差的性能。
在一个使用结果较为均匀的哈希函数中，哈希的增删改查都需要 O(1) 的时间复杂度，但是非常不均匀的哈希函数会导致所有的操作都会占用最差 O(n) 的复杂度，所以**在哈希表中使用好的哈希函数是至关重要的。**

##### 冲突解决
- **开放寻址法**是一种在哈希表中解决哈希碰撞的方法，这种方法的核心思想是在一维数组对元素进行探测和比较以判断待查找的目标键是否存在于当前的哈希表中，如果我们使用开放寻址法来实现哈希表，那么在初始化哈希表时就会创建一个新的数组，如果当我们向当前『哈希表』写入新的数据时发生了冲突，就会将键值对写入到下一个不为空的位置。当我们**查找某个键对应的数据时**，就会对哈希命中的内存进行**线性探测**，找到目标键值对或者空内存就意味着这一次查询操作的结束。开放寻址法中对性能影响最大的就是装载因子，它是数组中元素的数量与数组大小的比值，随着装载因子的增加，线性探测的平均用时就会逐渐增加，这会同时影响哈希表的查找和插入性能，当装载率超过 70% 之后，哈希表的性能就会急剧下降，而一旦装载率达到 100%，整个哈希表就会完全失效，所以在实现哈希表时一定要时刻关注装载因子的变化。

- **拉链法**是哈希表中最常见的实现方法，大多数的编程语言都是用拉链法实现哈希表，它的实现相对比较简单，平均查找的长度也比较短，而且各个用于存储节点的内存都是动态申请的，可以节省比较多的存储空间。拉链法的实现其实就是使用数组加上链表组合起来实现哈希表，数组中的每一个元素其实都是一个链表，我们可以将它看成一个可以扩展的二维数组。当我们需要将一个键值对**加入一个哈希表时**，键值对中的键都会先经过一个哈希函数，哈希函数返回的哈希会帮助我们『选择』一个桶，然后遍历当前桶中持有的链表，找到键相同的键值对执行更新操作或者遍历到链表的末尾追加一个新的键值对。如果要通过某个键在哈希表中获取对应的值，就会经历如下的过程：Key11 就是展示了一个键在哈希表中不存在的例子，当哈希表发现它命中 2 号桶时，它会依次遍历桶中的链表，解决遍历到链表的末尾也没有找到期望的键，所以该键对应的值就是空。**每一次读写哈希表的时间基本上都花在了定位桶和遍历链表这两个过程**。

# 结束语
如果问题要求找极值，那么还可以利用其他特殊的数据结构。

> 遇到算法题，首先考虑应该使用何种ADT，再接着考虑编码问题以及最终的结果。

# 参考资料
- [[Data Structure & Algorithm] 七大查找算法](https://www.cnblogs.com/maybe2030/p/4715035.html)
- [bigocheatsheet](http://bigocheatsheet.com/)
- [BST](https://www.cnblogs.com/yangecnu/p/Introduce-Binary-Search-Tree.html)
- [Hashtable](https://www.cnblogs.com/yangecnu/p/Introduce-Hashtable.html)

# TODO
- [ ] 2019/06/16: 同时附加对面试中海量数据问题的解题思路的学习总结。

# 面试中的海量数据问题
考虑使用bitmap数据结构解决，如何从对问题的分析，到抽象数据结构的设计，复杂度的分析过程细节。
- [ ] [BITMAP 多语言实现及应用](http://blog.studygolang.com/2014/09/bitmap_multi_language/)
- [ ] [Go 小知识之 Go 中如何使用 set](https://juejin.im/post/5ceffeddf265da1bc8540df5)
- [ ] [Go 之 SortedMap 与 LinkedHashMap](https://windmt.com/2019/03/26/golang-sortedmap-linkedhashmap/)