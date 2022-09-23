---
title: Javascript-Event-1
date: 2021-04-06 15:50:08
categories: [学习笔记, Javascript]
tags: [JS红宝书, Event]
keywords:
description:
photos:
---

## 事件流

事件流描述了页面接收事件的顺序。

### 事件冒泡

IE 事件流被称为事件冒泡，这是因为事件被定义为从最具体的元素(文档树中最深的节点)开始触发，然后向上传播指没有那么具体的元素(文档)。比如有如下 HTML 页面

<!-- more -->

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>事件冒泡示例</title>
  </head>
  <body>
    <div id="myDiv">Click Me</div>
  </body>
</html>
```

在点击页面中的<div\>元素后，click 事件会以如下顺序发生：

1. <div\>
2. <body\>
3. <html\>
4. document

所有现代浏览器都支持事件冒泡，只是在实现方式上会有一些变化。IE5.5 及早期版本会跳过<html\>元素。现代浏览器中的事件会一直冒泡到 window。

### 事件捕获

事件捕获的意思是最不具体的节点应该最先收到事件，而最具体的节点应该最后收到事件。事件捕获实际上是为了在事件到达最终目标前拦截事件。如果前面的例子使用时间捕获，则点击<div\>元素会以下列顺序触发 click 事件：

1. document
2. <html\>
3. <body\>
4. <div\>

由于旧版本浏览器不支持，因此实际当中几乎不会使用事件捕获。通常建议使用事件冒泡，特殊情况下可以使用事件捕获。

### DOM 事件流

DOM2 Event 规范规定事件流分为 3 个阶段：事件捕获、到达目标和事件冒泡。事件捕获是最先发生，为提前拦截事件提供了可能。然后实际的目标元素接收到事件。最后一个阶段是冒泡，最迟要在这个阶段响应事件。
![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/event.png)
在 DOM 事件流中，实际的目标(<div\>元素)在捕获阶段不会接收到事件。这是因为捕获阶段从 document 到<html\>再到<body\>就结束了。下一阶段，即会在<div\>元素上触发事件的“到达目标”阶段，通常在事件处理时被认为是冒泡阶段的一部分。然后冒泡阶段开始，事件反向传播至文档。
大多数支持 DOM 事件流的浏览器实现了一个小小的扩展。虽然 DOM2 Event 规范明确捕获阶段不命中事件目标，但现代浏览器都会在捕获阶段在事件目标上触发事件。最终结果是在事件目标上有两个机会来处理事件。

## 事件处理程序

为响应事件而调用的函数被称为事件处理程序(或事件监听器)。

### HTML 事件处理程序

特定元素支持的每个事件都可以使用事件处理程序的名字以 HTML 属性的形式来指定。此时属性的值必须是能够执行的 Javascript 代码。

```html
<!-- 不能在未经转义的情况下使用HTML语法字符 -->
<input type="button" value="Click Me" onclick="console.log('Clicked')" />

<!-- 作为事件处理程序执行的代码可以访问全局作用域中的一切 -->
<script>
  function showMessage() {
    console.log("Hello world!");
  }
</script>
<input type="button" value="Click Me" onclick="showMessage()" />

<!-- 以这种方式指定的事件处理程序有一些特殊的地方。首先，会创建一个函数来封装属性的值。这个函数有一个特殊的局部变量event，其中保存的就是event对象 。-->

<input type="button" value="Click Me" onclick="console.log(event.type)" />
<!-- click -->

<input type="button" value="Click Me" onclick="console.log(this.value)" />
<!-- Click Me -->

<!-- 这个函数的作用域链被扩展了。在这个函数中document和元素自身的成员都可以被当成局部变量来访问 -->
<input type="button" value="Click Me" onclick="console.log(value)" />

<!-- 如果这个元素是一个表单输入框，则作用域中还会包含表单元素 -->
<form method="post">
  <input type="text" name="username" value="" />
  <input type="button" value="Click Me" onclick="console.log(username.value)" />
</form>
<!-- 点击会显示出文本框中包含的文本。注意，这里直接引用了username -->
```

### DOM0 事件处理程序

在 Javascript 中指定事件处理程序的传统方式是把一个函数赋值给(DOM 元素的)一个事件处理程序属性。每个元素(包括 window 和 document)都有通常小写的事件处理程序属性，比如 onclick

```javascript
let btn = document.getElementById("myBtn");
btn.onclick = function () {
  console.log(this.id); // "myBtn"
};
/* 事件处理程序会在元素的作用域中运行，即this等于元素 */

btn.onclick = null; // 移除事件处理程序
```

以这种方式添加事件处理程序是注册在事件流的冒泡阶段的。

### DOM2 事件处理程序

DOM2 Events 为事件处理程序的赋值和移除定义了两个方法：addEventListener()和 removeEventListener()。这两个方法暴露在所有 DOM 节点上，它们接收 3 个参数：事件名、事件处理函数和一个布尔值，true 表示在捕获阶段调用事件处理程序，false(默认值)表示在冒泡阶段调用事件处理程序。

```javascript
let btn = document.getElementById("myBtn");
let handler = function () {
  console.log(this.id);
};
btn.addEventListener("click", handler, false);

