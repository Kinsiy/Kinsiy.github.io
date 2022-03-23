---
title: 排序算法-Ⅰ
date: 2021-11-11 21:24:28
tags: ['冒泡排序','简单选择排序','直接插入排序','希尔排序']
categories: ['数据结构','排序']
description:
photos:
---

## 排序基本概念


{% note primary %}


本文未指定出处的引用均出自 [大话数据结构](https://book.douban.com/subject/6424904/)


{% endnote %}


### 严格定义


> 假设含有n个记录的序列为{r<sub>1</sub>, r<sub>2</sub>, ...... , r<sub>n</sub>}, 其相应的关键字分别为{k<sub>1</sub>, k<sub>2</sub>, ...... , k<sub>n</sub>}, 需确定1, 2, ......, n 的一种排列p<sub>1</sub>, p<sub>2</sub>, ......, p<sub>n</sub>, 使其相应的关键字满足k<sub>p1</sub> ≤ k<sub>p2</sub> ≤ ...... ≤ k<sub>pn</sub> (非递减或非递增)关系，即使得序列成为一个按关键字有序得序列{r<sub>p1</sub>, r<sub>p2</sub>, ......, r<sub>pn</sub>}, 这样得操作就称为排序 


### 稳定性


> 假设k<sub>i</sub> = k<sub>j</sub> (1 ≤ i ≤ n, 1 ≤ j ≤ n, i ≠ j)，且在排序前的序列中r<sub>i</sub>领先于r<sub>j</sub> (即 i ＜ j)。如过排序后 r <sub>i</sub> 仍然领先于 r<sub>j</sub> ，则称所用的排序方法是稳定的；反之，若可能使得排序后的序列中 r<sub>j</sub> 领先于 r<sub>i</sub> ，则称所用的排序方法是不稳定的。

也就是如果一个排序算法是稳定的，当有两个相等键值的纪录**R**和**S**，且在原本的列表中**R**出现在**S**之前，在排序过的列表中**R**也将会是在**S**之前。

<!-- more -->


### 原地算法


> 在计算机科学中，一个原地算法（in-place algorithm，也称“就地算法”）是基本上不需要借助额外的数据结构就能对输入的数据进行变换的算法。不过，分配少量空间给部分辅助变量是被允许的。算法执行过程中，输入的数据往往会被输出结果覆盖。原地算法只能通过替换或交换元素的方式来修改原始的输入。不满足“原地”原则的算法也被称为非原地（not-in-place）算法或异地（out-of-place）算法。
>
> ——wikipedia. 原地算法


## 冒泡排序


> 冒泡排序(Bubble Sort) 一种交换排序，它的基本思想是：两两比较相邻记录的关键字，如果反序则交换，直到没有反序的记录为止

{% note primary%}

[可视化](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)

{% endnote %}


```JavaScript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));

function bubbleSort(array) {
  for (let i = 0; i < array.length; i++) {
    for (let j = array.length - 1; j > i; j--) {
      if (array[j - 1] > array[j]) {
        [array[j - 1], array[j]] = [array[j], array[j - 1]];
      }
    }
  }
}

bubbleSort(sort);
console.log(sort); // [0, 4, 16, 31, 32, 34, 84, 95, 97, 99]
```


### 优化


在待排序序列仅前几个记录是无序的情况下，例如 [2, 1, 3, 4, 5, 6, 7]。添加交换标志位可减少无用比较次数


```javascript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));


function bubbleSort(array) {
  let swapFlag = true;
  for (let i = 0; i < array.length && swapFlag; i++) {
    swapFlag = false;
    for (let j = array.length - 1; j > i; j--) {
      if (array[j - 1] > array[j]) {
        [array[j - 1], array[j]] = [array[j], array[j - 1]];
        swapFlag = true;
      }
    }
  }
}

bubbleSort(sort);
console.log(sort); // [7, 9, 25, 43, 54, 55, 59, 82, 89, 96]
```


## 简单选择排序


> 简单选择排序(Simple Selection Sort) 就是通过 n - i 次关键字间的比较，从 n - i + 1 个记录中选出关键字最小的记录，并和第 i (1 ≤ i ≤ n) 个记录交换之

{% note primary%}

[可视化](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)

{% endnote %}


```javascript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));


function simpleSelectSort(array) {
  for (let i = 0; i < array.length; i++) {
    let minIndex = i;
    for (let j = i + 1; j < array.length; j++) {
      if (array[j] < array[minIndex]) {
        minIndex = j;
      }
    }
    if (i !== minIndex) {
      [array[i], array[minIndex]] = [array[minIndex], array[i]];
    }
  }
}


simpleSelectSort(sort);
console.log(sort); // [5, 25, 34, 39, 55, 65, 70, 82, 86, 94]
```


## 直接插入排序


> 直接插入排序(Straight Insertion Sort) 的基本操作就是将一个记录插入到已经排好序的有序表中，从而得到一个新的、记录数增1的有序表

{% note primary%}

[可视化](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)

{% endnote %}


```javascript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));

function insertSort(array) {
  for (let i = 1; i < array.length; i++) {
    if (array[i] < array[i - 1]) {
      let value = array[i];
      let j;
      for (j = i - 1; array[j] > value; j--) {
        array[j + 1] = array[j];
      }
      array[j + 1] = value;


      /** 可以不声明变量
       * let j;
       * for (j = i - 1; array[j] > array[j + 1]; j--) {
       *  [array[j + 1], array[j]] = [array[j], array[j + 1]];
       * }
       *
       */
    }
  }
}

insertSort(sort);
console.log(sort);
```


## 希尔排序


> 希尔排序（Shellsort），也称递减增量排序算法，是插入排序的一种更高效的改进版本。希尔排序是非稳定排序算法。
>
> ——wikipedid. 希尔排序

{% note primary%}

[可视化](https://www.cs.usfca.edu/~galles/visualization/ComparisonSort.html)

{% endnote %}


```JavaScript
const sort = Array.from({ length: 10 }, (_, i) => ~~(Math.random() * 100));

function shellSort(array) {
  const evenGap = (i) => 9 * Math.pow(4, i) - 9 * Math.pow(2, i) + 1;
  const oddGap = (i) => Math.pow(2, i + 2) * (Math.pow(2, i + 2) - 3) + 1;


  let gapList = [];
  let gap;
  let n = 0;
  do {
    gap = evenGap(n);
    gap <= array.length && gapList.unshift(gap);


    gap = oddGap(n);
    gap <= array.length && gapList.unshift(gap);        // unshift 方便下面遍历


    n++;
  } while (gap < array.length);


  console.log(gapList); // (1, 5, 19, 41, 109,...) 的反序


  // 上半部分只是为了生成已知最好的步长序列，下面才是排序的主要代码
  for (const gap of gapList) {
    for (let i = gap; i < array.length; i++) {
      let tempValue = array[i];
      let j;
      for (j = i - gap; j >= 0 && array[j] > tempValue; j -= gap) { // gap定好这里无需 >= 0
        array[j + gap] = array[j];
      }
      array[j + gap] = tempValue;
      
      /** 可以不声明变量
       * let j;
       * for (j = i - gap; j >= 0 && array[j] > array[j + gap]; j -= gap) {
       *  [array[j + gap], array[j]] = [array[j], array[j + gap]];
       * }
       *
       */
    }
  }
}

shellSort(sort);
console.log(sort);
```


> 步长的选择是希尔排序的重要部分。只要最终步长为1任何步长序列都可以工作。算法最开始以一定的步长进行排序。然后会继续以一定步长进行排序，最终算法以步长为1进行排序。当步长为1时，算法变为普通插入排序，这就保证了数据一定会被排序。
>
> Donald Shell最初建议步长选择为 n/2 并且对步长取半直到步长达到1。虽然这样取可以比  O (n<sup>2</sup>) 类的算法（插入排序）更好，但这样仍然有减少平均时间和最差时间的余地
>
> 已知的最好步长序列是由Sedgewick提出的(1, 5, 19, 41, 109,...)，该序列的项，从第0项开始，偶数来自9 \* 4<sup>i</sup> - 9 \* 2<sup>i</sup> +1 和奇数来自 2<sup>i+2</sup> * (2<sup>i+2</sup> - 3) +1 这两个算式这项研究也表明“比较在希尔排序中是最主要的操作，而不是交换。”用这样步长序列的希尔排序比插入排序要快，甚至在小数组中比快速排序和堆排序还快，但是在涉及大量数据时希尔排序还是比快速排序慢。
>
> 另一个在大数组中表现优异的步长序列是（斐波那契数列除去0和1将剩余的数以黄金分割比的两倍的幂进行运算得到的数列）：(1, 9, 34, 182, 836, 4025, 19001, 90358, 428481, 2034035, 9651787, 45806244, 217378076, 1031612713,…)
>
> ——wikipedia. 希尔排序

## 参考

[1\] [程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/)

[2\] [wikepedia. 排序算法](https://zh.wikipedia.org/wiki/%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)

