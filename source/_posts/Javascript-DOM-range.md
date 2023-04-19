---
title: Javascript-DOM-range
date: 2021-10-22 19:55:58
tags: [ DOM]
categories: [学习笔记, Javascript]
description:
photos:
---

## 范围


为了支持对页面更细致的控制，DOM2 Traversal and Range 模块定义了范围接口。范围可用于在文档中选择内容，而不用考虑节点之间的界限。（选择在后台发生，用户是看不到的。）范围在常规 DOM操作的粒度不够时可以发挥作用。


### DOM范围


DOM2 在 Document 类型上定义了一个 createRange()方法，暴露在 document 对象上。使用这个方法可以创建一个 DOM 范围对象

与节点类似，这个新创建的范围对象是与创建它的文档关联的，不能在其他文档中使用。然后可以使用这个范围在后台选择文档特定的部分。创建范围并指定它的位置之后，可以对范围的内容执行一些操作，从而实现对底层 DOM 树更精细的控制。

<!-- more -->


每个范围都是 Range 类型的实例，拥有相应的属性和方法


- startContainer，范围起点所在的节点(选区中第一个子节点的父节点)
- startOffset，范围起点在startContainer中的偏移量。如果startContainer是文本节点、注释节点或CData区块节点，则startOffset指范围起点之前跳过的字符数；否则，表示范围中第一个节点的索引
- endContainer，范围终点所在的节点(选区中最后一个子节点的父节点)
- endOffset，范围起点在startContainer中的偏移量(与startOffset中偏移量的含义相同)
- commonAncestorContainer，文档中以startContainer和endContainer为后代的最深的节点


这些属性会在范围被放到文档中特定位置时获得响应的值


### 简单选择


通过范围选择文档中某个部分最简单的方式，就是使用selectNode()或selectNodeContenets()方法。这两个方法都接收一个节点作为参数，并将该节点的信息添加到调用它的范围。selectNode()方法选择整个节点，包含其后代节点，而selectNodeContents()只选择节点的后代


```html
<!DOCTYPE html> 
<html> 
  <body> 
    <p id="p1"><b>Hello</b> world!</p> 
  </body> 
</html> 
```


```javascript
let range1 = document.createRange()
let range2 = document.createRange()
let p1 = document.getElementById("p1")
range1.selectNode(p1); 
range2.selectNodeContents(p1); 
```


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/simpleRangeSelect.png)


调用 selectNode()时，startContainer、endContainer 和 commonAncestorContainer 都等于传入节点的父节点。在这个例子中，这几个属性都等于 document.body。startOffset 属性等于传入节点在其父节点 childNodes 集合中的索引（在这个例子中，startOffset 等于 1，因为 DOM的合规实现把空格当成文本节点），而 endOffset 等于 startOffset 加 1（因为只选择了一个节点）。


在调用 selectNodeContents()时，startContainer、endContainer 和 commonAncestor 
Container 属性就是传入的节点，在这个例子中是<p\>元素。startOffset 属性始终为 0，因为范围从传入节点的第一个子节点开始，而 endOffset 等于传入节点的子节点数量（node.child Nodes.length），在这个例子中等于 2。


在像上面这样选定节点或节点后代之后，还可以在范围上调用相应的方法，实现对范围中选区的更精细控制


- setStartBefore(refNode)，把范围的起点设置到refNode之前，从而让refNode成为选区的第一个子节点。startContainer属性被设置为refNode.parentNode，而startOffset属性被设置为refNode在其父节点childNodes集合中的索引
- setStartAfter(refNode)，把范围的起点设置到refNode之后，从而将refNode排除在选区之外，让其下一个同胞节点成为选区的第一个字节点。startContainer属性被refNode.parentNode，startOffset属性被设置为refNode在其父节点childNodes集合中的索引加1
- setEndBefore(refNode)，把范围的终点设置到refNode之前，从而将refNode排除在选区之外、让其上一个同胞节点成为选区的最后一个字节点。endContainer属性被设置为refNode.parentNode，endOffset属性被设置为refNode在其父节点childNodes集合中的索引
- setEndAfter(refNode)，把范围的终点设置到refNode之后，从而让refNode成为选区的最后一个子节点。endContainer属性被设置为refNode.parentNode，endOffset属性被设置为refNode在其父节点child集合中的索引加1


调用这些方法时，所有属性都会自动重新赋值。不过，为了实现复杂的选区，也可以直接修改这些属性的值


### 复杂选择


要创建复杂的范围，需要使用setStart()和setEnd()方法。这两个方法都接收两个参数：参照节点和偏移量。对setStart()来说，参照节点会成为startContainer，而偏移量会赋值给startOffset。对setEnd()而言，参照节点会成为endContainer，而偏移量会赋值给endOffset。


使用这两个方法，可以模拟 selectNode()和 selectNodeContents()的行为。


