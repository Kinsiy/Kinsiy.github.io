---
title: HTML-快速参考
date: 2020-12-01 00:25:37
description: 
categories: [学习笔记, HTML]
---

{% note danger %}

2020年 刚开始学习时的一个笔记, 非常简陋, 不具有任何参考价值, 参阅 [MDN. HTML元素参考](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element)

{% endnote %}

<!-- More -->

## HTML 链接

### target 属性

```html
<a name="mark_a">Mark_A</a> <a href="https://www.baidu.com" target="_blank">Baidu</a>
```

|    值    |               描述               |    值     |             描述             |
| :------: | :------------------------------: | :-------: | :--------------------------: |
| \_blank  |     在新窗口中打开被链接文档     |   \_top   |   在整个窗口打开被链接文档   |
|  \_self  | 默认，在相同框架中打开被链接文档 | framename | 在指定的框架中打开被链接文档 |
| \_parent |     在父框架中打开被链接文档     |

### 电子邮件

```html
<a href="#mark_a">跳转到Mark_A</a>
<br />
<a href="mailto:solo@gmail.com?cc=kinsiy@qq.com&bcc=someone@outlook.com&subject=Hello%20word&body=正文">发送邮件！</a>
```

| 值  | 描述 | 值  | 描述 |   值    | 描述 |  值  | 描述 |
| :-: | :--: | :-: | :--: | :-----: | :--: | :--: | :--: |
| cc  | 抄送 | bcc | 暗送 | subject | 主题 | body | 正文 |

### 图片链接

```html
<a href="https://kinsiy.github.io"><img src="URL" alt="kinsiy" /></a>
```

|  值   |                              描述                              |
| :---: | :------------------------------------------------------------: |
|  alt  |                          替换文本属性                          |
| align | bottom/middle/top 图像行内对齐方式 left/right 浮动图像到那一侧 |

### 图像映射

```html
<img src="girl.jpg" alt="girl" border="0" usemap="#grilmap" width="200" height="200" />
<map name="grilmap" id="grilmap">
  <area shape="circle" coords="100,150,10" href="https://www.baidu.com" alt="baidu" target="_blank" />
</map>
```

#### ismap

ismap 可将图片转换为图像映射。点击图片任意位置访问时在自动在 URL 中添加点击点坐标

```html
<a href="https://www.baidu.com"><img src="girl.jpg" width="250" height="100" ismap /></a>
```

## 表格

```html
<table frame="box" border="1">
  <tr>
    <th rowspan="2">标题1，横跨两行</th>
    <td>数据格</td>
  </tr>
  <tr>
    <td>数据格</td>
  </tr>
  <tr>
    <th colspan="2">标题2，横跨两列</th>
  </tr>
  <tr>
    <td>&nbsp;</td>
    <!-- 空格占位符 -->
    <td>数据格</td>
  </tr>
</table>
```

|     值      |     描述     |     值      |      描述      |
| :---------: | :----------: | :---------: | :------------: |
|   colspan   |  标题横跨列  |   rowspan   |   标题横跨行   |
| cellpadding |  单元格边距  | cellspacing |   单元格间距   |
|    align    | 单元格内对齐 |   border    | 单元格边框宽度 |

### frame

{% note danger %}

