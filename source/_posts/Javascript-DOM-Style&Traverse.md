---
title: Javascript-DOM-Style/Traverse
date: 2021-10-22 19:54:58
tags: [ DOM]
categories: [学习笔记, Javascript]
keywords: 
description: DOM1主要定义了HTML和XML文档的底层结构。DOM2和 DOM3在这些结构之上加入更多交互能力，提供了更高级的 XML 特性
photos:
---
## DOM 的演进

DOM2 和 DOM3 Core 模块的目标是扩展 DOM API，满足 XML 的所有需求并提供更好的错误处理 和特性检测。很大程度上，这意味着支持 XML 命名空间的概念。DOM2 Core 没有新增任何类型，仅仅 在 DOM1 Core 基础上增加了一些方法和属性。DOM3 Core 则除了增强原有类型，也新增了一些新类型。

更多信息，请自行查阅🤣


## 样式


HTML 中的样式有 3 种定义方式：外部样式表（通过<link\>元素）、文档样式表（使用<style\>元素）和元素特定样式（使用 style 属性）。DOM2 Style 为这 3 种应用样式的机制都提供了 API。


### 存取元素样式


任何支持 style 属性的 HTML 元素在 JavaScript 中都会有一个对应的 style 属性。这个 style 属性是 CSSStyleDeclaration 类型的实例，其中包含通过 HTML style 属性为元素设置的所有样式信息，但不包含通过层叠机制从文档样式和外部样式中继承来的样式。HTML style 属性中的 CSS 属性在 JavaScript style 对象中都有对应的属性。因为 CSS 属性名使用连字符表示法（用连字符分隔两个单词，如 background-image），所以在 JavaScript 中这些属性必须转换为驼峰大小写形式（如backgroundImage）。


| CSS属性          | JavaScript 属性       |
| ---------------- | --------------------- |
| background-image | style.backgroundImage |
| color            | style.color           |
| display          | style.display         |
| font-family      | style.fontFamily      |


大多数属性名会这样直接转换过来。但有一个 CSS 属性名不能直接转换，它就是 float。因为 float 是 JavaScript 的保留字，所以不能用作属性名。DOM2 Style 规定它在 style 对象中对应的属性 应该是 cssFloat。 


任何时候，只要获得了有效 DOM 元素的引用，就可以通过 JavaScript 来设置样式。


```javascript
let myDiv = document.getElementById("myDiv"); 
// 设置背景颜色
myDiv.style.backgroundColor = "red"; 
// 修改大小
myDiv.style.width = "100px"; 
myDiv.style.height = "200px"; 
// 设置边框
myDiv.style.border = "1px solid black"; 
```


#### DOM样式属性和方法


DOM2 Style 规范也在 style 对象上定义了一些属性和方法。这些属性和方法提供了元素 style 属性的信息并支持修改，列举如下。


- cssText，包含style属性中的CSS代码
- length，应用给元素的CSS属性数量
- parentRule，表示CSS信息的CSSRule对象
- getPropertyPriority(propertyName) ，如果CSS属性propertyName使用了!important，则返回"important"，否则返回空字符串
- getPropertyValue(propertyName)，返回属性propertyName的字符串值
- item(index)，返回索引为index的CSS属性名
- removeProperty(propertyName)，从样式表删除CSS属性propertyName
- setProperty(propertyName, value, priority)，设置CSS属性propertyName的值为value，priority是"important"或空字符串
- getPropertyCSSValue(propertyName)，返回包含 CSS 属性 propertyName 值的 CSSValue对象（已废弃）。


通过 cssText 属性可以存取样式的 CSS 代码。在读模式下，cssText 返回 style 属性 CSS 代码在浏览器内部的表示。在写模式下，给 cssText 赋值会重写整个 style 属性的值，意味着之前通过style 属性设置的属性都会丢失


```javascript
let prop, value, i, len; 
for (i = 0, len = myDiv.style.length; i < len; i++) { 
 prop = myDiv.style[i]; // 或者用 myDiv.style.item(i) 
 value = myDiv.style.getPropertyValue(prop); 
 console.log(`prop: ${value}`); 
} 
```


### 操作样式表


