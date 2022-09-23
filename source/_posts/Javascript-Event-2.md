---
title: Javascript-Event-2
date: 2021-04-06 23:13:39
categories: [学习笔记, Javascript]
tags: [JS红宝书, Event]
keywords:
description:
photos:
---

## 事件类型

DOM3 Events 定义了如下事件类型

-   用户界面事件(UIEvent): 涉及与 BOM 交互德尔通用浏览器事件
-   焦点事件(FocusEvent): 在元素获得和失去焦点时触发
-   鼠标事件(MouseEvent): 使用鼠标在页面上执行某些操作是触发
-   滚轮事件(WheelEvent)：使用鼠标滚轮(或类似设备)时触发
-   输入事件(InputEvent): 在文档中输入文本时触发
-   键盘事件(KeyboardEvent): 使用键盘在页面上执行某些操作时触发
-   合成事件(CompositionEvent): 在使用某种 IME(Input Method Editor, 输入法编辑器)输入字符时触发

<!--more-->

### 用户界面事件

-   load：在 window 上当页面加载完成后触发，在窗套(<frameset\>)上所有窗格(<frame\>)都加载完成后触发
-   unload：在 window 上当页面完全卸载后触发，在窗套(<frameset\>)上当所有窗格(<frame\>)都卸载完成后触发，在<object\>元素上当相应对象卸载完成后触发
-   abort：在<object\>元素上当相应对象加载完成前被用户提前终止下载时触发
-   error：在 window 上当 Javascript 报错时触发，在<img\>元素上当无法加载指定图片时触发，在<object\>元素上当无法加载相应对象时触发，在窗套(<frameset\>)上当一个或多个窗格(<frame\>)无法加载时触发
-   select：在文本框(<input\>或 textarea)上当用户选择了一个或多个字符时触发
-   resize：在 window 或窗格上当窗口或窗格被缩放是触发
-   scroll：当用户滚动包含滚动条的元素时在元素上触发。<body\>元素包含已加载页面的滚动条
-   DOMActivate：元素被用户通过鼠标或键盘操作激活时触发(DOM3 Event 已废弃)

### 焦点事件

焦点事件在页面元素获得或失去焦点时触发。这些事件可以与 document.hasFocus()和 document.activeElement 一起为开发者提供用户在页面中的导航信息。

-   blur：当元素失去焦点时触发。这个事件不冒泡，所有浏览器都支持
-   focus：当元素获得焦点时触发。这个事件不冒泡，所有浏览器都支持
-   focusout：当元素失去焦点时触发。这个事件是 blur 的通用版
-   focusin：当元素获得焦点时触发。这个事件是 focus 的冒泡版
-   DOMFocusOut：当元素失去焦点时触发。这个事件是 blur 的的通用版。Opera 是唯一支持这个事件的主流浏览器。DOM3 Events 废弃了 DOMFocusIn，推荐 focusout。
-   DOMFocusIn：当元素获得焦点时触发。这个事件是 focus 的冒泡版。Opera 是唯一支持这个事件的主流浏览器。DOM3 Events 废弃了 DOMFocusIn，推荐 focusin。

当焦点从页面中的一个元素移到另一个元素上时，会依次发生如下事件。

1. focusout 在失去焦点的元素上触发
2. focusin 在获得焦点的元素上触发
3. blur 在失去焦点的元素上触发
4. DOMFocusOut 在失去焦点的元素上触发
5. focus 在获得焦点的元素上触发
6. DOMFocusIn 在获得焦点的元素上触发

blur、DOMFocusOut 和 focusout 的事件目标是失去焦点的元素，而 focus、DOMFocusIn 和 focusin 的事件目标是获得焦点的元素

### 鼠标和滚轮事件

