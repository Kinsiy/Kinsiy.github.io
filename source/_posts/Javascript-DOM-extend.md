---
title: Javascript - DOM扩展
date: 2021-10-21 21:33:28
tags: [JS红宝书, DOM]
categories: [学习笔记, Javascript]
keywords:
description: 尽管DOM API 已经相当不错，但仍然不断有标准或专有的扩展出现，已支持更多功能
photos:
---


尽管DOM API 已经相当不错，但仍然不断有标准或专有的扩展出现，已支持更多功能。


## Selector API


JavaScript 库中最流行的一种能力就是根据 CSS 选择符的模式匹配 DOM 元素。比如，jQuery 就完全 以 CSS 选择符查询 DOM 获取元素引用，而不是使用 getElementById()和 getElementsByTagName()。 


Selectors API（参见 W3C 网站上的 Selectors API Level 1）是 W3C 推荐标准，规定了浏览器原生支 持的 CSS 查询 API。支持这一特性的所有 JavaScript 库都会实现一个基本的 CSS 解析器，然后使用已有 的 DOM 方法搜索文档并匹配目标节点。虽然库开发者在不断改进其性能，但 JavaScript 代码能做到的 毕竟有限。通过浏览器原生支持这个 API，解析和遍历 DOM 树可以通过底层编译语言实现，性能也有 了数量级的提升。


 Selectors API Level 1 的核心是两个方法：querySelector()和 querySelectorAll()。在兼容浏 览器中，Document 类型和 Element 类型的实例上都会暴露这两个方法。 


Selectors API Level 2 规范在 Element 类型上新增了更多方法，比如 matches()、find()和 findAll()。不过，目前还没有浏览器实现或宣称实现 find()和 findAll()。


### querySelector()


querySelector()方法接收 CSS 选择符参数，返回匹配该模式的第一个后代元素，如果没有匹配 项则返回 null。下面是一些例子:


```javascript
// 取得<body>元素
let body = document.querySelector("body"); 
// 取得 ID 为"myDiv"的元素
let myDiv = document.querySelector("#myDiv"); 


// 取得类名为"selected"的第一个元素
let selected = document.querySelector(".selected"); 
// 取得类名为"button"的图片
let img = document.body.querySelector("img.button"); 
```


在 Document 上使用 querySelector()方法时，会从文档元素开始搜索；在 Element 上使用 querySelector()方法时，则只会从当前元素的后代中查询。


 用于查询模式的 CSS 选择符可繁可简，依需求而定。如果选择符有语法错误或碰到不支持的选择符， 则 querySelector()方法会抛出错误。


### querySelectorAll()


querySelectorAll()方法跟 querySelector()一样，也接收一个用于查询的参数，但它会返回 所有匹配的节点，而不止一个。**这个方法返回的是一个 NodeList 的静态实例**。


 再强调一次，querySelectorAll()返回的 NodeList 实例一个属性和方法都不缺，但它是一 个静态的“快照”，而非“实时”的查询。这样的底层实现避免了使用 NodeList 对象可能造成的性 能问题。 


以有效 CSS 选择符调用 querySelectorAll()都会返回 NodeList，无论匹配多少个元素都可以。 如果没有匹配项，则返回空的 NodeList 实例。 


与 querySelector()一样，querySelectorAll()也可以在 Document、DocumentFragment 和 Element 类型上使用。下面是几个例子：


```javascript
// 取得 ID 为"myDiv"的<div>元素中的所有<em>元素
let ems = document.getElementById("myDiv").querySelectorAll("em"); 
// 取得所有类名中包含"selected"的元素
let selecteds = document.querySelectorAll(".selected"); 
// 取得所有是<p>元素子元素的<strong>元素
let strongs = document.querySelectorAll("p strong"); 
```


返回的 NodeList 对象可以通过 for-of 循环、item()方法或中括号语法取得个别元素。比如：


```javascript
let strongElements = document.querySelectorAll("p strong"); 
// 以下 3 个循环的效果一样
for (let strong of strongElements) { 
 strong.className = "important"; 
} 
for (let i = 0; i < strongElements.length; ++i) { 
 strongElements.item(i).className = "important"; 
} 
for (let i = 0; i < strongElements.length; ++i) { 
 strongElements[i].className = "important"; 
} 
```


 querySelector()方法一样，如果选择符有语法错误或碰到不支持的选择符，则 querySelectorAll()方法会抛出错误。


### mathes()