已被废弃, 参考[MDN. Table Element](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/table#%E5%B1%9E%E6%80%A7)

{% endnote %}

| 值  | 描述 |  值   |  描述  |  值   |  描述  |   值   |   描述   |   值   |   描述   |
| :-: | :--: | :---: | :----: | :---: | :----: | :----: | :------: | :----: | :------: |
| box | 四周 | above | 上边框 | below | 下边框 | hsides | 上下边框 | vsides | 左右边框 |

## 列表

### 有序列表

```html
<ol>
  <li>有序列表1</li>
  <li>有序列表2</li>
  <li>有序列表3</li>
  <li>有序列表4</li>
</ol>
```

### 无序列表

```html
<ul>
  <li>无序列表1</li>
  <li>无序列表2</li>
  <li>无序列表3</li>
  <li>无序列表4</li>
</ul>
```

### 自定义列表

```html
<dl>
  <dt>Java</dt>
  <dd>Java是一种非常强大的语言</dd>
  <dt>Python</dt>
  <dd>Python是一种脚本语言</dd>
  <dt>Javascript</dt>
  <dd>Javascript也是一种脚本语言</dd>
</dl>
```

## 块/内联元素

大多数 HTML 元素被定义为块级元素或内联元素。{% label primary@“块级元素” %}译为 {% label info@block level element %}，{% label primary@“内联元素” %}译为 {% label info@inline element %}。块级元素在浏览器显示时，通常会以新行来开始（和结束）

### 块元素

```html
<h1>
  <p></p>
  <ul>
    <table>
      <!-- 块元素 -->
    </table>
  </ul>
</h1>
```

div元素是典型块级元素，它是可用于组合其他 HTML 元素的容器。<div\> 元素没有特定的含义。除此之外，由于它属于块级元素，浏览器会在其前后显示折行。如果与 CSS 一同使用，<div\> 元素可用于对大的内容块设置样式属性。

<div\> 元素的另一个常见的用途是文档布局。它取代了使用表格定义布局的老式方法。使用 <table\> 元素进行文档布局不是表格的正确用法。<table\> 元素的作用是显示表格化的数据。

### 内联元素

```html
<b>
  <td>
    <a>
      <img/><!-- img 比较特殊 严格来说它并不是内联元素, 但也不是块元素 -->
    </a>
  </td>
</b>
```

内联元素在显示时通常不会以新行开始。

<span\> 元素是典型内联元素，可用作文本的容器。<span\> 元素也没有特定的含义。当与 CSS 一同使用时，<span\> 元素可用于为部分文本设置样式属性。

## 表单

```html
<form action="#">
  <fieldset>
    <legend>Html边框测试</legend>
    <label for="sex">性别：</label>
    <input type="text" name="sex" />
    <br />
    <label for="name">姓名：</label>
    <input type="text" name="name" />
    <br />
    <label for="password">密码：</label>
    <input type="password" name="password" />
    <br />
    <input type="radio" value="male" />Male
    <br />
    <input type="radio" value="female" checked />Female
    <br />
    <input type="checkbox" value="1" checked />男生
    <br />
    <input type="checkbox" value="0" />女生
    <br />
    <input type="submit" />
    <br />
    <input type="reset" />
    <br />
    <input type="hidden" />
    <select name="cars" id="cars">
      <option value="1">大众</option>
      <optgroup label="豪车">
        <option value="2">宝马</option>
        <option value="3" selected>奔驰</option>
      </optgroup>
      <option value="4">东风</option>
    </select>
  </fieldset>
</form>
```

|    值    |          描述           |    值    |    描述    |
| :------: | :---------------------: | :------: | :--------: |
| fieldset |         定义域          | optgroup | 定义选项组 |
|  legend  | fieldset 元素定义标题。 |

## 框架

不能将 <body\></body\> 标签与 <frameset\></frameset\> 标签同时使用！不过，假如你添加包含一段文本的 <noframes\> 标签，就必须将这段文字嵌套于 <body\></body\> 标签内。

```html
<frameset rows="50%,50%">
  <frame src="frame_a.html" />
  <frameset cols="25%,75%">
    <frame src="frame_b.html" />
    <frame src="frame_c.html" />
  </frameset>
  <noframes>
    <body>
      您的浏览器无法处理框架！
    </body>
  </noframes>
</frameset>
<!-- 内联框架 -->
<iframe src="key.html" width="100%" height="30%" frameborder="0" name="iframe_a"></iframe>
```

## 格式化文档

### 文本格式化标签

|  标签  |   描述   | 标签  |   描述   |   标签    |   描述   |  标签  |  描述  |  标签  |  描述  |
| :----: | :------: | :---: | :------: | :-------: | :------: | :----: | :----: | :----: | :----: |
|  <b\>  | 粗体文字 | <em\> | 着重文字 | <small\>  |  小号字  | <sub\> | 下标字 | <ins\> | 插入字 |
| <big\> |  大号字  | <i\>  |  斜体字  | <strong\> | 加重语气 | <sup\> | 上标字 | <del\> | 删除字 |

### 计算机输出标签

|  标签   |    描述    |  标签  |  描述  |  标签   |      描述      |
| :-----: | :--------: | :----: | :----: | :-----: | :------------: |
| <code\> | 计算机代码 | <kbd\> | 键盘码 | <samp\> | 计算机代码样本 |
|  <tt\>  | 打字机代码 | <var\> |  变量  | <pre\>  |   预格式文本   |

### 引用与术语定义

|     标签      |  描述  |    标签    |    描述    |    标签    |    描述    |  标签  |   描述   |
| :-----------: | :----: | :--------: | :--------: | :--------: | :--------: | :----: | :------: |
|    <abbr\>    |  缩写  | <acronym\> | 首字母缩写 | <address\> |    地址    | <bdo\> | 文字方向 |
| <blockquote\> | 长引用 |    <q\>    |   短引用   |  <cite\>   | 引用、引证 | <dfn\> | 定义项目 |