```javascript
let range1 = document.createRange()
let range2 = document.createRange()
let p1 = document.getElementById("p1")
let p1Index = -1
let i
let len

for (i = 0, len = p1.parentNode.childNodes.length; i < len; i++) { 
 if (p1.parentNode.childNodes[i] === p1) { 
   p1Index = i; 
   break; 
 } 
}

range1.setStart(p1.parentNode, p1Index); 
range1.setEnd(p1.parentNode, p1Index + 1); 
range2.setStart(p1, 0); 
range2.setEnd(p1, p1.childNodes.length); 
```


注意，要选择节点（使用 range1），必须先确定给定节点（p1）在其父节点 childNodes 集合中的索引。而要选择节点的内容（使用 range2），则不需要这样计算，因为可以直接给 setStart()和setEnd()传默认值。虽然可以模拟 selectNode()和 selectNodeContents()，但 setStart()和setEnd()真正的威力还是选择节点中的某个部分。

假设我们想通过范围从前面示例中选择从"Hello"中的"llo"到" world!"中的"o"的部分。很简单，第一步是取得所有相关节点的引用

```html
<p id="p1"><b>Hello</b> world!</p>
```


```javascript
let p1 = document.getElementById("p1")
let helloNode = p1.firstChild.firstChild
let worldNode = p1.lastChild

let range = document.createRange(); 
range.setStart(helloNode, 2); 
range.setEnd(worldNode, 3);
```


因为选区起点在"Hello"中的字母"e"之后，所以要给 setStart()传入 helloNode 和偏移量 2（"e"后面的位置，"H"的位置是 0）。要设置选区终点，则要给 setEnd()传入 worldNode 和偏移量 3，即不属于选区的第一个字符的位置，也就是"r"的位置 3（位置 0 是一个空格）。**也就是半开区间 [ start, end )** 


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/compRangeSelect.png)


因为 helloNode 和 worldNode 是文本节点，所以它们会成为范围的 startContainer 和endContainer，这样startOffset 和 endOffset 实际上表示每个节点中文本字符的位置，而不是子节点的位置（传入元素节点时的情形）。而commonAncestorContainer 是<p\>元素，即包含这两个节点的第一个祖先节点。


当然，只选择文档中的某个部分并不是特别有用，除非可以对选中部分执行操作


### 操作范围


创建范围之后，浏览器会在内部创建一个文档片段节点，用于包含范围选区中的节点。为操作范围的内容，选区中的内容必须格式完好。在前面的例子中，因为范围的起点和终点都在文本节点内部，并不是完好的 DOM 结构，所以无法在 DOM 中表示。不过，范围能够确定缺失的开始和结束标签，从而可以重构出有效的 DOM 结构，以便后续操作。


仍以前面例子中的范围来说，范围发现选区中缺少一个开始的<b\>标签，于是会在后台动态补上这个标签，同时还需要补上封闭"He"的结束标签</b\>，结果会把 DOM 修改为这样：


```html
<p><b>He</b><b>llo</b> world!</p> 
```


而且，" world!"文本节点会被拆分成两个文本节点，一个包含" wo"，另一个包含"rld!"


这样创建了范围之后，就可以使用很多方法来操作范围的内容。（注意，范围对应文档片段中的所有节点，都是文档中相应节点的指针。）


第一个方法最容易理解和使用：deleteContents()。顾名思义，这个方法会从文档中删除范围包含的节点


```javascript
let p1 = document.getElementById("p1"), 
 helloNode = p1.firstChild.firstChild, 
 worldNode = p1.lastChild, 
 range = document.createRange(); 
range.setStart(helloNode, 2); 
range.setEnd(worldNode, 3); 
range.deleteContents(); 


/**
执行上面的代码之后，页面中的 HTML 会变成这样：<p><b>He</b>rld!</p> 
*/
```


因为前面介绍的范围选择过程通过修改底层 DOM 结构保证了结构完好，所以即使删除范围之后，剩下的 DOM 结构照样是完好的。


另一个方法 extractContents()跟 deleteContents()类似，也会从文档中移除范围选区。但不同的是，extractContents()方法返回范围对应的文档片段。这样，就可以把范围选中的内容插入文档中其他地方


```javascript
let p1 = document.getElementById("p1")
let helloNode = p1.firstChild.firstChild
let worldNode = p1.lastChild
let range = document.createRange()

range.setStart(helloNode, 2); 
range.setEnd(worldNode, 3); 
let fragment_1 = range.extractContents(); 
p1.parentNode.appendChild(fragment_1);

/** result:
  <p><b>He</b>rld!</p> 
  <b>llo</b> wo 
*/

// 如果不想把范围从文档中移除，也可以使用 cloneContents()创建一个副本，然后把这个副本插入到文档其他地方

let fragment_2 = range.cloneContents(); 
p1.parentNode.appendChild(fragment_2);

/** result:
  <p><b>Hello</b> world!</p> 
  <b>llo</b> wo 
*/
```