-   click：在用户单击鼠标主键(通常是左键)或按键盘回车键时触发。这主要是基于无障碍的考虑，让键盘和鼠标都可以触发 onclick 事件处理程序
-   dbclick：在用户双击鼠标主键(通常是左键)时触发
-   mousedown：在用户按下任意鼠标键时触发。这个事件不能通过键盘触发
-   mouseenter：在用户把鼠标光标从元素外部移到元素内部时触发。这个事件不冒泡，也不会在光标经过后代元素时触发
-   mouseleave：在用户把鼠标光标从元素内部移到元素外部时触发。这个事件不冒泡，也不会在光标经过后代元素时触发。
-   mousemove：在鼠标光标在元素上移到时反复触发。这个事件不能通过键盘触发。
-   mouseout：在用户把光标从一个元素移动到另一个元素上时触发。移到到的元素可以是原始元素的外部元素，也可以是原始元素的子元素。这个事件不能通过键盘触发
-   mouseover：在用户把鼠标光标从元素外部移到元素内部时触发。这个事件不能通过键盘触发
-   mouseup：在用户释放鼠标键时触发。这个事件不能通过键盘触发。

mousedown、mouseup、click、dbclick 四个事件永远会按照如下顺序触发

1. mousedown
2. mouseup
3. click
4. mousedown
5. mouseup
6. click
7. dbclick

使用 event.shiftKey/ctrlKey/altKey/metaKey 可以检测出用户是否按下了某个修饰键。
对于 mouseover 和 mouseout 事件而言，还存在与事件相关的其他元素。对 mouseover 事件来说，事件的主要目标是获得光标的元素，相关元素是失去光标的元素。mouseout 则相反，DOM 通过 event 对象的 relatedTarget 属性提供了相关元素的信息。这个属性只有在 mouseover 和 mouseout 事件发生时才有包含值，其他所有事件的这个属性的值都是 null。

### 键盘和输入事件

键盘事件是用户操作键盘时触发的

-   keydown：用户按下键盘某个键时触发，而且持续按住会重复触发
-   keypress，用户按下键盘上某个键并产生字符时触发，而且持续按住会重发触发。Esc 键也会触发这个事件。DOM3 Events 废弃了 keypress 事件，而推荐 textInput 事件。
-   keyup，用户释放键盘上某个键时触发

输入事件只有一个，即 textInput。这个事件是对 keypress 事件的扩展，用于在文本显示给用户之前更方便地截取文本输入。textInput 会在文本被插入到文本框之前触发。

### 合成事件

合成事件是 DOM3 Events 中新增的，用于处理通常使用 IME 输入时的复杂输入序列。IME 可以让用户输入物理键盘上没有的字符。例如，使用拉丁字母键盘的用户还可以使用 IME 输入日文。IME 通常需要同时按下多个键才能输入一个字符。合成事件用于监测和控制这种输入。

-   compositionstart: 在 IME 的文本合成系统打开时触发，表示输入即将开始
-   compositionupdate： 在新字符插入输入字段时触发
-   compositionend：在 IME 的文本合成系统关时触发，表示恢复正常键盘输入。

合成事件在很多方面与输入事件很类似。在合成事件触发时，事件目标是接收文本的输入字段。唯一增加的事件属性是 data，其中包含的值视情况而异：

-   compositionstart 事件中，包含正在编辑的文本(例如，已经选择了文本但还没替换)
-   compositionupdate 事件中，包含要插入的新字符
-   compositionend 事件中，包含本次合成过程中输入的全部内容

### HTML5 事件

#### contextmenu 事件

contextmenu 事件专门用于表示何时该显示上下文菜单，从而允许开发者取消默认的上下文菜单并提供自定义菜单。contextmenu 事件冒泡，因此只要给 document 指定一个事件处理程序就可以处理页面上的所用同类事件。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>事件冒泡示例</title>
  </head>
  <body>
    <div id="myDiv">此处右键获取特有上下文菜单，别处右键获取默认上下文菜单</div>
    <ul id="myMenu" style="position: absolute;visibility: hidden;background-color: silver;">
      <li>菜单1</li>
      <li>菜单2</li>
      <li>菜单3</li>
    </ul>
    <script>
      window.addEventListener("load", (event) => {
        let div = document.getElementById("myDiv");

        div.addEventListener("contextmenu", (event) => {
          event.preventDefault();
          let menu = document.getElementById("myMenu");
          menu.style.left = event.clientX + "px";
          menu.style.top = event.clientY + "px";
          menu.style.visibility = "visible";
        });

        document.addEventListener("click", (event) => {
          document.getElementById("myMenu").style.visibility = "hidden";
        });
      });
    </script>
  </body>
