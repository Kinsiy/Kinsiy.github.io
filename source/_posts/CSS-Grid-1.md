---
title: CSS - 栅格布局 - Ⅰ 
mathjax: false
date: 2022-07-16 11:08:29
tags: ['CSS', 'Grid']
categories:
description:
photos:
---

很早之前就看过如何应用栅格布局,但平常都是flex布局用的多.导致grid布局都忘了.最近重新复习了一遍.这里记录下

## 基本术语

- **栅格容器**:  是确立栅格格式化上下文的框体,即定义一个栅格区域,其中的元素根据栅格布局(而非块级布局)规则排布
- **栅格元素**:  是在栅格格式化上下文中参与栅格布局的东西
- **栅格轨道**: 指两条相邻的栅格线之间夹住的整个区域,从栅格容器的一边延伸到对边,即栅格列或栅格行
- **栅格单元**: 指四条栅格线限定的区域,内部没有其他栅格线贯穿
- **栅格区域**:  指任何四条栅格线限定的矩形区域,有一个或多个栅格单元构成

<!-- more -->

## 放置栅格线

使用`grid-template-rows`, `grid-template-columns`大致定义栅格模板中的栅格线

### 宽度固定的栅格轨道

“**宽度固定**”的栅格线指栅格线之间的距离不随栅格轨道中的内容变化而变化

```css
#grid {
  display: grid;
  grid-template-columns: 200px 50% 100px;
}
```

为栅格线命名

```css
#grid {
  display: grid;
  grid-template-columns: [start col-a] 200px [col-b] 50% [col-c] 100px [stop end last];
}
```

为轨道尺寸设定极值

```css
#grid {
  display: grid;
  grid-template-columns: [start col-a] 200px [col-b] 50% [col-c] 100px [stop end last];
  grid-template-rows: [start masthead] 3em [content] minmax(3em, 100%) [footer] 2em [stop end]
}
```

注意: 如果minmax()中指定的最大值比最小值小,最大值将被忽略,最小值将用于设定宽度固定的轨道长度

也可以使用`calc()`计算

### 弹性栅格轨道

弹性栅格轨道的尺寸基于弹性容器中非弹性轨道意外的空间确定,或者基于整个轨道中的具体内容而定

使用`fr`单位把余下的空间分成一定份数, 以分配给各栏

```css
#grid {
  display: grid;
  grid-template-columns: 15em 1fr 10%;
}
```

注意: `minmax()`表达式的最小值部分不允许使用`fr`单位

使用`min-content`, `max-context`根据内容设定轨道尺寸

- max-content. 占据内容所需的最大空间. “宽度尽量大,以防换行”
- min-content. 占据内容所需的最小空间.  “尽量少占据空间,够显示内容即可”, 对文本来说,只保证最长的单词能够在一行里完整显示,会导致大量断行

```css
#grid {
  display: grid;
  grid-template-columns: max-content max-content max-content max-content;
  grid-template-rows: max-content max-content max-content;
}
```

### 自动适配

除`min-content`和`max-content`外,还可以使用`fix-content()`函数以简练的方式表达特定类型的尺寸模型

```css
#grid {
  display: grid;
  grid-template-colums: 1fr fit-content(150px) 2fr;
}
```

`fit-content`的伪公式

```js
fit-content(arg) => min(max-content, max(min-content, arg))
```

可以这么理解: `fit-content(arg)` 等价于 `minmax(min-content, max-content)`, 除非参数指定的值设置了更大的上限

### 重复栅格线

可以使用`repeat()`重复声明同一尺寸的轨道

```css
#grid {
  display: grid;
  grid-template-columus: repeat(10, 5em);
}
```

上述规则创建了10个列轨道,每个轨道的宽度为5em, 共计50em.

在repeat中, 轨道的尺寸可以使用任何值,可以是`min-content`和`max-content`,也可以是`fr`值和`auto`等,而且不限于只提供一个值. 但是不能在重复中嵌套重复