### 范围插入

使用insertNode()方法可以在范围选区的开始位置插入一个节点

```html
<p id="p1"><b>Hello</b> world!</p>
```


```javascript
// 插入: <span style="color: red">Inserted text</span>

let p1 = document.getElementById("p1")
let helloNode = p1.firstChild.firstChild 
let worldNode = p1.lastChild
let range = document.createRange(); 
range.setStart(helloNode, 2); 
range.setEnd(worldNode, 3); 
let span = document.createElement("span"); 
span.style.color = "red"; 
span.appendChild(document.createTextNode("Inserted text")); 
range.insertNode(span); 
```

```html
<p id="p1"><b>He<span style="color: red">Inserted text</span>llo</b> world</p>
```

注意，<span\>正好插入到"Hello"中的"llo"之前，也就是范围选区的前面。同时，也要注意原始的 HTML 并没有添加或删除<b\>元素，因为这里并没有使用之前提到的方法。使用这个技术可以插入有用的信息，比如在外部链接旁边插入一个小图标。


除了向范围中插入内容，还可以使用 surroundContents()方法插入包含范围的内容。这个方法接收一个参数，即包含范围内容的节点。调用这个方法时，后台会执行如下操作：


1. 提取出范围的内容
2. 在原始文档中范围之前所在的位置插入给定的节点
3. 将范围对应文档的内容添加到给定节点


这种功能适合在网页中高亮显示某些关键词


```JavaScript
let p1 = document.getElementById("p1")
let helloNode = p1.firstChild.firstChild
let worldNode = p1.lastChild
let range = document.createRange()

range.selectNode(helloNode); 

let span = document.createElement("span"); 
span.style.backgroundColor = "yellow"; 

range.surroundContents(span); 
```

```html
<p><b><span style="background-color:yellow">Hello</span></b> world!</p>
```

### 范围折叠


如果范围并没有选择文档的任何部分，则称为折叠（collapsed）。折叠范围有点类似文本框：如果文本框中有文本，那么可以用鼠标选中以高亮显示全部文本。这时候，如果再单击鼠标，则选区会被移除，光标会落在某两个字符中间。而在折叠范围时，位置会被设置为范围与文档交界的地方，可能是范围选区的开始处，也可能是结尾处


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/rangeCollapsed.png)


折叠范围可以使用 collapse()方法，这个方法接收一个参数：布尔值，表示折叠到范围哪一端。true 表示折叠到起点，false 表示折叠到终点。要确定范围是否已经被折叠，可以检测范围的 collapsed属性


```JavaScript
range.collapse(true); // 折叠到起点
console.log(range.collapsed); // 输出 true

/**
<p id="p1">Paragraph 1</p>
<p id="p2">Paragraph 2</p> 
*/

let p1 = document.getElementById("p1")
let p2 = document.getElementById("p2")
let range = document.createRange(); 

range.setStartAfter(p1); 
range.setStartBefore(p2); 
console.log(range.collapsed); // true
```


### 范围比较


如果有多个范围，则可以使用 compareBoundaryPoints()方法确定范围之间是否存在公共的边界（起点或终点）。这个方法接收两个参数：要比较的范围和一个常量值，表示比较的方式。这个常量参数包括


- Range.START_TO_START(0)，比较两个范围的起点
- Range.START_TO_END(1)，比较第一个范围的起点和第二个范围的终点
- Range.END_TO_END(2)，比较两个范围的终点
- Range.END_TO_START(3)，比较第一个范围的终点和第二个范围的起点

compareBoundaryPoints()方法在第一个范围的边界点位于第二个范围的边界点之前时返回-1， 在两个范围的边界点相等时返回 0，在第一个范围的边界点位于第二个范围的边界点之后时返回 1

```html
<p id="p1"><b>Hello</b> world!</p>
```


```JavaScript
let range1 = document.createRange(); 
let range2 = document.createRange(); 
let p1 = document.getElementById("p1"); 

range1.selectNodeContents(p1); 
range2.selectNodeContents(p1); 
range2.setEndBefore(p1.lastChild); 

console.log(range1.compareBoundaryPoints(Range.START_TO_START, range2)); // 0 
console.log(range1.compareBoundaryPoints(Range.END_TO_END, range2)); // 1 
```


### 复制范围 & 清理


调用范围的 cloneRange()方法可以复制范围。新范围包含与原始范围一样的属性，修改其边界点不会影响原始范围


在使用完范围之后，最好调用 detach()方法把范围从创建它的文档中剥离。调用 detach()之后，就可以放心解除对范围的引用，以便垃圾回收程序释放它所占用的内存。


```JavaScript
let newRange = range.cloneRange(); 

range.detach(); // 从文档中剥离范围
range = null; // 解除引用
```

这两步是最合理的结束使用范围的方式。剥离之后的范围就不能再使用了

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
