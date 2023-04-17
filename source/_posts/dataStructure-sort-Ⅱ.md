---
title: 排序算法 - Ⅱ
date: 2021-11-12 22:45:58
tags: ['堆排序','归并排序','快速排序']
categories: ['数据结构','排序']
description:
photos:
---

## 排序算法 - Ⅱ

排序基础-Ⅰ 学习了冒泡排序、简单选择排序、直接插入排序以及希尔排序(直接插入排序的升级)。Ⅱ 学习堆排序(简单选择排序的升级)、归并排序以及快速排序(冒泡排序的升级)

{% note primary %}

本文未指定出处的引用均出自 [大话数据结构](https://book.douban.com/subject/6424904/)

{% endnote %}

### 堆排序

> 堆是具有下列性质的完全二叉树：每个节点的值都大于或等于其左右孩子的值，称为大顶堆；或者每个结点的值都小于或等于其左右孩子结点的值，称为小顶堆
>
> 堆排序(Heap Sort)就是利用堆(假设利用大顶堆)进行排序的方法。它的基本思想是，将待排序的序列构造成一个大顶堆。此时，整个序列的最大值就是对顶的根节点。将它移走(其实就是将其与堆数组的末尾元素交换，此时末尾元素就是最大值)，然后将剩余的 n - 1 个序列重新构造成一个堆，这样会得到 n 个元素中的次小值。如此反复执行，便能得到一个有序序列了。

<!-- more -->

{% note primary%}

[可视化](https://www.cs.usfca.edu/~galles/visualization/HeapSort.html)

{% endnote %}

```JavaScript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));


function heapSort(array) {
  for (let i = Math.floor(array.length / 2) - 1; i >= 0; i--) {
    // 完全二叉树性质，从最后一个双亲节点开始遍历
    heapAdjust(array, i, array.length - 1);
  }


  for (let j = array.length - 1; j > 0; j--) {
    [array[0], array[j]] = [array[j], array[0]];
    heapAdjust(array, 0, j - 1);
  }
}


function heapAdjust(array, start, end) {
  let parents = start;
  for (let child = 2 * parents + 1; child <= end; child = 2 * child + 1) {
    // 完全二叉树性质，定位子节点
    if (child + 1 <= end && array[child] < array[child + 1]) ++child; // 找出左右子节点中最大的


    if (array[parents] >= array[child]) break;
    // 因为是从最后一个双亲节点开始遍历的，可以保证当上层满足双亲节点大于子节点时，整个子树满足大顶堆要求


    [array[parents], array[child]] = [array[child], array[parents]];


    parents = child;
  }
}


heapSort(sort);
console.log(sort); //  [13, 33, 47, 52, 57, 64, 68, 84, 87, 98]
```

### 归并排序

> 归并排序(Merging Sort) 就是利用归并的思想实现的排序方法。它的原理是假设初始序列含有 n 个记录，则可以 看成是 n 个有序的子序列，每个子序列的长度为 1，然后两两归并，得到 ⌈ n/2 ⌉ ( ⌈ x ⌉ 表示不小于 x 的最小整数)个长度为 2 或 1 的有序子序列；再两两归并，.......，如此反复，直至得到一个长度为 n 的有序序列为止，这种排序方法称为 2 路归并排序

{% note primary%}

[可视化](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)

{% endnote %}

#### 递归实现

> 1. 申请空间，使其大小为两个排序序列之和，该空间用来存放合并后的序列
> 2. 设定两个指针，最初位置分别为两个已经排序序列的起始位置
> 3. 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
> 4. 重复步骤 3 直到某一指针到达序列尾
> 5. 将另一序列剩下的所有元素直接复制到合并序列尾
>
> —— Wikipedia. 归并排序

##### 实现 Ⅰ

这种写法每次递归仅声明一个数组，巧妙利用下标复用数组，改变原数组。应该是较省内存的 **(未实测)** ，但是理解起来比较麻烦

```JavaScript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));


function mergeSort(array, result, start, end) {
  if (start === end) {
    result[start] = array[start];
  } else {
    const middle = (start + end) >> 1;
    // 将array，平分为 array[start, middle]和 array[middle + 1, end]
    const supResult = [];
    mergeSort(array, supResult, start, middle);
    // 递归将 array[start, middle]归并为有序的 supResult[start, middle]
    mergeSort(array, supResult, middle + 1, end);
    // 递归将 array[middle + 1, end]归并为有序的 supResult[middle + 1, end]
    merge(result, supResult, start, middle, end);
    // 将 supResult[start, middle] 和 supResult[middle + 1, end] 归并到 result[start, end]
  }
}

// 将有序的supResult[start,middle]和supResult[middle+1, end] 归并为有序的 result[Start, end]
function merge(result, supResult, start, middle, end) {
  let curIndex = start;
  let left, right;
  for (left = start, right = middle + 1; left <= middle && right <= end; curIndex++) {
    // 将supResult 中记录由小到大归并到Result
    if (supResult[left] <= supResult[right]) {
      result[curIndex] = supResult[left++];
    } else {
      result[curIndex] = supResult[right++];
    }
  }
  if (left <= middle) {
    for (let i = 0; i <= middle - left; i++) {
      result[curIndex + i] = supResult[left + i]; // 将剩余的supResult 复制到 result
    }
  }
  if (right <= end) {
    for (let j = 0; j <= end - right; j++) {
      result[curIndex + j] = supResult[right + j]; // 将剩余的supResult 复制到 result
    }
  }
}
mergeSort(sort, sort, 0, 9);
console.log(sort); // [7, 11, 24, 27, 27, 40, 46, 48, 51, 88]
```

##### 实现 Ⅱ

这种写法就很直接，也好理解，但是申请了较多的空间(变量)，并且返回的数组并不是原数组

```JavaScript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));


function mergeSort(array) {
  if (array.length <= 1) return array;
  const middle = array.length >> 1;
  const left = array.slice(0, middle);
  const right = array.slice(middle);
  return merge(mergeSort(left), mergeSort(right));
}


function merge(left, right) {
  const result = [];
  while (left.length > 0 && right.length > 0) {
    if (left[0] <= right[0]) {
      result.push(left.shift());
    } else {
      result.push(right.shift());
    }
  }
  result.push(...left, ...right);
  return result;
}


const sortResult = mergeSort(sort);
console.log(sortResult); // [0, 5, 16, 17, 27, 38, 52, 54, 72, 82]
console.log(sortResult === sort); // false
```

#### 迭代实现

> 1. 将序列每相邻两个数字进行归并操作，形成⌈ n/2 ⌉ 个序列，排序后每个序列包含两/一个元素
> 2. 若此时序列数不是 1 则将上述序列再次归并，形成⌈ n/4 ⌉个序列，每个序列包含四/三个元素
> 3. 重复步骤 2，直到所有元素排序完毕，即序列数为 1
>
> ——Wikipedia. 归并排序

```javascript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));

function mergeSort(array) {
    const result = [];
    let windowLength = 1; // 初始窗口长度为1
    while (windowLength < array.length) {
        mergeByWindow(result, array, windowLength, array.length);
        // 当前窗口 array 有序，result 存储归并后结果

        windowLength *= 2;

        mergeByWindow(array, result, windowLength, array.length);
        // 当前窗口 result有序， array 存储归并后结果

        windowLength *= 2;
    }
}

// 按窗口长度，将有序的supResult，归并为有序的result
function mergeByWindow(result, supResult, windowLength, arrayLength) {
    let index = 0;

    while (index <= arrayLength - 2 * windowLength) {
        // 保证能够归并两个窗口长度的数据，不足部分则作为独立窗口归并
        merge(result, supResult, index, index + windowLength - 1, index + 2 * windowLength - 1);
        index = index + 2 * windowLength;
    }
    if (index + windowLength < arrayLength) {
        // 最后剩两个序列时，index + window 必然小于数组长度
        merge(result, supResult, index, index + windowLength - 1, arrayLength - 1);
    } else {
        // 否则只剩下单个子序列
        for (let j = index + 1; j < arrayLength; j++) {
            result[j] = supResult[j];
        }
    }
}

// 上方递归实现Ⅰ 方法
// 将有序的supResult[start,middle]和supResult[middle+1, end] 归并为有序的 result[Start, end]
function merge(result, supResult, start, middle, end) {
    let curIndex = start;
    let left, right;
    for (left = start, right = middle + 1; left <= middle && right <= end; curIndex++) {
        // 将supResult 中记录由小到大归并到Result
        if (supResult[left] <= supResult[right]) {
            result[curIndex] = supResult[left++];
        } else {
            result[curIndex] = supResult[right++];
        }
    }
    if (left <= middle) {
        for (let i = 0; i <= middle - left; i++) {
            result[curIndex + i] = supResult[left + i]; // 将剩余的supResult 复制到 result
        }
    }
    if (right <= end) {
        for (let j = 0; j <= end - right; j++) {
            result[curIndex + j] = supResult[right + j]; // 将剩余的supResult 复制到 result
        }
    }
}

mergeSort(sort);
console.log(sort); // [8, 21, 25, 30, 46, 47, 65, 69, 74, 93]
```

### 快速排序

> 快速排序(Quick Sort)的基本思想是：通过一趟排序将待排记录分割成独立的两部分，其中一部分记录的关键字均比另一部分记录的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序的目的

{% note primary%}

[可视化](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)

{% endnote %}

```Javascript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));

function quickSort(array, start, end) {
    if (start < end) {
        const pivotIndex = partition(array, start, end);

        quickSort(array, start, pivotIndex - 1); // 对低子表递归排序
        quickSort(array, pivotIndex + 1, end); // 对高子表递归排序
    }
}

// 交换Array记录，使基准值记录到位，并返回其所在位置
// 在此之前（后）的记录均不大（小）于它
function partition(array, left, right) {
    const pivotValue = array[left]; // 用子表第一个记录为基准值

    while (left < right) {
        // 从表的两端交替向中间扫描

        while (left < right && array[right] >= pivotValue) {
            right--;
        }

        [array[left], array[right]] = [array[right], array[left]]; // 将比基准值小的值交换到左侧

        while (left < right && array[left] <= pivotValue) {
            left++;
        }

        [array[left], array[right]] = [array[right], array[left]]; // 将比基准值大的值交换到右侧
    }
    return left;
}

quickSort(sort, 0, sort.length - 1);
console.log(sort); // [25, 26, 29, 41, 51, 55, 60, 62, 63, 80]
```

#### 优化

##### 选取基准值

> **三数取中(median-of-three)**；即取三个关键字先进行排序，将中间数作为基准值，一般是取左端、右端和中间三个数

partition 函数取基准值部分变换为三数取中逻辑即可

```Javascript
const middle = left + ((right - left) >> 1);
if (array[left] > array[right]) {
    [array[left], array[right]] = [array[right], array[left]];
}
if (array[middle] > array[right]) {
    [array[middle], array[right]] = [array[right], array[middle]];
}
if (array[left] > array[middle]) {
    [array[left], array[middle]] = [array[middle], array[left]];
}
const pivotValue = array[middle]; // 用子表 头中尾 三个值中的中间数作为基准值
```

## 参考

[1\] [程杰. 大话数据结构.](https://book.douban.com/subject/6424904/)

[2\][wikipedia. 堆排序](https://zh.wikipedia.org/wiki/%E5%A0%86%E6%8E%92%E5%BA%8F)

[3\][wikipedia. 归并排序](https://zh.wikipedia.org/wiki/%E5%BD%92%E5%B9%B6%E6%8E%92%E5%BA%8F)

[4\][wikipedia. 快速排序](https://zh.wikipedia.org/wiki/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F)

