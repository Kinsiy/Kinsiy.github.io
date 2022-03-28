---
title: Javascript-DOM-1
date: 2021-03-30 20:39:31
tags: [JS红宝书, DOM]
categories: [学习笔记, Javascript]
keywords:
description: 
photos:
---

## 节点层级

任何 HTML 或 XML 文档都可以用 DOM 表示为一个由节点构成的层级结构。节点分很多类型，每种类型对应着文档中不同的信息和(或)标记，也有自己不同的特性、数据和方法，而且与其他类型有某种关系。这些关系构成了层级，让标记可以表示为一个以特定节点为根的树形结构。

```html
<html>
    <head>
        <title>Sample Page</title>
    <head>
    <body>
        <p>Hello word!</p>
    </body>
</html>
```

<!-- more -->

document 节点表示每个文档的根节点。在这里，根节点的唯一子节点是<html\>元素,我们称之为文档元素(documentElement)。文档元素是最外层的元素，所有其他元素都存在于这个元素之内。每个文档只能有一个文档元素。在 Html 页面中，文档元素始终是<html\>元素。

### Node 类型

DOMLevel 1 描述了名为 Node 的接口，这个接口是所有 DOM 节点类型都必须实现的。

#### nodeName 与 nodeValue

nodeName 与 nodeValue 保存着有关节点的信息。这两个属性的值完全取决于节点类型。对元素而言，nodeName 始终等于元素的标签名，而 nodeValue 则始终为 null。

#### 节点关系

文档中的所有节点都与其他节点有关系。这些关系可以形容为家族关系，相当于把文档树比作家谱。
每个节点都有一个 childNodes 属性，其中包含一个 NodeList 的实例。NodeList 是一个类数组对象，用于存储可以按位置存取的有序节点。注意，NodeList 并不是 Array 的实例，但可以使用中括号访问它的值，而且它也有 length 属性。NodeList 是实时的活动对象，而不是第一次访问时所获得内容的快照。

```javascript
let firstChild = someNode.childNodes[0];
let secondChild = soNode.childNodes.item(1);
let count = someNode.childNodes.length;
```

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Node.jpg)
利用这些关系指针，几乎可以访问到文档树中的任何节点。还可以使用 hasChildNodes()方法检测节点是否存在子节点，如果这个方法返回 true 说明节点有一个或多个子节点。

#### 操纵节点

1. <b>appendChild()</b>
   用于在 childNodes 列表末尾添加节点。添加新节点会更新相关的关系指针，包活父节点和之前的最后一个子节点。appendChild()方法返回新添加的节点。如果把文档中已经存在的节点传给 appendChild()，则这个节点会从之前的位置被转移到新位置。
2. <b>insertBefore()</b>
   如果想把节点放到 childNodes 中的特定位置而不是末尾，则可以使用 insertBefore()方法。这个方法接收两个参数：要插入的节点和参照节点。调用这个方法后，要插入的节点会变成参照节点的前一个同胞节点，并被返回。如果参照节点是 null，则 insertBefore()与 appendChild()效果相同。
3. <b>replaceChild()</b>
   replaceChild()方法接收两个参数：要插入的节点和要替换的节点。要替换的节点会被返回并从文档树中完全移除，要插入的节点会取而代之。
4. 其他方法
   <b>cloneNode()</b>,会返回与调用它的节点一模一样的节点。cloneNode()还接收一个布尔值参数表示是否深复制。在传入 true 参数时，会进行深复制，即复制节点及其整个字 DOM 树。如果传入 false，则只会复制调用该方法的节点。复制返回的节点属于文档所有，但尚未指定父节点，可称为孤儿节点(orphan)
   <b>normalize()</b>，处理文档子树中的文本节点。在节点上调用 normalize()方法会检测这个节点的所有后代，从中搜索不包含文本的文本节点已经互为同胞关系的文本节点，如果发现空文本节点，则将其删除；如果两个同胞节点是相邻的，则将其合并为一个文本节点。

```javascript
// 假设someNode有多个子节点
let returnedNode = someNode.appendChild(someNode.firstChild);
console.log(returnedNode === someNode.firstChild); // false
console.log(returnedNode === someNode.lastChild); // true

let returnNode_1 = someNode.insertBefore(newNode, someNode.firstChild); // 在第一个节点前插入一个新节点
let returnNode_1 = someNode.replaceChild(newNode, someNode.childNodes[someNode.childNodes.length - 2]); // 替换倒数第二个节点
```

### Document 类型

Document 类型是 Javascript 中表示文档节点的类型。在浏览器中，文档对象是 HTMLDocument 的实例(HTMLDocument 继承 Document)，表示整个 HTML 页面。document 是 window 对象的属性，因此是一个全局对象

#### 文档子节点

DOM 规范规定 Document 节点的子节点可以是 DocumentType、Element、ProcessingInstruction 或 Comment 类型，也提供了两个访问子节点的快捷方式。第一个是 documentElement 属性，始终指向 HTML 页面中的<html\>元素。第二个是 body 属性，直接指向<body\>元素

