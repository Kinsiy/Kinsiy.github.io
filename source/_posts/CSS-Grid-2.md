---
title: CSS - 栅格布局 - Ⅱ
mathjax: false
date: 2022-07-17 20:16:01
tags:
categories:
description:
photos:
---

承接上篇[CSS - 栅格布局 - Ⅰ](https://kinsiy.github.io/CSS-Grid/) , 上篇说了如何定义栅格轨道及如何将栅格元素附加到栅格轨道/区域上.这篇主要说下栅格流/栏距/栅格的对齐方式等

## 栅格流

前面附加元素时基本上都明确指定了栅格元素在栅格中的位置,对与未明确指定的栅格元素,其位置将按照栅格流自动放入栅格中.

栅格流主要分为两种模式,即行优先和列优先,不过二者都可以通过密集流增强.栅格流通过`grid-auto-flow`设置

<!--more-->

```html
<ol class="grid">
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>4</li>
  <li>5</li>
</ol>

<style>
  .grid {
    display: grid;
    width: 45em;
    height: 8em;
    grid-auto-flow: row;
  }
</style>
```

![image-20220717203504024](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220717203504024.png)

```css
.grid {
  grid-auto-flow: column;
}
```

![image-20220717203632833](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220717203632833.png)

对栅格元素的尺寸,如果没有显示设定,元素的尺寸将自动调整,以便附加到所定义的栅格线上.

```css
.grid li {
  width: 7.5em;
  height: 2em;
}
```

![image-20220717204038058](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220717204038058.png)

对比为栅格元素设定尺寸前后两张图,不难发现每个栅格的起始位置是一致的,但结束位置不一致.这表面,栅格流其实放置的是栅格区域,然后再把栅格元素附加到栅格区域中.

自动附加的栅格元素不会考虑其他其他栅格元素的存在,观察下面的图,可以发现,有些图被覆盖了.

![image-20220717210745862](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220717210745862.png)

可以通过为占据多个轨道的栅格元素添加适合的类名,指定其跨度,得到不重叠的布局

```css
.wide {
  grid-column: auto / span 2;
}

.tall {
  grid-row: auto / span 2;
}
```

![image-20220717212225412](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220717212225412.png)

注意上图, 05 右边存在空白, 这是因为我们设定的栅格流设定的是`row`. 行流在每一行从左向右排,如果有足够的空间放下一个栅格元素,那就把该栅格元素方在哪儿,如果放不下或栅格单元被其他栅格元素占据了,就挑个那个栅格单元. 

05 右边的栅格单元明显放不下06, 所以跳过了 06 导致了空白.

 如果想让栅格元素尽量靠紧,而不管顺序会受到什么影响. 在`grid-auto-flow`的值里加上关键字`dense`即可

```css
.grid {
  grid-auto-flow: row dense;
}
```

![image-20220717212949221](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220717212949221.png)

## 自动增加栅格线

当栅格元素超出了栅格边界时,默认情况下会根据布局要求增加所需的行或列.因此,如果在行主导的栅格末尾有一个跨三行的栅格元素,将在显式栅格后增加三行.

默认情况下,自动增加的行是所需的最小尺寸.可以使用`grid-auto-rows`和`grid-auto-columns`控制自动增加的轨道尺寸

```css
.grid {
  display: grid;
  grid-template-rows: 50px 50px;
  grid-template-columns: 50px 50px;
}
```

![image-20220718211818826](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220718211818826.png)

```css
.grid {
  grid-auto-columns: 50px;
}
```

![image-20220718212044968](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220718212044968.png)

## grid简写属性

grid属性的作用是以简洁的句法定义栅格模板.**未定义的值都重置为默认值**

下面两个声明是等效的

```css
.grid {
  grid: 
    "h h h h" 3em
    ". c c ." 4em
    "f f f f" 5em /
    2em 3em minmax(10em, 20em) 2em;
}

.grid {
  grid-template-areas:
    "h h h h"
    ". c c ."
    "f f f f";
  grid-template-rows: 3em 4em 5em;
  grid-template-columns: 2em 3em minmax(10em, 20em) 2em;
}
```

在使用grid声明时,`grid-template-rows`的值被拆开了,分配与每行栅格区域声明后.

当不声明栅格区域时,可以简写为

```css
.grid {
  gird: 3em 4em 5em / 2em 3em minmax(10em, 20em) 2em;
}
```

融合命名栅格线

```css
.grid {
  grid: 
    "h h h h" 3em
    [main-start] ". c c ." 4em [main-end]
    "f f f f" 5em [page-end] /
    2em 3em minmax(10em, 20em) 2em;
}
```

{% note warning %}

`grid`属性会将所有未设置的值设定为默认值, 所以一定要把`grid`属性设定在其他与栅格有关的其他属性之前

{% endnote %}

### 子栅格

`grid`属性还可以取一个值: `subgrid`

此时,每个栅格元素的子元素将根据父栅格对齐. **略**

## 释放栅格空间

### 栏距

栏距是两个栅格轨道之间的间隔.使用`row-gap`, `column-gap`设定

```css
.grid {
  display: grid;
  grid: repeat(3, 50px) / repeat(4, 50px);
  row-gap: 2em;
  column-gap: 2em;
}

/* 使用简写属性 */
.grid {
  grid-gap: 2em / 2em;
}
```

![image-20220718220605124](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220718220605124.png)

### margin

在栅格区域中,盒模型中的外边距属性`margin`与普通布局稍有不同,设置`margin`时**元素在外边距的边界处附加到栅格中**

这意味着, 正外边距使元素向栅格内部收缩, 负外边距, 元素向栅格外部扩张, 也不会出现外边距坍缩的现象

{% note primary %}

计算栅格轨道的尺寸时将忽略栅格元素的外边距.这意味着,不管栅格元素的外边距有多大,都不会改变`min-content`列的尺寸,增加栅格元素的外边距也不会改变使用`fr`单位设定尺寸的栅格轨道的尺寸

{% endnote %}

{% note warning %}

对于绝对定位, 书<sup>[1]</sup>中指出开始栅格线和结束栅格线围成的栅格区域用作容纳块和定位上下文,栅格元素就在这个上下文中定位.

但截至`2022/07/19`,在`Chrome v103.0.5060.114`没有测试成功

{% endnote %}

## 栅格对齐

| 属性            | 对齐的目标                     | 适用于   |
| --------------- | ------------------------------ | -------- |
| justify-self    | 行内方向(横向)上的一个栅格元素 | 栅格元素 |
| justify-items   | 行内方向(横向)上的全部栅格元素 | 栅格容器 |
| justify-content | 行内方向(横向)上的整个栅格     | 栅格容器 |
| align-self      | 块级方向(纵向)上的一个栅格元素 | 栅格元素 |
| align-items     | 块级方向(纵向)上的全部栅格元素 | 栅格容器 |
| align-content   | 块级方向(纵向)上的整个栅格     | 栅格容器 |

### 对齐单个栅格元素

`justify-self`各个值的取值情况, `align-self`除方向不一致外,这8个取值效果完全一样

![image-20220719220231406](D:\CSS\image-20220719220231406.png)

另外纵向对齐还可以使用`baseline`,`last-baseline`.这两个值把栅格元素中的第一条或最后一条基线与栅格轨道中最高或最低的基线对齐. 

`flex-start`与`flex-end`之应该在弹性盒布局中使用,在其他布局中等同与`start`与`end`

### 对齐全部栅格元素

`justify-items`与`align-items`取值与`justify-self`/`align-self`完全一致,只不过应用与栅格容器中,对所有栅格元素起作用.

`justify-content`/`align-content`的作用与`flex`布局中的作用类似, 用于分配剩余空间.

![image-20220719222614834](C:\Users\Kinsiy\AppData\Roaming\Typora\typora-user-images\image-20220719222614834.png)

分配多出的空间其实就是调整栅格栏距的尺寸.如果没有声明栏距,那么分配的空间将成为栏距.如果声明了栏距,其尺寸将根据分配的情况调整

{% note warning %}

`stretch`把余下的空间均匀的分配给各个栅格轨道,而不分配给栏距. 

截至`2022/07/19`,在`Chrome v103.0.5060.114`没有测试成功

{% endnote %}

## 分层和排序

对于重叠的栅格元素,默认按照在文档源码中的顺序叠放.可以使用`z-index`或者`order`进行调整, `z-index`无需多说, 与其他布局的行为一样. 

也可以调整`order`的值改变栅格元素的顺序,以改变叠放顺序,但是**不推荐**使用.

仅当与视觉顺序无需与阅读和导航顺序一致时才可以使用`order`属性; 否则应该调整元素在文档源码中的顺序



## 参考

[1\][Eric A. Meyer / Estelle Weyl. CSS权威指南（第四版）](https://book.douban.com/subject/33398314/)
