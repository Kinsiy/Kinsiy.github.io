---
title: Javascript-BOM-1
date: 2021-04-18 11:03:39
categories: [学习笔记, Javascript]
tags: [JS红宝书, BOM]
keywords:
description: 浏览器对象模型BOM，是使用Javascript开发Web应用程序的核心。本文对window对象及窗口关系进行学习
photos:
---

## window 对象

BOM 的核心是 window 对象，表示浏览器的实例。window 对象在浏览器中有两重身份，一个是 ECMAscript 中的 Global 对象，另一个是就是浏览器窗口的 Javascript 接口。这意味着网页中定义的所用对象、变量和函数都以 window 作为其 Global 对象，都可以访问其上定义的 parseInt()等全局方法

### Global 作用域

因为 window 对象被复用为 ECMAScript 的 Global 对象，所有通过 var 声明的所有全局变量和函数都会变成 window 对象的属性和方法

```javascript
// 全局 var 声明
var age = 23;
var sayAge = () => console.log(this.age);

sayAge(); // 23
window.sayAge(); // 23

// 全局 let 声明
let age_1 = 24;
const sayAge_1 = () => console.log(this.age_1);

sayAge_1(); // undefined
window.sayAge_1(); // Uncaught TypeError: window.sayAge_1 is not a function

// 访问未声明的变量会抛出错误，但是可以在window对象上查询是否存在未声明的变量

var newValue = oldValue; // Uncaught ReferenceError: oldValue is not defined

var anotherNewValue = window.anotherOldValue;
console.log(anotherNewValue); // undefined
```

## 窗口关系

top 对象始终指向最上层(最外层)窗口，即浏览器窗口本身。而 parent 对象则始终指向当前窗口的父窗口。如果当前窗口是最上层窗口，则 parent 等于 top(都等于 window)。最上层的 window 如果不是通过 window.open()打开的，那么其 name 属性就不会包含值。
还有一个 self 对象，它是终极 window 对象，始终会指向 window。实际上，self 和 window 就是同一个对象。之所以还要暴露 self，就是为了和 top、paren 保持一致。

### 窗口位置与像素比

window 对象的位置可以通过不同的属性和方法来确定。现代浏览器提供了 screenLeft 和 screenTop 属性，用于表示窗口相对于屏幕左侧和顶部的位置，返回值的单位事 CSS 像素。可以使用 moveTo()和 moveBy()方法移动窗口。这两个方法都接收两个参数，其中 moveTo()接收要移动到的新位置的绝对坐标 x 和 y;而 moveBy()则接收相对当前位置在两个方向上移动的像素数。

```javascript
// 依浏览器而定。以下方法可能会被部分或全部禁用

// 把窗口移到左上角
window.moveTo(0, 0);

// 把窗口向下移动100个像素
window.moveBy(0, 100);
```

### 窗口大小

在不同浏览器中确定浏览器窗口大小没有想象中那么容易。所有现代浏览器都支持 4 个属性：innerWidth、innerHeight、outerWidth 和 outerHeight、outerWidth 和 outerHeight 返回浏览器窗口自身的大小(不管是在最外层 window 上使用还是在窗格<frame\>中使用)。innerWidth 和 innerHeight 返回浏览器窗口中页面视口的大小(不包含浏览器边框和工具栏)
document.documentElement.clientWidth 和 document.documentElement.clientHeight 返回页面视口的宽度和高度。

```javascript
let pageWidth = window.innerWidth,
  pageHeight = window.innerHeight;

if (typeof pageWidth != "number") {
  if (document.compatMode == "CSS1Compat") {
    pageWidth = document.documentElement.clientWidth;
    pageHeight = document.documentElement.clientHeight;
  } else {
    pageWidth = document.body.clientWidth;
    pageHeight = document.body.clientHeight;
  }
}

console.log(pageWidth, " ", pageHeight); // 690 " " 875
```

可以使用 resizeTo()和 resizeBy()方法调整窗口大小。这两个方法都接收两个参数，resizeTo()接收新的宽度和高度值，而 resizeBy()接受宽度和高度各要缩放多少。与移动窗口的方法一样，缩放窗口的方法可能会被浏览器禁用，而且在某些浏览器中默认是禁用的。同样，缩放窗口的方法只能应用到最上层的 window 对象。

### 视口位置

浏览器窗口尺寸通常无法满足完整显示整个页面，为此用户可以通过滚动在有限的视口中查看文档。度量文档相对于视口滚动距离的属性有两对，返回相等的值：window.pageXoffset/window.scrollX 和 window.pageYoffset/window.scrollY。
可以使用 scroll()、scrollTo()和 scrollBy()方法滚动页面。这 3 个方法都接收表示相对<b>视口距离</b>的 x 和 y 坐标，这两个参数在前两个方法中表示要滚动到的坐标，在最后一个方法中表示滚动的距离。