matches()方法（在规范草案中称为 matchesSelector()）接收一个 CSS 选择符参数，如果元素 匹配则该选择符返回 true，否则返回 false。例如：


```javascript
if (document.body.matches("body.page1")){ 
 // true 
} 
```


使用这个方法可以方便地检测某个元素会不会被 querySelector()或 querySelectorAll()方 法返回。


## HTML5(影响DOM节点部分)


HTML5 代表着与以前的 HTML 截然不同的方向。在所有以前的 HTML 规范中，从未出现过描述 JavaScript 接口的情形，HTML 就是一个纯标记语言。JavaScript 绑定的事，一概交给 DOM 规范去定义。 然而，HTML5 规范却包含了与标记相关的大量 JavaScript API 定义。其中有的 API 与 DOM 重合， 定义了浏览器应该提供的 DOM 扩展。


### CSS类扩展


#### getElementsByClassName()


getElementsByClassName()是 HTML5 新增的最受欢迎的一个方法，暴露在 document 对象和 所有 HTML 元素上。这个方法脱胎于基于原有 DOM 特性实现该功能的 JavaScript 库，提供了性能高好 的原生实现。


getElementsByClassName()方法接收一个参数，即包含一个或多个类名的字符串，返回类名中 包含相应类的元素的 NodeList。如果提供了多个类名，则顺序无关紧要。下面是几个示例:


```javascript
// 取得所有类名中包含"username"和"current"元素
// 这两个类名的顺序无关紧要
let allCurrentUsernames = document.getElementsByClassName("username current"); 
// 取得 ID 为"myDiv"的元素子树中所有包含"selected"类的元素
let selected = document.getElementById("myDiv").getElementsByClassName("selected");
```


如果要给包含特定类（而不是特定 ID 或标签）的元素添加事件处理程序，使用这个方法会很方便。 不过要记住，因为返回值是 NodeList，所以使用这个方法会遇到跟使用 getElementsByTagName()和其他返回 NodeList 对象的 DOM 方法同样的问题。


