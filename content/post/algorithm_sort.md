---
title: "「算法」经典排序算法的Go实现"
date: 2020-10-13T16:56:36+08:00
draft: true
toc: true
author: "彬Goh"
keywords: ["排序","go"]
description: "「算法」经典排序算法的Go实现"
tags: ["算法","go"]
categories: ["算法"]
mathjax: true
mathjaxEnableSingleDollar: true
---
在学习算法的过程中，会把写的代码放到下面的仓库中做记录，以下仓库都是`go`语言版本的。  
[链接：仓库地址](https://github.com/BinGoh98/Algorithm.git)
<!--more-->

## 插入排序

### 基本实现

**要点提示**：插入排序重复地将新的元素插入到一个排序好的子线性表中，直到整个线性表排好序。

```go
// insertSort
func insertSort(arr []int) []int {
    for i := 1; i < len(arr); i++ {
        curElement := arr[i]
        k := i - 1
        for ; k >= 0 && arr[k] > curElement; k-- {
            arr[k+1] = arr[k]
        }
        // 注意是 k+1
        arr[k+1] = curElement
    }
    return arr
}
```

### 参数

1. 复杂度

   平均：$O(n^2)$

   最好：$O(n)$

   最差：$O(n^2)$

2. 稳定性

   是稳定的！



## 冒泡排序

### 基本实现

**要点提示**：多次遍历数组，每次遍历中连续比较相邻的元素，如果元素没有按照顺序排列，则互换值。

```go
// bubble sort version 1
func bubbleSort1(arr []int) []int {
    swap := func(arr []int, i, j int) {
        tmp := arr[i]
        arr[i] = arr[j]
        arr[j] = tmp
    }
    for k := 0;k < len(arr); k++ {
        for i := 1;i < len(arr) - k;i++ {
            if arr[i-1] > arr[i] {
                swap(arr, i-1, i)
            }
        }
    }
    return arr
}
```

如果在某一次遍历中，没有发生交换，说明前面是排好序的，所以可以不用再排了。

```go
// bubble sort version 2
func bubbleSort2(arr []int) []int {
    swap := func(arr []int, i, j int) {
        tmp := arr[i]
        arr[i] = arr[j]
        arr[j] = tmp
    }
    swapped := true
    for k := 0; k < len(arr) && swapped; k++ {
        swapped = false
        for i := 1; i < len(arr)-k; i++ {
            if arr[i-1] > arr[i] {
                swap(arr, i-1, i)
                swapped = true
            }
        }
    }
    return arr
}
```

### 参数

1. 复杂度

   平均：$O(n^2)$

   最好：$O(n)$

   最差：$O(n^2)$

2. 稳定性

   是稳定的！

## 归并排序

### 基本实现

**要点提示**：将数组分成两半，每部分递归地应用归并排序，排完之后合并。

```go
// mergeSort
func mergeSort(arr []int) []int {
    // 1. 递归终止
    if len(arr) <= 1 {
        return arr
    }

    // 2. 递归
    prePart := mergeSort(arr[:len(arr)/2])
    postPart := mergeSort(arr[len(arr)/2:])

    // 3. 合并
    prePoint := 0
    postPoint := 0
    res := make([]int, 0, len(arr))
    for prePoint < len(prePart) && postPoint < len(postPart) {
        if prePart[prePoint] < postPart[postPoint] {
            res = append(res, prePart[prePoint])
            prePoint++
        }else{
            res = append(res, postPart[postPoint])
            postPoint++
        }
    }
    for prePoint < len(prePart) {
        res = append(res, prePart[prePoint])
        prePoint++
    }
    for postPoint < len(postPart) {
        res = append(res, postPart[postPoint])
        postPoint++
    }
    return res
}
```

### 参数

1. 复杂度

   平均：$O(n\log{n})$

2. 稳定性

   是稳定的！

### 拓展

[剑指offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

```go
var pair int
func reversePairs(nums []int) int {
    pair = 0
    help(nums)
    return pair

}

func help(nums []int) []int {
    if len(nums) <= 1 {
        return nums
    }

    pre := help(nums[:len(nums)/2])
    post := help(nums[len(nums)/2:])

    l1 := 0
    l2 := 0
    ans := make([]int, 0, len(nums))
    for l1 < len(pre) && l2 < len(post) {
        if pre[l1] <= post[l2] {
            ans = append(ans, pre[l1])
            l1++
            pair += l2 // 与下面区别开来，才可以避免重复计算
        }else{
            ans = append(ans, post[l2])
            l2++
        }
    }

    for l1 < len(pre) {
        ans = append(ans, pre[l1])
        l1++
        pair += l2
    }
    for l2 < len(post) {
        ans = append(ans, post[l2])
        l2++
    }
    return ans
}
```



## 快速排序

**要点提示**：在数组中选择一个主元，将数组分成两部分，前半部分所有元素都小于或等于主元；后半部分都大于主元。然后对两部分递归地应用快排。

### 基本实现

```go
// quickSort
func quickSort(arr []int) []int {
    if len(arr) == 0 {
        return arr
    }

    j := partition(arr)
    quickSort(arr[:j])
    if j < len(arr)-1 {
        quickSort(arr[j+1:])
    }
    return arr
}

// quickSort partition
func partition(arr []int) int {
    if len(arr) <= 1 {
        return len(arr) - 1
    }

    key := arr[0]
    lo := 1
    hi := len(arr) - 1
    for {
        for lo < hi && arr[lo] < key {
            lo++
        }
        for arr[hi] > key {
            hi--
        }
        if lo >= hi {
            break
        }
        tmp := arr[lo]
        arr[lo] = arr[hi]
        arr[hi] = tmp
    }
    arr[0] = arr[hi]
    arr[hi] = key
    return hi
}

```

### 参数

1. 复杂度

   平均：$O(n\log{n})$

2. 稳定性

   是不稳定的！

### 拓展





## 堆排序

**要点提示**：使用二叉堆。将所有元素添加到一个堆上，然后不断移除最小元素以获得一个排好序的线性表。

### 基本概念

#### 二叉堆

二叉堆是一棵完全二叉树，它每个节点都大于或等于它的左右孩子。那么根结点必然是数字最大的。取出`root`之后再次建堆，如果反复就可以得到有序数组。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gjnu4cmzpbj30jk0c874u.jpg" style="zoom:50%;" />

#### 堆的存储

堆一般可以存储到一个数组中，因为是完全二叉树，所以从堆 <--> 数组是唯一的。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gjnu4s2aidj30nm096tb1.jpg" style="zoom:50%;" />

设数组下标就是堆中结点的编号。对于编号为 $i$ 的结点，它的父亲节点编号为 $(i-1)/2$，左孩子编号 $2i+1$，右孩子编号 $2i+2$。

推导：

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjnkatm2qrj314h0u01kz.jpg" style="zoom:50%;" />



#### 添加一个新的元素

在已经建好的堆上添加一个新的元素，先添加到末尾，再调整堆的顺序，使其符合定义。插入的元素有两种可能性，一个比父节点小，那么不需要调整；另一个比父节点大，那么和父节点交换顺序。然后再继续向上调整。

```
将这个结点放在最后，并作为当前结点；
while (当前结点大于父结点) {
    当前结点与父结点交换；
    当前结点忘上推一层；
}
```

#### 删除一个元素

也就是取出根结点之后，让剩下的重构成一个堆。

```
用最后一个结点替换根结点；
让根结点成为当前结点；
while(当前结点的子结点有比它大的) {
    将当前结点和它的较大子结点交换；
    当前结点向下退一层；
}
```

### 基本实现

实现一个需要额外空间的堆排序

```go
// 定义一个结构体
type heap struct {
    arr    []int // 堆
    sorted []int // 排好序的数组
}

// heap sort
func heapSort(arr []int) []int {
    h := heap{}

    h.arr = arr
    // 1. build heap
    for i := range arr {
        h.add(i)
    }

    // 2. sort
    for i := 0; i < len(arr); i++ {
        h.remove()
    }

    arr = h.sorted
    return arr
}

func (h *heap) add(i int) {
    swap := func(arr []int, i, j int) {
        tmp := arr[i]
        arr[i] = arr[j]
        arr[j] = tmp
    }

    cur := i
    for h.getParent(cur) != -1 && h.arr[cur] > h.arr[h.getParent(cur)] {
        swap(h.arr, cur, h.getParent(cur))
        cur = h.getParent(cur)
    }
}

func (h *heap) remove() {
    h.sorted = append(h.sorted, h.arr[0])

    swap := func(arr []int, i, j int) {
        tmp := arr[i]
        arr[i] = arr[j]
        arr[j] = tmp
    }

    h.arr[0] = h.arr[len(h.arr) - 1]
    h.arr = h.arr[:len(h.arr)-1]
    cur := 0
    for {
        l := h.getLeftChild(cur)
        r := h.getRightChild(cur)
        if l != -1 && r != -1 {
            if h.arr[cur] < h.arr[l] || h.arr[cur] < h.arr[r] {
                if h.arr[l] > h.arr[r] {
                    swap(h.arr, l, cur)
                    cur = l
                } else {
                    swap(h.arr, r, cur)
                    cur = r
                }
            }else{
                return
            }
        } else if l != -1 {
            if h.arr[cur] < h.arr[l] {
                swap(h.arr, l, cur)
                cur = l
            }else{
                return
            }
        } else {
            return
        }
    }

}

func (h *heap) getParent(i int) int {
    if i == 0 {
        return -1
    }
    return (i - 1) / 2
}

func (h *heap) getLeftChild(i int) int {
    l := 2*i + 1
    if l < len(h.arr) {
        return l
    }
    return -1
}

func (h *heap) getRightChild(i int) int {
    l := 2*i + 2
    if l < len(h.arr) {
        return l
    }
    return -1
}
```



### 参数

1. 时间复杂度

   堆的高度为 $\log(n)$，用 add 方法会追踪叶子结点到根结点的路径，因此加一个新元素最多需要 $\log(n)$ 步。那么建立一个初始堆，需要 $n\log(n)$的时间。同理用 remove 方法也是这么多时间。所以复杂度是 $O(\log(n))$。

2. 稳定性

   是不稳定的！

## 桶排序和基数排序

是对于整数进行排序的高效算法。

**桶排序**：假设键值的范围是0 ～ t，那就需要 $t+1$个桶，如果元素的键值是 $i$，就把该元素放入同 $i$ 中。在范围比较小的时候，排序比较高效，否则会浪费很多空间。

**基数排序**：从桶排序扩展。将这些键值基于他们基数位置分为子组，然后反复地从最小的基数位置开始，堆其上的键值应用桶排序。比如一个三位数，分别按个位、十位、百位（或者反过来）来排序！

### 基本实现

#### 桶排序

```go
// bucket sort
func bucketSort(arr []int) []int {
    record := make([]int, maxNum + 1) // maxNum 为最大的数

    // 1. 统计个数
    for _, v := range arr {
        record[v]++
    }

    // 2. 累加和
    total := -1
    for i, v := range record {
        total += v
        record[i] = total
    }

    // 3. 放置
    res := make([]int, len(arr))
    for i := len(arr) - 1;i >= 0;i-- { // 从后往前放，可以保证稳定性！
        res[record[arr[i]]] = arr[i]
        record[arr[i]]--
    }

    return res
}
```



#### 基数排序

**从高位到低位**

这种比较好理解，相当于把百位=0的放在一个桶里，百位=1的放在第二个桶里，以此类推。然后针对一个桶里的数排十位。这是一种递归的排序方法。

```go
// radix sort from high to low
func radixSortFromHigh2Low(arr []int) []int {
    // element range from 0 to 999, please let maxNum = 1000
    bitSize := int(math.Log10(float64(maxNum))) // bitSize = 3
    res := helpRadix(arr, bitSize)
    return res
}

func helpRadix(arr []int, k int) []int {
    if k <= 0 || len(arr) == 0{
        return arr
    }

    record := make([][]int, 10) // 0 ~ 9
    for i := 0; i < len(arr); i++ {
        v := arr[i]
        cur := v / int(math.Pow10(k-1)) % 10
        record[cur] = append(record[cur], v)
    }

    res := make([]int, 0, len(arr))
    for _, v := range record {
        ans := helpRadix(v, k-1)
        res = append(res, ans...)
    }

    return res
}
```



**从低位到高位**

这种相对难理解一些，因为这是先排个位，再排高位的，为什么排了3次就能有序呢？<u>这是因为基数排序是稳定的！</u>。比如  $abc$ 和 $xyz$，如果 $c \lt z$，在 $a==x, b == y$的情况下，由于排序是稳定的，那么他们3次排完之后 $abc$必然在 $xyz前面$ 。



```go
// radix sort from low to high
func radixSortFromLow2High(arr []int) []int {
    // element range from 0 to 999, please let maxNum = 1000
    bitSize := int(math.Log10(float64(maxNum))) - 1
    for k := bitSize;k >= 0;k-- {
        record := make([]int, 10) // 0 ~ 9
        for _, v := range arr {
            cur := v/int(math.Pow10(2 - k)) % 10
            record[cur]++
        }
        total := -1
        for i, v := range record {
            total += v
            record[i] = total
        }
        // 3. 放置
        res := make([]int, len(arr))
        for i := len(arr) - 1;i >= 0;i-- {
            bucket := arr[i]/int(math.Pow10(2 - k)) % 10
            res[record[bucket]] = arr[i]
            record[bucket]--
        }
        arr = res
    }
    return arr
}
```

### 参数

1. 复杂度

   桶排序： $O(n+t)$， $n$ 为元素个数， $t$ 为取值范围  
   .

   基数排序： $O(d(n+r))$， $n$为元素个数，$d$为基数位数， $r$为桶个数

2. 稳定性

   稳定的！

## 总结

| 排序算法 | 平均情况/最差/最好                    | 稳定性 |
| -------- | ------------------------------------- | ------ |
| 插入排序 | $O(n^2)/O(n^2)/O(n)$                  | 稳定   |
| 冒泡排序 | $O(n^2)/O(n^2)/O(n)$                  | 稳定   |
| 快速排序 | $O(n\log{n})/O(n^2)/O(n\log{n})$      | 不稳定 |
| 选择排序 | $O(n^2)/O(n^2)/O(n^2)$                | 不稳定 |
| 堆排序   | $O(n\log{n})/O(n\log{n})/O(n\log{n})$ | 不稳定 |
| 归并排序 | $O(n\log{n})/O(n\log{n})/O(n\log{n})$ | 稳定   |
| 基数排序 | $O(d(n+r))/O(d(n+r))/O(d(n+r))$       | 稳定   |