CSSStyleSheet 类型表示 CSS 样式表，包括使用<link\>元素和通过<style\>元素定义的样式表。注意，这两个元素本身分别是 HTMLLinkElement 和 HTMLStyleElement。CSSStyleSheet 类型是一个通用样式表类型，可以表示以任何方式在 HTML 中定义的样式表。另外，元素特定的类型允许修改HTML 属性，而 CSSStyleSheet 类型的实例则是一个只读对象（只有一个属性例外）


CSSStyleSheet类型继承StyleSheet，后者可用作非CSS样式表的基类。以下是CSSStyleSheet从 StyleSheet 继承的属性


-  disabled，布尔值，表示样式表是否被禁用了（这个属性是可读写的，因此将它设置为 true会禁用样式表）
-  href，如果是使用<link\>包含的样式表，则返回样式表的 URL，否则返回 null。 
-  media，样式表支持的媒体类型集合，这个集合有一个 length 属性和一个 item()方法，跟所有 DOM 集合一样。同样跟所有 DOM 集合一样，也可以使用中括号访问集合中特定的项。如果样式表可用于所有媒体，则返回空列表。
-  ownerNode，指向拥有当前样式表的节点，在 HTML 中要么是<link\>元素要么是<style\>元素（在 XML 中可以是处理指令）。如果当前样式表是通过@import 被包含在另一个样式表中，则这个属性值为 null。
-  parentStyleSheet，如果当前样式表是通过@import 被包含在另一个样式表中，则这个属性指向导入它的样式表
-  title，ownerNode 的 title 属性。
-  type，字符串，表示样式表的类型。对 CSS 样式表来说，就是"text/css"。


上述属性里除了 disabled，其他属性都是只读的。除了上面继承的属性，CSSStyleSheet 类型还支持以下属性和方法。


-  cssRules，当前样式表包含的样式规则的集合。
-  ownerRule，如果样式表是使用@import 导入的，则指向导入规则；否则为 null。
-  deleteRule(index)，在指定位置删除 cssRules 中的规则。
-  insertRule(rule, index)，在指定位置向 cssRules 中插入规则。


document.styleSheets 表示文档中可用的样式表集合。这个集合的 length 属性保存着文档中样式表的数量，而每个样式表都可以使用中括号或 item()方法获取


```javascript
let sheet = null; 
for (let i = 0, len = document.styleSheets.length; i < len; i++) { 
 sheet = document.styleSheets[i]; 
 console.log(sheet.href); 
} 
```


#### CSS 规则


CSSRule 类型表示样式表中的一条规则。这个类型也是一个通用基类，很多类型都继承它，但其中最常用的是表示样式信息的 CSSStyleRule（其他 CSS 规则还有@import、@font-face、@page 和@charset 等，不过这些规则很少需要使用脚本来操作）。以下是 CSSStyleRule 对象上可用的属性


- cssText，返回整条规则的文本。
- parentRule，如果这条规则被其他规则(如@media)包含，则指向包含规则，否则就是null
- parentStyleSheet，包含当前绘制的样式表
- selectorText，返回规则的选择符文本。
- style，返回CSSStyleDeclaration对象，可以设置和获取当前规则中的样式
- type，数值常量，表示规则类型。对于样式规则，它始终是-1


这里的文本可能于样式表中实际的文本不一样，因为浏览器内部处理样式表的方式也不一样


在这些属性中，使用最多的是 cssText、selectorText 和 style。cssText 属性与 style.cssText类似，不过并不完全一样。前者包含选择符文本和环绕样式声明的大括号，而后者则只包含样式声明（类似于元素上的 style.cssText）。此外，cssText 是只读的，而 style.cssText 可以被重写。


```javascript
/** 
假设这条规则位于页面中的第一个样式表中，而且是该样式表中唯一一条 CSS 规则


div.box { 
 background-color: blue; 
 width: 100px; 
 height: 200px; 
} 
*/


let sheet = document.styleSheets[0]; 
let rules = sheet.cssRules || sheet.rules; // 取得规则集合
let rule = rules[0]; // 取得第一条规则
console.log(rule.selectorText); // "div.box" 
console.log(rule.style.cssText); // 完整的 CSS 代码
console.log(rule.style.backgroundColor); // "blue" 
console.log(rule.style.width); // "100px" 
console.log(rule.style.height); // "200px"


let sheet = document.styleSheets[0]; 
let rules = sheet.cssRules || sheet.rules; // 取得规则集合
let rule = rules[0]; // 取得第一条规则
rule.style.backgroundColor = "red" 
```


#### 创建规则