```javascript
// 相当于当前视口向下滚动100像素
window.scrollBy(0, 100);

// 相当于当前视口向右滚动40像素
window.scrollBy(40, 0);

// 滚动到页面左上角
window.scrollTo(0, 0);

// 滚动到距离屏幕左边及顶边各100像素的位置
window.scrollTo(100, 100);

// 正常滚动
window.scrollTo({
  left: 100,
  top: 100,
  behavior: "auto",
});

// 平滑滚动
window.scrollTo({
  left: 100,
  top: 100,
  behavior: "smooth",
});
```

### 导航与打开新窗口

window.open()方法可以用于导航到指定 URL，也可以用于打开新浏览器窗口。这个方法接收 4 个参数：要加载的 URL、目标窗口、特性字符串和表示新窗口在浏览器历史中是否替代当前加载页面的布尔值。通常，调用这个方法时只传前 3 个参数，最后一个参数只有在不打开新窗口时才会使用。
如果 window.open()的第二个参数是一个已经存在的窗口或窗格(frame)的名字，则会在对应窗口或窗格中打开 URL。

#### 弹出窗口

如果 window.open()的第二个参数不是已有窗口，则会打开一个新窗口或标签页。第三个参数，即特性字符串，用于指定新窗口的配置。如果没有传第三个参数，则新窗口(或标签页)会带有所有默认的浏览器特性(工具栏、地址栏、状态栏等都是默认配置)。如果打开的不是新窗口，则忽略第三个参数。
特性字符串是一个逗号分隔的设置字符串，用于指定新窗口包含的特性。下表列出了一些选项

| 设置       |   值   | 说明                                                                                               |
| :--------- | :----: | :------------------------------------------------------------------------------------------------- |
| fullscreen | yes/no | 表示窗口是否最大化。仅限 IE 支持                                                                   |
| height     |  数值  | 新窗口高度。这个值不能小于 100                                                                     |
| left       |  数值  | 新窗口的 x 轴坐标。这个值不能是负值                                                                |
| location   | yes/no | 表示是否显示地址栏。不同浏览器的默认值也不一样。在设置为"no"时，地址栏可能隐藏或禁用(取决于浏览器) |
| Menubar    | yes/no | 表示是否显示菜单栏。默认为"no"                                                                     |
| resizeable | yes/no | 表示是否可以拖动改变新窗口大小。默认为"no"                                                         |
| scrollbars | yes/no | 表示是否可以在内容过长时滚动。默认为"no"                                                           |
| status     | yes/no | 表示是否显示状态栏。不同浏览器的默认值不一样                                                       |
| toolbar    | yes/no | 表示是否显示工具栏。默认为"no"                                                                     |
| top        |  数值  | 新窗口的 y 轴坐标。这个值不能是负值                                                                |
| width      |  数值  | 新窗口的宽度。这个值不能小于 100                                                                   |

window.open()方法返回一个对新建窗口的引用。这个对象与普通 window 对象没有区别。只是为了控制新窗口提供了方便。例如某些浏览器默认不允许缩放或移动主窗口，但可能允许缩放或移动通过 window.open()创建的窗口。跟使用任何 window 对象一样，可以使用这个对象操纵新打开的窗口。
还可以使用 close()方法来关闭新打开的窗口。<span style="text-decoration: line-through;">这个方法只能用于 window.open()创建的弹出窗口。</span>在 chrome 控制台试了下，可以直接关闭。
新创建窗口的 window 对象有一个属性 opener，指向打开它的窗口。这个属性只在弹出窗口的最上层 window 对象(top)有定义，是指向调用 window.open()打开它的窗口或窗格的指针。

#### 安全限制

从 IE7 开始，地址栏不能隐藏，而且弹窗默认是不能移动或缩放的。Firefox 1 禁用了隐藏状态栏的功能，因此无论 window.open()的特性字符串是什么，都不会隐藏弹窗的状态栏。Firefox 3 强制弹窗始终显示地址栏。Opera 只会在主窗口中打开新窗口，但不允许他们出现在系统对话框的位置。此外，浏览器会在用户操作下才允许创建弹窗。

#### 弹窗屏蔽程序

所有现代浏览器都内置了屏蔽弹窗的程序，因此大多数意料之外的弹窗都会被屏蔽。在浏览器屏蔽弹窗时，可能会发生一些事。如果浏览器内置的弹窗屏蔽程序组织了弹窗，那么 window.open()很可能会返回 null。在浏览器扩展或其他程序屏蔽弹窗时，window.open()通常会抛出错误。要准确检测弹窗是否被屏蔽，除了检测 window.open()的返回值，还有把它用 try/catch 包装起来

