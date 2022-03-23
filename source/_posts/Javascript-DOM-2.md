---
title: Javascript-DOM-2
date: 2021-03-31 20:31:10
tags: [JS红宝书, DOM]
categories: [学习笔记, Javascript]
keywords:
description: 
photos: https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/dom-2.png
---

## DOM 编程

### 动态脚本

```javascript
let script = document.createElement("script");
script.src = "foo.js";
document.body.appendChild(script);
```

在上面最后一行把<script\>元素添加到页面之前，是不会开始下载外部文件的。

<!-- more -->

### 动态样式

```javascript
let link = document.createElement("link");
link.rel = "stylesheet";
link.type = "text/css";
link.href = "styles.css";
let head = document.getElementByTagName("head")[0];
head.appendChild(link);
```

通过外部文件加载样式是一个异步过程。因此，样式的加载和正执行的 Javascript 代码并没有先后顺序。一般来说也不需要知道样式什么时候加载完成

### 操作表格

可以使用类似前面的方法操作表格，但是涉及大量的标签，包括表行、表元、表题等。为了方便创建表格，HTMLDOM 给<table\>、<tbody\>和<tr\>元素添加了一些属性和方法。

<table\>元素添加了以下属性和方法

-   caption，指向<caption\>元素的指针(如果存在)
-   tBodies, 包含<tbody\>元素的 HTMLCollection
-   tFood，指向<tfoot\>元素(如果存在)
-   tHead，指向<tHead\>元素(如果存在)
-   rows，包含表示所有行的 HTMLCollection
-   createTHead(),创建<thead\>元素，放到表格中，返回引用
-   createTFoot(),创建<tfoot\>元素，放到表格中，返回引用
-   createCaption(),创建<caption\>元素，放到表格中，返回引用
-   deleteTHead(),删除<thead\>元素
-   deleteTFoot(),删除<tfoot\>元素
-   deleteTCaption(),删除<caption\>元素
-   deleteRow(pos),删除给定位置的行
-   insertRow(pos),在行集合中给定位置插入一行

<tbody\>元素添加了以下属性和方法

-   rows, 包含<tbody\>元素中所有行的 HTMLCOllection
-   deleteRow(pos), 删除给定位置的行
-   insertRow(pos), 在行集合中给定位置插入一行，返回该行的引用

<tr\>元素添加了以下属性和方法

-   cells, 包含<tr\>元素所有表元的 HTMLCollection
-   deleteCell(pos), 删除给定位置的表元
-   insertCell(pos), 在表元集合给定位置插入一个表元，返回该该表元的引用。

```javascript
// 创建表格
let table = document.createElement("table");
table.border = 1;
table.width = "100%";

// 创建表体
let tbody = document.createElement("tbody");
table.appendChild(tbody);

// 创建第一行
tbody.insertRow(0);
tbody.rows[0].insertCell(0);
tbody.rows[0].cells[0].appendChild(document.createTextNode("Cell 1,1"));
tbody.rows[0].insertCell(1);
tbody.rows[0].cells[1].appendChild(document.createTextNode("Cell 2,1"));

// 创建第二行
tbody.insertRow(1);
tbody.rows[1].insertCell(0);
tbody.rows[1].cells[0].appendChild(document.createTextNode("Cell 1,2"));
tbody.rows[1].insertCell(1);
tbody.rows[1].cells[1].appendChild(document.createTextNode("Cell 2,2"));

// 把表格添加到文档主体
document.body.appendChild(table);
```

### 使用 NodeList

理解 NodeList 对象和相关的 NameNodeMap、HTMLCollection，是理解 DOM 编程的关键。这 3 个集合类型都是"实时的"，意味着文档结构的变换会实时地在它们身上反映出来，因此它们的值始终代表最新的状态。

```javascript
let divs = document.getElementByTagName("div");

for (let i = divs.length - 1; i >= 0; --i) {
	let div = document.createElement("div");
	document.body.appendChild(div);
}
```

一般来说，最好限制操作 NodeList 的次数，因为每次查询都会搜索整个文档，所有最好把查询到的 NodeList 缓存起来。

## MutationObserver 接口

MutationObserver 接口，可以在 DOM 被修改时异步执行回调。使用 MutationObserver 可以观察整个文档，DOM 树的一部分，或某个元素。此外还可以观察元素属性、子节点、文本，或者前三者任意组合的变化。

### 基本用法

```javascript
let observer = new MutationObserver((MutationRecord) => console.log("<body> 属性发生改变", MutationRecord));
observer.observe(document.body, { attributes: true });
document.body.className = "foo";
console.log("改变 body 类");

// 改变 body 类
// <body> 属性发生改变 [MutationRecord]

// 调用 disconnect()方法提前终止回调
observer.disconnect();
document.body.className = "bar"; // 不会有日志输出
```

每个回调都会收到一个 MutationRecord 实例的数组。MutationRecord 实例包含的信息包括发生了什么变化，以及 DOM 的那一部分受到了影响。因为回调之前可能同时发生多个满足观察条件的事件，所以每次执行回调都会传入一个包含按顺序入队的 MutationRecord 实例的数组。

### 后续

略 🤣