```html
<html>
    <body>
        <script>
            let html = document.documentElement
            let body = document.body
            console.log(html.nodeName); // HTML
            console.log(body.nodeName);  // BODY
        <script>
    </body>
</html>
```

#### 文档信息

title 属性，包含<title\>元素中的文档，通常显示在浏览器窗口或标签页的标题栏。通过这个属性可以读写页面的标题，修改后的标题也会反映在浏览器标题栏上。不过，修改 title 属性并不会改变<title\>元素
URL 属性包含当前页面的完整 URL，domain 包含页面的域名，而 referrer 包含链接到当前页面的那个页面的 URL。如果当前页面没有来源，则 referrer 属性包含空字符串。所有这些信息都可以在请求的 HTTP 头部信息中获取，只是在 Javascript 中通过这几个属性暴露出来而已。

```javascript
let url = document.URL;
let domain = document.domain;
let referrer = document.referrer;
```

#### 定位元素

使用 DOM 最常见的情形可能就是获取某个或某组元素的引用，然后对它们执行某些操作。

1. getElementById()
   方法接收一个参数，即要获取元素的 ID，如果找到了则返回这个元素，如果没找到则返回 null。参数 ID 必须跟在页面中的 id 属性值完全匹配，包括大小写。如果页面中存在多个具有多个相同的 ID 的元素，则 getElementById()返回在文档中出现的第一个元素。
2. getElementByTagName()
   方法接收一个参数，即要获取元素的标签名，返回包含零个或多个元素的 NodeList
3. getElementByName()
   顾名思义，这个方法会返回具有给定 name 属性的所有元素。

#### 特殊集合

-   document.anchors 包含文档中所有带 name 属性的<a\>元素
-   document.froms 包含文档中所有<from\>元素
-   document.images 包含文档中所有<img\>元素
-   document.links 包含文档中所有带 href 属性的<a\>元素

### Element 类型

Elemenet 表示 XML 或 HTML 元素，对外暴露出访问元素标签名

#### HTML 元素

所有 HTML 元素都通过 HTMLElement 类型表示，包括其直接实例和间接实例。另外，HTMLElement 直接继承 Element 并增加了一些属性。每个属性都对应下列属性之一，他们是所有 HTML 元素都有的标准属性

-   id，元素在文档中的唯一标识符
-   title，包含元素的额外信息，通常以提示条形式展示
-   lang，元素内容的语言代码
-   dir，语言的书写方向
-   className，相当于 class 属性，用于指定元素的 CSS 类

#### 取得属性

每个元素都有零个或多个属性，通常用于为元素或其内容附加更多信息。

-   getAttribute()
-   setAttribute()
-   removeAttribute()

顾名思义，这几个方法不用多说。稍微需要注意的是，通过对象形式为元素添加自定义属性是不会让它变成元素的属性。

```javascript
div.myColor = "red";
console.log(div.getAttribute("myColor")); // null (IE除外)
```

#### attributes 属性

Element 类型是唯一使用 attributes 属性的 DOM 节点类型。attribute 属性包含一个 NamedNodeMap 实例，是一个类似 NodeList 的"实时"集合。元素的每个属性都表示为一个 Attr 节点，并保存在这个 NamedNodeMap 对象中。NamedNodeMap 对象包含下列方法：

-   getNamedItem(name),返回 nodeName 属性等于 name 的节点
-   removeNamedItem(name),删除 nodeName 属性等于 name 的节点
-   setNamedItem(node),向列表中添加 node 节点，以其 nodeName 为索引
-   item(pos),返回索引位置 pos 处的节点

一般不怎么用，getAttribute()、setAttribute()、removeAttribute()这三个使用起来更简便，attributes 属性最有用的场景是需要迭代元素上所有属性的时候。

#### 创建元素

可以使用 document.createElement()方法创建新元素。这个方法接收一个参数，即要创建元素的标签名。在 HTML 文档中，标签名是不区分大小写的，而 XML 文档(包括 XHTML)是区分大小写的
创建的元素没有添加到文档树中，不会影响到浏览器显示，要把元素添加到文档树中，可以使用 appendChild()、insertBefore()或 replaceChild()。

#### 元素后代

元素可以拥有任意多个子元素和后代元素，因为元素本身也可以是其他元素的子元素。childNodes 属性包含所有的子节点，这些子节点可能是其他元素、文本节点、注释或处理指令。不同浏览器在识别这些节点时的表现明显不同。

```html
<ul>
	<li>item1</li>
	<li>item2</li>
	<li>item3</li>
</ul>
```

在解析以上代码是，<ul\>元素会包含 7 个子元素，其中 3 个事<li\>元素，还有 4 个 Text 节点(表示<li\>元素周围的空格)。如果把元素之间的空格删掉，则所有浏览器都会返回相同数量的子节点

### 其他类型

Text 类型 && Comment 类型 && CDATASection 类型
DocumentType 类型 && DocumentFragment 类型 && Attr 类型
略 🤣