```css
#grid {
  display: grid;
  grid-template-columus: repeat(4, 10px [col-start] 250px 1fr [col-end] 10px);
}
```

还可以使用`auto-fill`,对轨道尺寸进行简单重复,直到填满整个栅格容器为止.

```css
#grid {
  display: grid;
  grid-template-columus: repeat(auto-fill, [top] 5em [bottom])
}
```

每隔5em放置一条行栅格线,直到没有空间为止.

`auto-fill`的局限是只能有一个可选的栅格线名称,一个尺寸固定的轨道和另一个栅格线名称.上面`[top] 5em [bottom]`就是这种模式下值最多的情况. 且在一个轨道模板中只能有一个自动重复的模式. 

但是, 可以和固定数量的重复结合

```css
#grid {
  display: grid;
  grid-template-columus: repeat(3, 20em) repeat(auto-fill, 100px)
}
```

与`auto-fill`的行为类似的声明还有一个`auto-fit`, 其与`auto-fill`的区别是, **没有栅格元素的轨道将被剔除**

## 栅格区域

使用`grid-template-areas`可以栅格画出来

```css
#grid {
  display: grid;
  grid-template-areas: 
    "h h h h"
    "l c c r"
    "l f f f";
}
```

在`grid-template-areas`中每个字符串(放在一对双引号中)定义栅格中的一行. 相同的字符串值用于定义栅格区域的形状. 

注意: 栅格区域必须是矩形, 如果区域形状太复杂,整个模板都将失效

对不想命名的区域可以使用一个或多个`.`字符占位

```css
#grid {
  display: grid;
  grid-template-areas: 
    "header header header header"
    "left ... ... right"
    "footer footer footer footer";
  grid-template-colums: 1fr 20em 20em 1fr;
  grid-template-rows: 40px 10em 3em;
}
```

定义好栅格区域之后, 接下来要使用前面介绍的`grid-template-columns`和`grid-template-rows`定义栅格轨道的尺寸.

对于命名区域, 会自动为构成栅格区域的栅格线进行命名, 对矩形的左/上两条栅格线命名为`区域名-start`, 右/下则为`区域名-end`. 对与上方的规则来说, 自动为第一条列栅格线命名`header-start`, 最后一条列栅格线命名`header-end`, 行栅格线同理.

除显式声明栅格区域, 也可以通过为栅格线命名`区域名-start`,`区域名-end`的形式隐式声明栅格区域**(不建议)**

## 附加元素

### 使用列线和行线

可以通过使用`grid-row-start`,`grid-row-end`,`grid-column-start`,`grid-column-end`.四个属性把栅格元素附加到栅格中.

这几个属性的意思是, “我想把元素的边界附加到某条栅格线上”

```css
.grid {
  display: grid;
  width: 50em;
  grid-template-rows: repeat(5, 5em);
  grid-template-columns: repeat(10, 5em);
}

.one {
  grid-row-start: 2;
  grid-row-end: 4;
  grid-column-start: 2;
  grid-column-end: 4;
}
.two {
  grid-row-start: 4;
  grid-column-start: 6;
}
```

如果省略结束栅格线,那么结束栅格线使用序列中的下一条栅格线.

还可以使用 `span n` 相对于 开始/结束 栅格线横跨 `n` 条栅格线, `n` 为正整数, 默认值 1. 还可以在 `n` 前/后指定命名栅格线, 以横跨`n`条特定栅格线,  若未指定栅格线名字,则为任意栅格线

```css
.grid {
  display: grid;
  width: 50em;
  grid-template-rows: repeat(5, [R] 5em);
  grid-template-columns: repeat(5, [col-A] 5em [col-B] 5em) 2em;
}

.one {
  grid-row-start: R 2;
  grid-row-end: 5;
  grid-column-start: col-B;
  grid-column-end: span 2;
}
.two {
  grid-row-start: R;
  grid-row-end: span R 2;
  grid-column-start: col-A 3;
  grid-column-end: span 2 col-A
}
```

### 行和列的简写属性