[Javascript-DOM2#使用NodeList]: https://kinsiy.github.io/2021/03/31/Javascript-DOM-2/#kinsiy-5


#### classList 属性


要操作类名，可以通过 className 属性实现添加、删除和替换。但 className 是一个字符串， 所以每次操作之后都需要重新设置这个值才能生效，即使只改动了部分字符串也一样。以下面的 HTML 代码为例：


```html
<div class="bd user disabled">...</div> 
```


这个<div>元素有 3 个类名。要想删除其中一个，就得先把 className 拆开，删除不想要的那个， 再把包含剩余类的字符串设置回去。比如:


```javascript
// 要删除"user"类
let targetClass = "user"; 
// 把类名拆成数组
let classNames = div.className.split(/\s+/); 
// 找到要删除类名的索引
let idx = classNames.indexOf(targetClass); 
// 如果有则删除
if (idx > -1) { 
 classNames.splice(i,1); 
} 
// 重新设置类名
div.className = classNames.join(" "); 
```


HTML5 通过给所有元素增加 classList 属性为这些操作提供了更简单也更安全的实现方式。 classList 是一个新的集合类型 DOMTokenList 的实例。与其他 DOM 集合类型一样，DOMTokenList 也有 length 属性表示自己包含多少项，也可以通过 item()或中括号取得个别的元素。此外， DOMTokenList 还增加了以下方法。


- add(value)，向类名列表中添加指定的字符串值value。如果这个值已经存在，则什么也不做。
- contains(value)，返回布尔值，表示给定的value是否存在
- remove(value)，从类名列表中删除指定的字符串值value。
- toggle(value)，如果类名列表中已经存在指定的value，则删除；如果不存在，则添加。


```javascript
// 删除"disabled"类
div.classList.remove("disabled"); 


// 添加"current"类
div.classList.add("current"); 


// 切换"user"类
div.classList.toggle("user"); 


// 检测类名 
if (div.classList.contains("bd") && !div.classList.contains("disabled")){ 
 // 执行操作
) 
// 迭代类名
for (let class of div.classList){ 
 doStuff(class); 
} 
```


### 焦点管理


HTML5 增加了辅助 DOM 焦点管理的功能。首先是 document.activeElement，始终包含当前拥 有焦点的 DOM 元素。页面加载时，可以通过用户输入（按 Tab 键或代码中使用 focus()方法）让某个 元素自动获得焦点。例如：


```JavaScript
let button = document.getElementById("myButton"); 
button.focus(); 
console.log(document.activeElement === button); // true
```


默认情况下，document.activeElement 在页面刚加载完之后会设置为 document.body。而在 页面完全加载之前，document.activeElement 的值为 null。 


其次是 document.hasFocus()方法，该方法返回布尔值，表示文档是否拥有焦点：


```JavaScript
let button = document.getElementById("myButton"); 
button.focus(); 
console.log(document.hasFocus()); // true 
```


### HTMLDoucument扩展


#### readyState属性


readyState 是 IE4 最早添加到 document 对象上的属性，后来其他浏览器也都依葫芦画瓢地支持 这个属性。最终，HTML5 将这个属性写进了标准。document.readyState 属性有两个可能的值：


- loading，表示文档正在加载
- complete，表示文档加载完成


实际开发中，最好是把 document.readState 当成一个指示器，以判断文档是否加载完毕。在这 个属性得到广泛支持以前，通常要依赖 onload 事件处理程序设置一个标记，表示文档加载完了。这个 属性的基本用法如下： 


```JavaScript
if (document.readyState == "complete"){  // 执行操作 } 
```


#### compatMode 属性


自从 IE6 提供了以标准或混杂模式渲染页面的能力之后，检测页面渲染模式成为一个必要的需求。 IE 为 document 添加了 compatMode 属性，这个属性唯一的任务是指示浏览器当前处于什么渲染模式。 如下面的例子所示，标准模式下 document.compatMode 的值是"CSS1Compat"，而在混杂模式下， document.compatMode 的值是"BackCompat"：


```Javascript
if (document.compatMode == "CSS1Compat"){ 
 console.log("Standards mode"); 
} else { 
 console.log("Quirks mode"); 
} 
```


HTML5 最终也把 compatMode 属性的实现标准化了


#### head 属性


作为对 document.body（指向文档的元素）的补充，HTML5 增加了 document.head 属 性，指向文档的元素。可以像下面这样直接取得元素：


```javascript
let head = document.head
```


### 字符集属性


HTML5 增加了几个与文档字符集有关的新属性。其中，characterSet 属性表示文档实际使用的 字符集，也可以用来指定新字符集。这个属性的默认值是"UTF-16"，但可以通过元素或响应头， 以及新增的 characterSeet 属性来修改。下面是一个例子：


```javascript
console.log(document.characterSet); // "UTF-16" 
document.characterSet = "UTF-8"; 
```


### 自定义数据属性


HTML5 允许给元素指定非标准的属性，但要使用前缀 data-以便告诉浏览器，这些属性既不包含 与渲染有关的信息，也不包含元素的语义信息。除了前缀，自定义属性对命名是没有限制的，data-后 面跟什么都可以。


义了自定义数据属性后，可以通过元素的 dataset 属性来访问。dataset 属性是一个 DOMStringMap 的实例，包含一组键/值对映射。元素的每个 data-name 属性在 dataset 中都可以通 过 data-后面的字符串作为键来访问（例如，属性 data-myname、data-myName 可以通过 myname 访 问，但要注意 data-my-name、data-My-Name 要通过 myName 来访问）。下面是一个使用自定义数据 属性的例子：


```Javascript
// 本例中使用的方法仅用于示范
let div = document.getElementById("myDiv"); 
// 取得自定义数据属性的值
let appId = div.dataset.appId; 
let myName = div.dataset.myname; 
// 设置自定义数据属性的值
div.dataset.appId = 23456; 
div.dataset.myname = "Michael"; 
// 有"myname"吗？
if (div.dataset.myname){ 
 console.log(`Hello, ${div.dataset.myname}`); 
} 
```


### 插入标记


DOM 虽然已经为操纵节点提供了很多 API，但向文档中一次性插入大量 HTML 时还是比较麻烦。 相比先创建一堆节点，再把它们以正确的顺序连接起来，直接插入一个 HTML 字符串要简单（快速） 得多。HTML5 已经通过以下 DOM 扩展将这种能力标准化了


#### innerHTML


在读取 innerHTML 属性时，会返回元素所有后代的 HTML 字符串，包括元素、注释和文本节点。 而在写入 innerHTML 时，则会根据提供的字符串值以新的 DOM 子树替代元素中原来包含的所有节点。 比如下面的 HTML 代码：


```html
<div id="content"> 
 <p>This is a <strong>paragraph</strong> with a list following it.</p> 
 <ul> 
 <li>Item 1</li> 
 <li>Item 2</li> 
 <li>Item 3</li> 
 </ul> 
</div> 


<!-- 对于这里的<div>元素而言，其 innerHTML 属性会返回以下字符串：-->


<p>This is a <strong>paragraph</strong> with a list following it.</p> 
<ul> 
 <li>Item 1</li> 
 <li>Item 2</li> 
 <li>Item 3</li> 
</ul> 
```


实际返回的文本内容会因浏览器而不同。IE 和 Opera 会把所有元素标签转换为大写，而 Safari、 Chrome 和 Firefox 则会按照文档源代码的格式返回，包含空格和缩进。因此不要指望不同浏览器的 innerHTML 会返回完全一样的值


在写入模式下，赋给 innerHTML 属性的值会被解析为 DOM 子树，并替代元素之前的所有节点。 因为所赋的值默认为 HTML，所以其中的所有标签都会以浏览器处理 HTML 的方式转换为元素（同样， 转换结果也会因浏览器不同而不同）。如果赋值中不包含任何 HTML 标签，则直接生成一个文本节点


在所有现代浏览器中，通过 innerHTML 插入的<script>标签是不会执行的。


#### outerHTML


读取 outerHTML 属性时，会返回调用它的元素（及所有后代元素）的 HTML 字符串。在写入 outerHTML 属性时，调用它的元素会被传入的 HTML 字符串经解释之后生成的 DOM 子树取代。


```javascript
// HTML:
/*
<div id="container">
    <div id="d">This is a div.</div>
</div>
*/


let container = document.getElementById("container");
let d = document.getElementById("d");


console.log(container.firstChild.nodeName);
// logs "div"


d.outerHTML = "<p>This paragraph replaced the original div.</p>";


console.log(container.firstChild.nodeName);
// logs "P"


/*
    #d div不再是文档树的一部分，新段替换了它。
    (不在页面中显示,但仍然在内存中) 
*/
```


#### 内存与性能问题


使用本节介绍的方法替换子节点可能在浏览器（特别是 IE）中导致内存问题。比如，如果被移除的 子树元素中之前有关联的事件处理程序或其他 JavaScript 对象（作为元素的属性），那它们之间的绑定关 系会滞留在内存中。如果这种替换操作频繁发生，页面的内存占用就会持续攀升。在使用 innerHTML、 outerHTML 和 insertAdjacentHTML()之前，最好手动删除要被替换的元素上关联的事件处理程序和 JavaScript 对象。


 使用这些属性当然有其方便之处，特别是 innerHTML。一般来讲，插入大量的新 HTML 使用 innerHTML 比使用多次 DOM 操作创建节点再插入来得更便捷。这是因为 HTML 解析器会解析设置给 innerHTML（或 outerHTML）的值。解析器在浏览器中是底层代码（通常是 C++代码），比 JavaScript 快得多。不过，HTML 解析器的构建与解构也不是没有代价，因此最好限制使用 innerHTML 和 outerHTML 的次数


#### 跨站点脚本


尽管 innerHTML 不会执行自己创建的<script>标签，但仍然向恶意用户暴露了很大的攻击面，因为通过它可以毫不费力地创建元素并执行 onclick 之类的属性。


如果页面中要使用用户提供的信息，则不建议使用 innerHTML。与使用 innerHTML 获得的方便相
比，防止 XSS 攻击更让人头疼。此时一定要隔离要插入的数据，在插入页面前必须毫不犹豫地使用相关的库对它们进行转义。


### scrollIntoView()


DOM 规范中没有涉及的一个问题是如何滚动页面中的某个区域。为填充这方面的缺失，不同浏览器实现了不同的控制滚动的方式。在所有这些专有方法中，HTML5 选择了标准化 scrollIntoView()。


 scrollIntoView()方法存在于所有 HTML 元素上，可以滚动浏览器窗口或容器元素以便包含元 素进入视口。这个方法的参数如下：


- alignToTop 是一个布尔值
  - true: 窗口滚动后元素的顶部与视口顶部对齐
  - false：窗口滚动后元素的底部与视口底部对齐


- scrollIntoViewOption 是一个选项对象
  - behavior：定义过渡动画，可取的值为"smooth"和"auto"，默认为"auto"
  - block: 定义垂直方向的对齐，可取的值为"start"、"center"、"end"和"nearest"，默认为"start"
  - inline: 定义水平方向的对齐，可取的值为"start"、"center"、"end"和"nearest"，默认为"nearest"
- 不传参数等同于alignToTop为True


```JavaScript
// 确保元素可见
document.forms[0].scrollIntoView(); 
// 同上
document.forms[0].scrollIntoView(true); 
document.forms[0].scrollIntoView({block: 'start'}); 
// 尝试将元素平滑地滚入视口
document.forms[0].scrollIntoView({behavior: 'smooth', block: 'start'});
```


这个方法可以用来在页面上发生某个事件时引起用户关注。把焦点设置到一个元素上也会导致浏览 器将元素滚动到可见位置


## 专有扩展


### children 属性


children 属性是一个 HTMLCollection，只包含元素的 Element 类型的子节点。如果元素的子节点类型全部是元素类型，那 children 和 childNodes 中包含的节点应该是一样的


```javascript
let childCount = element.children.length; 
let firstChild = element.children[0]; 
```


### contains() 方法


DOM 编程中经常需要确定一个元素是不是另一个元素的后代。IE 首先引入了 contains()方法， 让开发者可以在不遍历 DOM 的情况下获取这个信息。contains()方法应该在要搜索的祖先元素上调 用，参数是待确定的目标节点


如果目标节点是被搜索节点的后代，contains()返回 true，否则返回 false另外，使用 DOM Level 3 的 compareDocumentPosition()方法也可以确定节点间的关系。这个 方法会返回表示两个节点关系的位掩码


| 掩码 | 节点关系                                  |
| ---- | ----------------------------------------- |
| 0x1  | 断开(传入的节点不在文档中)                |
| 0x2  | 领先(传入的节点在DOM树中位于参考节点之前) |
| 0x4  | 落后(传入的节点在DOM树中位于参考节点之后) |
| 0x8  | 包含(传入的节点是参考节点的祖先)          |
| 0x10 | 被包含(传入的节点是参考节点的后代)        |






```JavaScript
console.log(document.documentElement.contains(document.body)); // true 


let result = document.documentElement.compareDocumentPosition(document.body); 
console.log(!!(result & 0x10)); 


/** result 的值为 20（或 0x14，其中 0x4 表示“随后”，加上 0x10“被包含”）。
 * 对result 和 0x10 应用按位与会返回非零值，而两个叹号将这个值转换成对应的布尔值。 
 */
```


### 插入标记


HTML5 将 IE 发明的 innerHTML 和 outerHTML 纳入了标准，但还有两个属性没有入选。这两个剩 下的属性是 innerText 和 outerText。


#### innerText 属性


innerText 属性对应元素中包含的所有文本内容，无论文本在子树中哪个层级。在用于读取值时， innerText 会按照深度优先的顺序将子树中所有文本节点的值拼接起来。在用于写入值时，innerText 会移除元素的所有后代并插入一个包含该值的文本节点。


因为设置 innerText 只能在容器元素中生成一个文本节点，所以为了保证一定是文本节点，就必 须进行 HTML 编码。innerText 属性可以用于去除 HTML 标签。通过将 innerText 设置为等于 innerText，可以去除所有 HTML 标签而只剩文本，如下所示： 


```javascript
div.innerText = div.innerText; 
```


执行以上代码后，容器元素的内容只会包含原先的文本内容。


#### outerText 属性


outerText 与 innerText 是类似的，只不过作用范围包含调用它的节点。要读取文本值时， outerText 与 innerText 实际上会返回同样的内容。但在写入文本值时，outerText 就大不相同了。 写入文本值时，outerText 不止会移除所有后代节点，而是会替换整个元素。


本质上，这相当于用新的文本节点替代 outerText 所在的元素。此时，原来的元素会与文档脱离 关系，因此也无法访问。 


outerText 是一个非标准的属性，而且也没有被标准化的前景。因此，不推荐依赖这个属性实现 重要的操作。除 Firefox 之外所有主流浏览器都支持 outerText


### 滚动


如前所述，滚动是 HTML5 之前 DOM 标准没有涉及的领域。虽然 HTML5 把 scrollIntoView() 标准化了，但不同浏览器中仍然有其他专有方法。比如，scrollIntoViewIfNeeded()作 为 HTMLElement 类型的扩展可以在所有元素上调用。scrollIntoViewIfNeeded(alingCenter)会在 元素不可见的情况下，将其滚动到窗口或包含窗口中，使其可见；如果已经在视口中可见，则这个方法 什么也不做。如果将可选的参数 alingCenter 设置为 true，则浏览器会尝试将其放在视口中央。Safari、 Chrome 和 Opera 实现了这个方法


```javascript
// 如果不可见，则将元素可见
document.images[0].scrollIntoViewIfNeeded(); 
```