DOM 规定，可以使用 insertRule()方法向样式表中添加新规则。这个方法接收两个参数：规则 的文本和表示插入位置的索引值


```javascript
sheet.insertRule("body { background-color: silver }", 0); // 使用 DOM 方法
```


这个例子插入了一条改变文档背景颜色的规则。这条规则是作为样式表的第一条规则（位置 0）插入的，顺序对规则层叠是很重要的。


虽然可以这样添加规则，但随着要维护的规则增多，很快就会变得非常麻烦。这时候，更好的方式是使用动态样式加载技术。


#### 删除规则


支持从样式表中删除规则的 DOM 方法是 deleteRule()，它接收一个参数：要删除规则的索引。要删除样式表中的第一条规则，可以这样做：


```JavaScript
sheet.deleteRule(0); // 使用 DOM 方法
```


与添加规则一样，删除规则并不是 Web 开发中常见的做法。考虑到可能影响 CSS 层叠的效果，删除规则时要慎重

### 元素尺寸

[Javascript.info - 几何](https://zh.javascript.info/size-and-scroll#ji-he)

![](https://zh.javascript.info/article/size-and-scroll/metric-all.svg)


#### 偏移尺寸


第一组属性涉及偏移尺寸（offset dimensions），包含元素在屏幕上占用的所有视觉空间。元素在页面上的视觉空间由其高度和宽度决定，包括所有内边距、滚动条和边框（但不包含外边距）。以下 4 个属性用于取得元素的偏移尺寸


- offsetHeight，元素在垂直方向上占用的像素尺寸，包括它的高度、水平滚动条高度(如果可见)和上下边框的高度
- offsetLeft，元素左边框外侧距离包含元素左边框内侧的像素数
- offsetTop，元素上边框外侧距离包含元素上边框内侧的像素数
- offsetWidth，元素在水平方向上占用的像素尺寸，包括他的宽度、垂直滚动条高度(如果可见)和左、右边框的宽度。


其中，offsetLeft 和 offsetTop 是相对于包含元素的，包含元素保存在 offsetParent 属性中。offsetParent 不一定是 parentNode。比如，<td\>元素的 offsetParent 是作为其祖先的<table\>元素，因为<table\>是节点层级中第一个提供尺寸的元素。


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/offsetSize.png)


要确定一个元素在页面中的偏移量，可以把它的 offsetLeft 和 offsetTop 属性分别与 offsetParent的相同属性相加，一直加到根元素。


```JavaScript
function getElementLeft(element) { 
 let actualLeft = element.offsetLeft; 
 let current = element.offsetParent; 
 while (current !== null) { 
 actualLeft += current.offsetLeft; 
 current = current.offsetParent; 
 } 
 return actualLeft; 
} 


function getElementTop(element) { 
 let actualTop = element.offsetTop; 
 let current = element.offsetParent; 
 while (current !== null) { 
 actualTop += current.offsetTop; 
 current = current.offsetParent; 
 } 
 return actualTop; 
} 
```


#### 客户端尺寸


元素的客户端尺寸（client dimensions）包含元素内容及其内边距所占用的空间。客户端尺寸只有两个相关属性：clientWidth 和 clientHeight。其中，clientWidth 是内容区宽度加左、右内边距宽度，clientHeight 是内容区高度加上、下内边距高度


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/clientSize.png)


客户端尺寸实际上就是元素内部的空间，因此不包含滚动条占用的空间。这两个属性最常用于确定浏览器视口尺寸，即检测 document.documentElement 的 clientWidth 和 clientHeight。这两个属性表示视口（<html\>或<body\>元素）的尺寸。


#### 滚动尺寸


最后一组尺寸是滚动尺寸（scroll dimensions），提供了元素内容滚动距离的信息。有些元素，比如<html\>无须任何代码就可以自动滚动，而其他元素则需要使用 CSS 的 overflow 属性令其滚动。滚动尺寸相关的属性有如下 4 个


- scrollHeight，没有滚动条出现时，元素内容的总高度
- scrollLeft，内容区左侧隐藏的像素数，设置这个属性可以改变元素的滚动位置
- scrollTop，内容区顶部隐藏的像素数，设置这个属性可以改变元素的滚动位置
- scrollWidth，没有出现滚动条时，元素内容的总宽度


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/scrollSize.png)