/** 通过addEventListener()添加的事件处理程序只能使用removeEventListener()并传入与添加时同样的参数来移除。
 * 这意味着使用addEventListener()添加的匿名函数无法移除
 */

btn.removeEventListener("click", handler, false);
```

### IE 事件处理程序

IE 实现了与 DOM 类似的方法，即 attachEvent()和 detachEvent()。这两个方法接收两个同样的参数：事件处理程序的名字和事件处理函数。因为 IE8 及更早版本只支持事件冒泡，所以使用 attachEvent()添加的事件处理程序会添加到冒泡阶段。

```javascript
let btn = document.getElementById("myBtn");
let handler = function () {
  console.log(this.id);
};
btn.attachEvent("onclick", handler); // 这里第一个参数是 onclick 而不是 click

/** attachEvent()添加的事件处理程序只能使用detachEvent()并传入与添加时同样的参数来移除。
 * 这意味着使用attachEvent()添加的匿名函数无法移除
 */

btn.detachEvent("onclick", handler); // 这里第一个参数是 onclick 而不是 click
```

使用 DOM0 方式时，事件处理程序中的 this 值等于目标元素。而使用 attachEvent()时，事件处理程序是在全局作用域中运行的，因此 this 等于 window。另外，若给同一个元素添加了两个不同的事件处理程序，它们会以添加它们的顺序反向触发。

## 事件对象

在 DOM 中发生事件时，所有相关信息都会被收集并存储在一个名为 event 的对象中。这个对象包含了一些基本信息，比如导致事件的元素、发生的事件类型，以及可能与特定事件相关的任何其他数据

### DOM 事件对象

在 DOM 合规的浏览器中，event 对象是传给事件处理程序的唯一参数。不管以那种方式(DOM0 或 DOM2)指定事件处理程序，都会传入这个 event 对象。

```javascript
let btn = document.getElementById("myBtn");
let handler = function (event) {
  console.log(event.type);
};
btn.addEventListener("click", handler);

// click
```

所有事件对象都会包含下表列出的这些公共属性和方法

| 属性/方法                  |     类型     | 读/写 | 说明                                                                             |
| :------------------------- | :----------: | :---: | :------------------------------------------------------------------------------- |
| bubbles                    |    布尔值    | 只读  | 表示事件是否冒泡                                                                 |
| cancelable                 |    布尔值    | 只读  | 表示是否可以取消事件的默认行为                                                   |
| currentTarget              |     元素     | 只读  | 当前事件处理程序所在的元素                                                       |
| defaultPrevented           |    布尔值    | 只读  | true 表示已经调用 preventDefault()方法                                           |
| detail                     |     整数     | 只读  | 事件相关的其他信息                                                               |
| eventPhase                 |     整数     | 只读  | 表示调用事件处理程序的阶段：1-捕获，2-到达，3-冒泡                               |
| preventDefault()           |     函数     | 只读  | 事件相关的用于取消事件的默认行为。只有其他 cancelable 为 true 才可以调用这个方法 |
| stopImmediatePropagation() |     整数     | 只读  | 用于取消所有后续事件捕获或事件冒泡，并阻止调用任何后续事件处理程序               |
| stopPropagation()          |     整数     | 只读  | 用于取消所有后续事件捕获或事件冒泡，只有其他 bubbles 为 true 才可以调用这个方法  |
| target                     |     元素     | 只读  | 事件目标                                                                         |
| trusted                    |    布尔值    | 只读  | true 表示事件是由浏览器生成的。false 表示事件是开发者通过 Javascript 创建的      |
| type                       |    字符串    | 只读  | 被触发的事件类型                                                                 |
| view                       | AbstractView | 只读  | 与事件相关的抽象视图。等于事件所发生的 window 对象                               |

在事件处理程序内部，this 对象始终等于 currentTarget 的值，而 target 只包含事件的实际目标。(想一下事件委托就明白了)

### IE 事件对象

与 DOM 事件对象不同，IE 事件对象可以基于事件处理程序被指定的方式以不同方式来访问。如果事件处理程序是使用 DOM0 方式指定的，则 event 对象只是 window 对象的一个属性。如果事件处理程序是使用 attachEvent()指定的，则 event 对象会作为唯一的参数传给处理函数。

```javascript
let btn = document.getElementById("myBtn");

btn.onclick = function () {
  let event = window.event;
  console.log(event.type); // click
};

btn.attachEvent("onclick", function (event) {
  console.log(event.type); // click
});
```

所有的 IE 事件对象都会包含下表所列的公共属性和方法

| 属性/方法    |  类型  | 读/写 | 说明                                                                               |
| :----------- | :----: | :---: | :--------------------------------------------------------------------------------- |
| cancelBubble | 布尔值 | 读/写 | 默认 false，设置为 true 可以取消冒泡(与 DOM 的 stopPropagation()方法相同)          |
| returnValue  | 布尔值 | 读/写 | 默认 true，设置为 false 可以取消事件的默认行为(与 DOM 的 preventDefault()方法相同) |
| srcElement   |  元素  | 只读  | 事件目标(与 DOM 的 target 属性相同)                                                |
| type         | 字符串 | 只读  | 触发的事件类型                                                                     |

由于事件处理程序的作用域取决于指定它的方式，因此 this 值并不总是等于事件目标。为此，更好的方式是使用事件对象的 srcElement 属性代替 this。

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)