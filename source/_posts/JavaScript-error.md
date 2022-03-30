---
title: JavaScript-错误处理与调试
date: 2020-11-25 01:26:39
description: Javascript错误对象汇总、console、debugger、try-catch
categories: [学习笔记, Javascript]
tags: [JS红宝书, Debug]
---

# Error 对象

## SyntaxError

语法不正确。这是因为代码不遵循语言规范。通常因为输入错误导致。

<!--more-->

```javascript
//引号不匹配或未闭合
document.write("Howdy')
```

## ReferenceError

变量不存在。这是因为变量未声明或不在作用域内。

```javascript
//变量未声明
var width = 3;
var area = width * height;
```

## TypeError

值为意外的数据类型。这通常是因为尝试使用不存在的对象或方法而引起的。

```javascript
//方法不存在
var box = {};
box.GetArea();

//DOM节点不存在
var el = document.getElementById("Name");
el.innerHTML = "Kinsiy";
```

## RangeError

数字在可接受范围之外。使用了可接受范围之外的数字来调用某个函数所致。

```javascript
//无法创建长度为-1的数组
var AnArray = new Array(-1);
```

## URIError

没有正确使用 URI 函数。如果 URI 没有对以下字符进行转义，就会导致此错误：/ ? & # : ;

```javascript
//字符未转义
decodeURI("http://bbc.com/news.php ? a=1"); // URIError: URI error
```

## EvalError

没有正确使用 eval()函数。eval()函数通过解释器来评估文本，并将其作为代码来执行。很少能看到这种类型的错误，因为通常浏览器会在发现 EvalError 时，抛出其他类型错误。

## Error

一般错误类型。一般错误对象是用来创建其他所有错误对象的模板(或原型)。

## NAN

不是错误。如果使用一个非数字的值来执行数学运算的话，就会得到 NaN 这样的结果，这并不是错误。

```javascript
// 不是数字
var total = 3 * "Ivy";
```

# console

-   console.log()
-   console.info() 用于一般信息
-   console.warn() 用于警告
-   console.error() 用于输出错误
-   console.group() 消息分组 用于输出一组相关数据
-   console.table() 如果浏览器支持的话可以使用此方法输出表格
-   console.assert() 根据条件输出

```javascript
//console.group()
console.group("Area calculations");
console.info("Width ", width);
console.info("Height ", height);
console.log(area);
console.groupEnd();

//console.table()
var contacts = {
  London: {
    Tel: "110",
    Country: "UK",
  },
  Syfeny: {
    Tel: "120",
    Country: "CN",
  },
  Kinsiy: {
    Tel: "119",
    Country: "USA",
  },
};
console.table(contacts);

// console.assert()
var width, height, area;
width = $("#width").val();
height = $("#height").val();
area = width * height;
console.assert($.isNumeric(area), "用户输入非数字！");
```

# debugger

使用 debugger 关键字在代码中创建断点。打开开发者工具后，就会自动创建这个断点。可以将 debugger 关键字添加到条件语句中，在满足条件时触发断点。

```javascript
var width, height, area;
width = $("#width").val();
height = $("#height").val();
area = width * height;
if (area < 20) {
  debugger;
}
```

# try catch finally

```javascript
...
response = ' {"deal": {["title": "Javascript错误处理与调试",...'        // Json 串
if (response){
    try{
        var dealData = JSON.parse(response);
        ...
    }catch(e){
        var errorMessage = e.name + " " + e.message;
        console.log(errorMessage);
        feed.innerHTML = "<em>加载deal数据出错！</em>"
    }finally{
        var link = document.creatElement("a");
        link.innerHTML = "<a herf = 'dealerror.html'>重新加载</a>"；
        feed.appendChild(link);
    }
}
...
```

# 抛出错误

throw new Error('message')

```javascript
var width = 12;
var height = "Test";
function calculateArea(width, height) {
  try {
    var area = width * height;
    if (!isNaN(area)) {
      return area;
    } else {
      throw new Error("calculateArea() 入参无效！");
    }
  } catch (e) {
    var errorMessage = e.name + " " + e.message;
    console.log(erroeMessage);
    return "无法计算！";
  }
}
```