</html>
```

#### beforeunload 事件

beforeunload 事件会在 window 上触发，用意是给开发者提供阻止页面被卸载的机会。这个事件会在页面即将从浏览器中卸载时触发，如果页面要继续使用，则可以不被卸载。这个事件不能取消，否则就意味着可以把用户永久阻拦在一个页面上。相反，这个事件会向用户显示一个确认框，其中的消息表明浏览器即将卸载页面，并请用户确认是希望关闭页面，还是继续留在页面上

```javascript
window.addEventListener("beforeunload", (event) => {
  let message = "我会想你的！🔈🔉🔊";
  event.returnValue = message;
  return message;
});

// 在chrome这样子写上并不能显示message的信息(应该是限制了)，但确实阻止了页面卸载(会弹出确认框)
```

#### DOMContentLoaded 事件

window 的 load 事件会在页面完全加载后触发，因为要等待很多外部资源加载完成，所以会花费较长时间。而 DOMContentLoaded 事件会在 DOM 树构建完成后立即触发，而不用等待图片、Javascript 文件、CSS 文件或者其他资源加载完成。相对于 load 事件，DOMContentLoaded 可以让开发者在外部资源下载的同时就能指定事件处理程序，从而让用户能够更快地与界面交互。

#### readystatechange 事件

支持 readystatechange 事件的每个对象都有一个 readyState 属性，该属性具有一个以下列出的可能的字符串值

-   uninitialized：对象存在并尚未初始化
-   loading：对象正在加载数据
-   loaded：对象已经加载完数据
-   interactive：对象可以交互，但尚未加载完成
-   complete：对象加载完成

```javascript
document.addEventListener("readystatechange", (event) => {
  if (document.readyState == "interactive") {
    console.log("Content loaded");
  }
});
```

#### hashchange 事件

HTML5 增加了 hashchange 事件，用于在 URL 散列值(URL 最后#后面的部分)发生变化时通知开发者。onhashchange 事件处理程序必须添加给 window，每次 URL 散列值发生变化时会调用它。

### 设备事件

暂略

## 模拟事件

### DOM 事件模拟

任何时候，都可以使用 document.creatEvent()方法创建一个 event 事件。这个方法接收一个参数，此参数是一个表示要创建事件类型的字符串。
可用的字符串值是以下值之一

-   "UIEvents"(DOM3 中是"UIEvent"): 通用用户界面事件(鼠标事件和键盘事件都继承自这个事件)
-   "MouseEvents"(DOM3 中是"MouseEvent"): 通用鼠标事件
-   "HTMLEvents"(DOM3 中没有): 通用 HTML 事件(HTML 事件已经分散到了其他事件大类中)

创建 event 事件之后，需要使用事件相关的信息来初始化。每种类型的 event 对象都有特定的方法，可以使用相应数据来完成初始化。方法的名字并不相同，这取决于调用 createEvent()时传入的参数。
事件模拟的最后一步是触发事件。为此要使用 dispatchEvent()方法，这个方法存在于所有支持事件的 DOM 节点之上。dispatchEvent()方法接收一个参数，即表示要触发事件的 event 对象。调用 dispatchEvent()方法之后，事件就"转正"了,接着便冒泡并触发事件处理程序执行。

```javascript
// 模拟鼠标事件
let btn = document.getElementById("myBtn");
let event = document.createEvent("MouseEvents");

// 初始化event对象(参数含义暂略，需要时再查)
event.initMouseEvent("click", true, true, document.defaulyView, 0, 0, 0, 0, 0, false, false, false, false, 0, null);

// 触发事件
btn.dispatchEvent(event);

// 模拟键盘事件
let textbox = document.getElementById("myTextbox"),
  event;

// 按照DOM3 的方式创建event对象
if (document.implementation.hasFeature("KeyboardEvents", "3.0")) {
  event = document.createEvent("KeyboardEvent");

  // 初始化event对象(参数含义暂略，需要时再查)
  event.initKeyboardEvent("keydown", true, true, document.defaulyView, "a", 0, "Shift", 0);
}
// 触发事件
textbox.dispatchEvent(event);
```
## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)