scrollWidth 和 scrollHeight 可以用来确定给定元素内容的实际尺寸。例如，<html\>元素是浏览器中滚动视口的元素。因此，document.documentElement.scrollHeight 就是整个页面垂直方向的总高度。


scrollWidth 和 scrollHeight 与 clientWidth 和 clientHeight 之间的关系在不需要滚动的文档上是分不清的。如果文档尺寸超过视口尺寸，则在所有主流浏览器中这两对属性都不相等，scrollWidth 和 scollHeight 等于文档内容的宽度，而 clientWidth 和 clientHeight 等于视口的大小。


scrollLeft 和 scrollTop 属性可以用于确定当前元素滚动的位置，或者用于设置它们的滚动位置。元素在未滚动时，这两个属性都等于 0。如果元素在垂直方向上滚动，则 scrollTop 会大于 0，表示元素顶部不可见区域的高度。如果元素在水平方向上滚动，则 scrollLeft 会大于 0，表示元素左侧不可见区域的宽度。因为这两个属性也是可写的，所以把它们都设置为 0 就可以重置元素的滚动位置。


#### 确定元素尺寸


浏览器在每个元素上都暴露了 getBoundingClientRect()方法，返回一个 DOMRect 对象，包含6 个属性：left、top、right、bottom、height 和 width。这些属性给出了元素在页面中相对于视口的位置


![DOMRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect/element-box-diagram.png)


## 遍历


DOM2 Traversal and Range 模块定义了两个类型用于辅助顺序遍历 DOM 结构。这两个类型—— NodeIterator 和 TreeWalker——从某个起点开始执行对 DOM 结构的深度优先遍历。


如前所述，DOM 遍历是对 DOM 结构的深度优先遍历，至少允许朝两个方向移动（取决于类型）。遍历以给定节点为根，不能在 DOM 中向上超越这个根节点


### NodeIterator


NodeIterator 类型是两个类型中比较简单的，可以通过 document.createNodeIterator()方法创建其实例。这个方法接收以下 4 个参数。


- root，作为遍历根节点的节点。
- whatToShow，数值代码，表示应该访问哪些节点。
- filter，NodeFilter 对象或函数，表示是否接收或跳过特定节点。
- entityReferenceExpansion，布尔值，表示是否扩展实体引用。这个参数在 HTML 文档中没有效果，因为实体引用永远不扩展。


whatToShow 参数是一个位掩码，通过应用一个或多个过滤器来指定访问哪些节点。这个参数对应的常量是在 NodeFilter 类型中定义的。


- NodeFilter.SHOW_ALL，所有节点。
- NodeFilter.SHOW_ELEMENT，元素节点。
- NodeFilter.SHOW_ATTRIBUTE，属性节点。由于 DOM 的结构，因此实际上用不上
- NodeFilter.SHOW_TEXT，文本节点。
- NodeFilter.SHOW_CDATA_SECTION，CData 区块节点。不是在 HTML 页面中使用的。
- NodeFilter.SHOW_ENTITY_REFERENCE，实体引用节点。不是在 HTML 页面中使用的。
- NodeFilter.SHOW_ENTITY，实体节点。不是在 HTML 页面中使用的。
- NodeFilter.SHOW_PROCESSING_INSTRUCTION，处理指令节点。不是在 HTML 页面中使用的。
- NodeFilter.SHOW_COMMENT，注释节点。
- NodeFilter.SHOW_DOCUMENT，文档节点。
- NodeFilter.SHOW_DOCUMENT_TYPE，文档类型节点。
- NodeFilter.SHOW_DOCUMENT_FRAGMENT，文档片段节点。不是在 HTML 页面中使用的。
- NodeFilter.SHOW_NOTATION，记号节点。不是在 HTML 页面中使用的


这些值除了 NodeFilter.SHOW_ALL 之外，都可以组合使用。比如，可以像下面这样使用按位或 操作组合多个选项


```javascript
let whatToShow = NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT;
```


createNodeIterator()方法的 filter 参数可以用来指定自定义 NodeFilter 对象，或者一个作为节点过滤器的函数NodeFilter 对象只有一个方法 acceptNode()，如果给定节点应该访问就返回 NodeFilter.FILTER_ACCEPT，否则返回NodeFilter.FILTER_SKIP。因为 NodeFilter 是一个抽象类型，所以不可能创建它的实例。只要创建一个包含 acceptNode()的对象，然后把它传给createNodeIterator()就可以了。以下代码定义了只接收<p\>元素的节点过滤器对象：