```javascript
let blocked = false;

try {
  let baiduWin = window.open("http://www.baidu.com", "_blank");
  if (baiduWin == null) {
    blocked = true;
  }
} catch (ex) {
  blocked = true;
}
if (blocked) {
  console.log("弹窗被屏蔽了");
}
```

### 定时器

Javascript 在浏览器中是单线程执行的，但允许使用定时器指定在某个时间之后或每隔一段时间就执行相应的代码。setTimeout()用于指定在一定时间后执行某些代码，而 setInterval()用于指定每隔一段时间执行某些代码。
setTimeOut()方法通常接受两个参数：要执行的代码和在执行回调函数前等待的时间(ms)。第一个参数可以是包含 Javascript 代买的字符串(类似于传给 eval()的字符串)或者一个函数，第二个参数是要等待的毫秒数。
setTimeout()的第二个参数只是告诉 Javascript 引擎在指定的毫秒数过后把任务添加到这个队列。如果队列是空的，则会立即执行该代码。如果队列不是空的，则代码必须等待前面的任务执行完了才能执行。
调用 setTimeout()时，会返回一个表示该超时排期的数值 ID。这个超时 ID 是被排期执行代码的唯一标识符，可用于取消该任务。要取消等待中的排期任务，可以调用 clearTimeout()方法并传入超时 ID。

```javascript
let timeoutId = setTimeout(() => console.log("Kinsiy"), 1000);
clearTimeout(timeoutId);
```

setInterval()与 setTimeout()的使用方法类似，只不过指定的任务会每隔指定时间就执行一次，直到取消循环定时或者页面卸载。

```javascript
let num = 0, intervalId = null
let max = 10
 let incrementNumber = function(){
   num++
   // 如果达到最大值，则取消所有未执行的任务
   if(num == max){
     clearInterval(intervalId)
     console.log("Done")
   }
 }

 intervalId = setInterval(incrementNumber,500)

let num_1 = 0
let max_1 = 10
let incrementNumber_1 = function(){
  num_1++
  // 如果还没有达到最大值，再设置一个超时任务
  if(num_1 < max_1>){
    setTimeout(incrementNumber_1,500)
  }else{
    console.log("Done")
  }
}
setTimeout(incrementNumber_1,500)
```

#### 系统对话框

使用 alert()、confir()和 promt()方法，可以让浏览器调用系统对话框向用户显示消息。这些对话框与浏览器中显示的网页无关，而且也不包含 HTML。他们的外观由操作系统或者浏览器决定。无法使用 CSS 设置。此外，这些对话框都是同步的模态对话框，即在他们显示的时候，代码会停止执行，在它们消失以后，代码才会恢复执行。

alert()接收一个要显示给用户的字符串。与 console.log()可以接收任意数量的参数且能一次性打印这些参数不同，alert()只接收一个参数。调用 alert()时，传入的字符串会显示在一个系统对话框中。对话框只有一个 OK 按钮。如果传给 alert()的参数不是一个原始字符串，则会调用这个值得 ttoString()方法将其转换为字符串。

第二种对话框叫确认框。通过调用 confirm()来显示。确认框跟警告框类似，都会向用户显示消息。但不同之处在不，确认框有两个按钮："Cancel"和"OK"。用户通过单击不同的按钮表明接下来执行什么操作。要知道用户单击了 OK 按钮还是 Cancel 按钮，可以判断 confirm()方法的返回值：true 表示单击了 OK 按钮，false 表示单击了 Cancel()按钮或者单击某一角上的 X 图标关闭了确认框。

```javascript
if (confirm("Are you sure?")) {
  alert("I'm so glad you're sure!");
} else {
  alert("I'm sorry to hear you're not sure.");
}
```

最后一种对话框是提示框，通过调用 prompt()方法来显示。提示框的用途是提示用户输入信息。除了 OK 和 Cancel 按钮，提示框还会显示一个文本框，让用户输入内容。prompt()方法接收两个参数：要显示给用户的文本，以及文本框的默认值(可以是空字符串)。如果用户单击了 OK 按钮，则 prompt()会返回文本框中的值。如果用户单击了 Cancel 按钮，或者对话框被关闭，则 prompt()会返回 null。

```javascript
let result = prompt("What is your name?", "");

if (result != null) {
  alert("Welcome, " + result);
}
```

Javascript 还可以显示另外两种对话框：find()和 print()。这两种对话框都是异步显示。用户在浏览器菜单上选择"查找"和"打印"时显示的就是这两种对话框。通过在 window 对象上调用 find()和 print()可以显示它们。（chrome find()调不出来）

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
