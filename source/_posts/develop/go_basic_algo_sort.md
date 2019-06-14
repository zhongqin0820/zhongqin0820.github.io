---
title: Go实现经典排序算法
date: 2019-03-08 17:24:10
updated: 2019-06-14 18:25:51
categories:
- 计算机基础

tags:
- Go
- 经典算法
---
# 前言
使用Go语言实现常见经典排序算法以及总结Go中内置的排序算法实现。完整代码参见[Github repo](https://github.com/zhongqin0820/coding-playground/tree/master/go/test/sorts)。
实现包括常见的比较类排序算法：冒泡、选择、插入、希尔、归并、快排、堆排序。以及较为非比较类排序算法：计数排序、基数排序、桶排序算法。以及其它排序算法：TreeSort、TimSort、CubeSort。

![不同排序算法时间复杂度对比图](https://raw.githubusercontent.com/zhongqin0820/zhongqin0820.github.io/source-articles/source/images/algo_sorting_time_comparision.png)

<!-- more -->
# 排序
排序算法的目的是**使无序的序列有序**。在分析时间复杂度时，使用**平均时间复杂度**（即大$O$法）。

> 面试时会侧重"大"数据的考察：如，对海量文件进行排序（归并排序）、海量数据的Top K个元素解（堆排序，或利用快排的partition部分的特性）。

## 比较排序

```go
// 边界判断代码
func Defensive(a []int) bool {
    if a==nil || len(a) == 0 || len(a) == 1 {
        return true
    }
    return false
}
```

### 冒泡排序：$ O(n^2) $
可以将序列分为两个部分：无序|有序。按照某种约定，可以将大的放到后面，也可以将小的放到后面。但是一旦一轮之后，后面有序的部分都已经是最值了。**因此，对于有序的部分可以不需要再进行比较**。[动画解释](https://www.bilibili.com/video/av41042841/?p=7)

```go
// 冒泡排序
func BubleSort(s []int, flag bool) {
    if Defensive(s) {
        return
    }
    for i, _ := range s {
        for j := 0; j < len(s)-i-1; j++ {
            if flag && s[j] < s[j+1] {
                s[j], s[j+1] = s[j+1], s[j]
            } else if !flag && s[j] > s[j+1] {
                s[j], s[j+1] = s[j+1], s[j]
            }
        }
    }
}

```

### 选择排序：$ O(n^2) $
序列被分为两个部分：有序|无序。排序的过程即在无序的序列中找出最值放在有序部分末尾。类似，玩扑克时抓完所有牌（未排序）但是我们已经知道所有的牌，可以挑出自己想要的牌（无序中的最值），进行对牌的归档的过程。但是，在实现的过程中，我们可以考虑无序部分的第一个是key,在后续无序部分如果找到满足值大于（小于）它的，就交换这两个值。一轮之后，key就变为无序部分的最值了。[动画解释](https://www.bilibili.com/video/av41042841/?p=10)

```go
func SelectSort(a []int, flag bool) {
    if Defensive(s) {
        return
    }
    var temp int
    for i := 0; i < len(s)-1; i++ {
        temp = i
        if !flag {
            for j := i + 1; j < len(s); j++ {
                if s[j] < s[temp] {
                    temp = j
                }
            }
            s[i], s[temp] = s[temp], s[i]
        } else {
            for j := i + 1; j < len(s); j++ {
                if s[j] > s[temp] {
                    temp = j
                }
            }
            s[i], s[temp] = s[temp], s[i]
        }
    }
}
```

> 个人区分选择与插入的办法仅供参考：选择最值，插入有序。

### 插入排序：$ O(n^2) $
序列被分为两个部分：有序|无序。排序的过程即总是选择无序的第一个元素插入有序部分的合适位置。类似，玩扑克时，边抓牌（无序第一张），边往手上排好序的牌中插入的过程。[动画解释](https://www.bilibili.com/video/av41042841/?p=1)

```go
func InsertSort( a []int, flag bool) {
    if Defensive(a) {
        return
    }
    n := len(a)
    for i := 1; i < n; i++ {
        if flag && a[i] < a[i-1] { // small -> big
            for j := i; j > 0 && a[j] < a[j-1]; j-- {
                a[j], a[j-1] = a[j-1], a[j]
            }
        } else if !flag && a[i] > a[i-1] { // big -> small
            for j := i; j > 0 && a[j] > a[j-1]; j-- {
                a[j], a[j-1] = a[j-1], a[j]
            }
        }
    }
}
```

#### 希尔排序：$O \left ( n log \left ( n \right ) ^2  \right )$
是对简单插入排序的一种改进。[动画解释](https://www.bilibili.com/video/av41042841/?p=9)。
```go
func ShellSort(a []int, flag bool) {
    if Defensive(a) {
        return
    }
    n := len(a)
    // split the original array into gap groups
    for gap := n / 2; gap > 0; gap /= 2 {
        for i := gap; i < n; i++ {
            if !flag && a[i] < a[i-gap] { // small -> big
                for j := i; j-gap >= 0 && a[j] < a[j-gap]; j -= gap {
                    a[j], a[j-gap] = a[j-gap], a[j]
                }
            } else if flag && a[i] > a[i-gap] { // big-> small
                for j := i; j-gap >= 0 && a[j] > a[j-gap]; j -= gap {
                    a[j], a[j-gap] = a[j-gap], a[j]
                }
            }
        }
    }
}
```

## $ O\left ( n log \left ( n \right ) \right ) $ 简单推导
这里的说明主要参考这个[视频教程](https://www.youtube.com/watch?v=4Q72kbwyEmk)。
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
```go
func MergeSort(s []int, flag bool) []int {
    if Defensive(s) {
        return s
    }
    n := len(s)
    m := n / 2
    a := MergeSort(s[:m], flag)
    b := MergeSort(s[m:], flag)
    return Merge(a, b, flag)
}

func Merge(a, b []int, flag bool) (ret []int) {
    var i, j int
    if !flag { //small -> big
        for i, j = 0, 0; i < len(a) && j < len(b); {
            if a[i] < b[j] {
                ret = append(ret, a[i])
                i++
            } else {
                ret = append(ret, b[j])
                j++
            }
        }
    } else { //big -> small
        for i, j = 0, 0; i < len(a) && j < len(b); {
            if a[i] > b[j] {
                ret = append(ret, a[i])
                i++
            } else {
                ret = append(ret, b[j])
                j++
            }
        }
    }
    ret = append(ret, a[i:]...)
    ret = append(ret, b[j:]...)
    return ret
}
```

### 快速排序：$ O\left ( n log \left ( n \right ) \right ) $
利用i，pivot，j之间的关系进行快速排序。[动画解释](https://www.bilibili.com/video/av41042841/?p=6)
```go
func QuickSort(s []int, flag bool) {
    if Defensive(s) {
        return
    }
    // v is the value of pivot
    // k is the current index
    var i, j, v, k = 0, len(s) - 1, s[0], 1
    for i < j {
        if !flag { //small -> big
            if s[k] > v {
                s[k], s[j] = s[j], s[k]
                j--
            } else {
                s[i], s[k] = s[k], s[i]
                i++
                k++
            }
        } else { //big->small
            if s[k] < v {
                s[k], s[j] = s[j], s[k]
                j--
            } else {
                s[i], s[k] = s[k], s[i]
                i++
                k++
            }
        }
    }
    QuickSort(s[:i], flag)
    QuickSort(s[i+1:], flag)
}
```

### 堆排序：$ O\left ( n log \left ( n \right ) \right ) $
- 利用堆结构（近似完全二叉树）进行排序，利用了大根堆（或小根堆）堆顶记录的值最大（或最小）这一特征。
- 其中，构造堆的时间复杂度是$ O \left ( n \right ) $，每次调整堆的复杂度为$ log n $，共需要调整 $ n-1 $ 次；因此总的时间复杂为：$ O \left ( n log n \right ) $。
- [动画解释](https://www.bilibili.com/video/av41042841/?p=2)。其它参考资料[参阅](https://blog.csdn.net/m0_38132420/article/details/78611340)。

```go
// 调整堆的时间复杂度是 log n
func HeapAdjust(a []int, i, n int, flag bool) {
    var ichild int
    for ; 2*i+1 < n; i = ichild {
        ichild = 2*i + 1
        if !flag { //max heap: sort order small -> big
            if ichild < n-1 && a[ichild+1] > a[ichild] {
                ichild++
            }
            if a[i] < a[ichild] {
                a[i], a[ichild] = a[ichild], a[i]
            } else {
                break
            }
        } else { //min heap: sort order big -> small
            if ichild < n-1 && a[ichild+1] < a[ichild] {
                ichild++
            }
            if a[i] > a[ichild] {
                a[i], a[ichild] = a[ichild], a[i]
            } else {
                break
            }
        }

    }
}

func HeapSort(a []int, flag bool) {
    if Defensive(a) {
        return
    }
    n := len(a)
    // 构造堆
    for i := n / 2; i >= 0; i-- {
        // 调整堆，注意这里的范围
        HeapAdjust(a, i, n, flag)
    }
    // 堆排序：交换，调整剩下的元素
    for i := n - 1; i > 0; i-- {
        // 交换
        a[0], a[i] = a[i], a[0]
        // 调整剩下的元素，注意这里的范围
        HeapAdjust(a, 0, i, flag)
    }
}
```

### 比较排序小结
以上排序算法属于比较类排序算法，即通过比较来决定元素间的相对次序，由于其时间复杂度不能突破$ O\left ( n log \left ( n \right ) \right ) $，因此也称为非线性时间比较类排序。

## 非比较排序
### 计数排序：$O \left ( n+k \right )$
计数排序不是基于比较的排序算法，其核心在于**将输入的数据值转化为键存储在额外开辟的数组空间中**。 作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数。[动画解释](https://www.bilibili.com/video/av41042841/?p=5)。
- 当k不是很大并且序列比较集中时，计数排序是一个很有效的排序算法。

### 桶排序：$O \left ( n+k \right )$
[动画解释](https://www.bilibili.com/video/av41042841/?p=8)

### 基数排序：$O \left ( n \times k \right )$
[动画解释](https://www.bilibili.com/video/av41042841/?p=4)，其它参考资料[参阅](https://blog.csdn.net/m0_38132420/article/details/78627096)。

## 其它排序
### TreeSort
### TimSort
### CubeSort

# 使用Go中的排序算法
- sort：根据实际数据自动选择高效的排序算法（插入排序、归并排序、堆排序和快速排序）。
    - 对基本数据类型切片（[]int切片、[]float64切片和[]string切片）的排序支持
    - 判断基本数据类型切片是否已经排好序
    - 对排好序的数据集合逆序
    - 基本数据元素查找：**使用查找算法的前提是，数据已经排好序**

## 自定义数据类型排序
只要实现了sort.Interface定义的三个方法，就可以顺利对数据集合进行排序。
```go
// sort 包
type Interface interface {
    // 获取数据集合元素个数
    Len() int
    // 如果i索引的数据小于j索引的数据，返回true，且不会调用下面的Swap()，即数据升序排序。
    Less(i, j int) bool
    // 交换i和j索引的两个元素的位置
    Swap(i, j int)
}
// 实现了Interface接口的数据均可以
func Sort(data Interface)
// 判断基本数据类型切片是否已经排好序
func IsSorted(data Interface) bool
// 对排好序的数据集合逆序，使用实例：sort.Sort(sort.Reverse(inter Interface))
func Reverse(data Interface) Interface
// 使用“二分查找”算法（数据应该已经经过排序），来找出能使f(x)(0<=x<n)返回ture的最小值i
func Search(n int, f func(int) bool) int
```

## 内部数据类型排序
- IntSlice类型及`[]int`排序
    - `func Ints(a []int) { Sort(IntSlice(a)) }`
    - `func IntsAreSorted(a []int) bool`
    - `func SearchInts(a []int, x int) int`

- Float64Slice类型及`[]float64`排序
    - `func Float64s(a []float64)`
    - `func Float64sAreSorted(a []float64) bool`
    - `func SearchFloat64s(a []float64, x float64) int`

- StringSlice类型及`[]string`排序：两个string对象之间的大小比较是基于**“字典序”**的
    - `func Strings(a []string)`
    - `func StringsAreSorted(a []string) bool`
    - `func SearchStrings(a []string, x string) int`

## container/heap 包
> Package heap provides heap operations for any type that implements `heap.Interface`. 考虑到继承sort.Interface接口的三个方法，因此，定义一个堆需要实现五个方法。**属于最小堆**。

```go
type Interface interface {
    sort.Interface
    // Len() int
    // Less(i, j int) bool
    // Swap(i, j int)
    Push(x interface{}) // add x as element Len()
    Pop() interface{}   // remove and return element Len() - 1.
}
```

> 实现了`heap.Interface`的类型才可以调用下面的函数，注意函数的参数

```go
func Fix(h Interface, i int)//调整第i个节点元素，需要自己手动指定
func Init(h Interface)//初始化堆
func Pop(h Interface) interface{}//弹出堆
func Push(h Interface, x interface{})//压入堆
func Remove(h Interface, i int) interface{}//删除第i个节点元素
```

# 参考资料
- [十大经典排序算法（动图演示）](https://www.cnblogs.com/onepixel/p/7674659.html)
- [[译] 排序算法入门 — GO 语言实现](http://blog.studygolang.com/2017/07/sorting-algorithms-primer/)
- [bigocheatsheet](http://bigocheatsheet.com/)

# TODO
- [ ] 2019/06/14: 完成非比较排序部分
- [ ] 2019/06/14: 完成其它排序方法部分
- [ ] 2019/06/14: 面试中遇到的大数据问题总结分析