```javascript
let filter = { 
   acceptNode(node) { 
        return node.tagName.toLowerCase() == "p" ? NodeFilter.FILTER_ACCEPT : 
        NodeFilter.FILTER_SKIP;
    }
}; 
let iterator = document.createNodeIterator(root, NodeFilter.SHOW_ELEMENT, filter, false); 


// 创建一个简单的遍历所有节点的 NodeIterator
let iterator = document.createNodeIterator(document, NodeFilter.SHOW_ALL, null, false);
```

NodeIterator 的两个主要方法是 nextNode()和 previousNode()。nextNode()方法在 DOM子树中以深度优先方式进前一步，而 previousNode()则是在遍历中后退一步。创建 NodeIterator对象的时候，会有一个内部指针指向根节点，因此第一次调用 nextNode()返回的是根节点。当遍历到达 DOM 树最后一个节点时，nextNode()返回 null。previousNode()方法也是类似的。当遍历到达DOM 树最后一个节点时，调用 previousNode()返回遍历的根节点后，再次调用也会返回 null。

```html
<div id="div1"> 
    <p><b>Hello</b> world!</p> 
    <ul> 
         <li>List item 1</li> 
         <li>List item 2</li> 
         <li>List item 3</li> 
    </ul> 
</div> 
```




```javascript
// 遍历<div>元素内部的所有元素


let div = document.getElementById("div1"); 
let iterator = document.createNodeIterator(div, NodeFilter.SHOW_ELEMENT, null, false); 
let node = iterator.nextNode(); 
while (node !== null) { 
 console.log(node.tagName); // 输出标签名
 node = iterator.nextNode(); 
} 
```


nextNode()和 previousNode()方法共同维护 NodeIterator 对 DOM 结构的内部指针，因此修改 DOM 结构也会体现在遍历中。


### TreeWalker


TreeWalker 是 NodeIterator 的高级版。除了包含同样的 nextNode()、previousNode()方法，TreeWalker 还添加了如下在 DOM 结构中向不同方向遍历的方法。


- parentNode()，遍历到当前节点的父节点
- firstChild()，遍历到当前节点的第一个子节点
- lastChild()，遍历到当前节点的最后一个子节点
- nextSibling()，遍历到当前节点的下一个同胞节点
- previousSibling()，遍历到当前节点的上一个同胞节点


TreeWalker 对象要调用 document.createTreeWalker()方法来创建，这个方法接收与document.createNodeIterator()同样的参数：作为遍历起点的根节点、要查看的节点类型、节点过滤器和一个表示是否扩展实体引用的布尔值。因为两者很类似，所以 TreeWalker 通常可以取代NodeIterator


不同的是，节点过滤器（filter）除了可以返回 NodeFilter.FILTER_ACCEPT 和 NodeFilter. FILTER_SKIP，还可以返回 NodeFilter.FILTER_REJECT。在使用 NodeIterator 时，NodeFilter. FILTER_SKIP 和 NodeFilter.FILTER_REJECT 是一样的。但在使用 TreeWalker 时，NodeFilter. FILTER_SKIP 表示跳过节点，访问子树中的下一个节点，而NodeFilter.FILTER_REJECT 则表示跳过该节点以及该节点的整个子树。例如，如果把前面示例中的过滤器函数改为返回 NodeFilter. FILTER_REJECT（而不是 NodeFilter.FILTER_SKIP），则会导致遍历立即返回，不会访问任何节点。这是因为第一个返回的元素是<div\>，其中标签名不是"li"，因此过滤函数返回 NodeFilter.FILTER_ REJECT，表示要跳过整个树。因为<div\>本身就是遍历的根节点，所以遍历会就此结束。


当然，TreeWalker 真正的威力是可以在 DOM 结构中四处游走。如果不使用过滤器，单纯使用TreeWalker 的漫游能力同样可以在 DOM 树中访问<li\>元素


```javascript
let div = document.getElementById("div1"); 
let walker = document.createTreeWalker(div, NodeFilter.SHOW_ELEMENT, null, false); 
walker.firstChild(); // 前往<p> 
walker.nextSibling(); // 前往<ul> 
let node = walker.firstChild(); // 前往第一个<li> 
while (node !== null) { 
 console.log(node.tagName); 
 node = walker.nextSibling(); 
} 
```

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)