使用`grid-row`, `grid-column`简化把元素附加到栅格线上的过程.

```css
#grid {
  dispaly: grid;
  grid-template-rows: repeat(10, [R] 1.5em);
  grid-template-columns: 2em repeat(5, [col-A] 5em [col-B] 5em) 2em;
}

.one {
  grid-row: R 3 / 7;
  grid-column: col-B / span 2;
}
.two {
  grid-row: R / span R 2;
  grid-column: col-A 3 / span 2 col-A;
}
```

如果值中间没有斜线,那么定义的是开始栅格线,结束栅格线取决于开始栅格线的值.如果开始栅格线是用名称引用的,那么结束栅格线也使用那个名称引用.

### 隐式栅格

当栅格元素(或其一部分)超出了显式定义的栅格, 浏览器或隐式的创建栅格线以满足定义.

注意: 跨度是从显式栅格开始计数的. (注意看box4/5)

```css
#grid {
  display: grid;
  grid-template-rows: 2em 2em;
  grid-template-columns: repeat(6, 4em);
}

.box01 {
  grid-column: 1;
  grid-row: 1 / 4;
}
.box02 {
  grid-column: 2;
  grid-row: 3 / span 2;
}
.box03 {
  grid-column: 3;
  grid-row: span 2 / 3;
}
.box04 {
  grid-column: 4;
  grid-row: span 2 / 5;
}
.box05 {
  grid-column: 5;
  grid-row: span 4 / 5;
}

.box06 {
  grid-column: 6;
  grid-row: -1 / span 3;
}
.box07 {
  grid-column: 7;
  grid-row: span 3 / -1;
}
```

![image-20220716173940540](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20220716173940540.png)

### 错误处理

1. 开始线在结束线之后 => 对调两个值
2. 开始线与结束线都声明为跨度 => 丢弃结束线的值, 替换为 auto
3. 只使用具名跨度指明栅格元素的位置 => `span name`将被替换为`span 1`

### 使用区域

可以直接使用`grid-area`一个属性引用栅格区域

定义了具名栅格区域

```css
.grid {
  display: grid;
  grid-template-areas:
    "header header header header"
    "leftside content content rightside"
    "leftside footer footer footer"
}

.box01 {
  grid-area: header;
}
.box02 {
  grid-area: leftside;
}
.box03 {
  grid-area: content;
}
.box04 {
  grid-area: rightside;
}
.box05 {
  grid-area: footer;
}
```

![image-20220716180651012](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20220716180651012.png)

未定义栅格区域, 可以分别指定围成栅格区域的四条栅格线引用该栅格区域. 其顺序为row-start, column-start, row-end, column-end.

若存在省缺值, 且未省缺值为具名栅格线,那么省缺值与未省缺值一致.否则省缺值为auto

```css
.grid {
  display: grid;
  grid-template-rows: [r1-start] 1fr [r1-end r2-start] 2fr [r2-end];
  grid-template-columns: [col-start] 1fr [col-end main-start] 1fr [main-end];
}

.box01 {
  grid-area: r1 / main / r1 / main;
}
.box02 {
  grid-area: r2-start / col-start / r2-end / main-end;
}
.box03 {
  grid-area: 1 / 1 / 2 / 2;
}
```

![image-20220716182339313](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20220716182339313.png)

### 重叠栅格元素

附加栅格元素时,完全可以重叠每个元素

```css
.grid {
  display: grid;
  grid-template-rows: [r1-start] 1fr [r1-end r2-start] 2fr [r2-end];
  grid-template-columns: [col-start] 1fr [col-end main-start] 1fr [main-end];
}

.box01 {
  grid-area: r1 / col / r1 / main;
}
.box02 {
  grid-area: r1-start / main-start / r2-end / main-end;
}
```

![image-20220716182911866](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20220716182911866.png)

## 参考

[1\][Eric A. Meyer / Estelle Weyl. CSS权威指南（第四版）](https://book.douban.com/subject/33398314/)
