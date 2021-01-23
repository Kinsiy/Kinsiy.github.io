---
title: 基本引用类型(单例内置对象)
author: Kinsiy
avatar: https://cdn.jsdelivr.net/gh/Kinsiy/cdn/img/custom/avatar.jpg
authorLink: https://kinsiy.github.io
authorAbout: I'm True
authorDesc: I'm True
categories: SKILL
comments: true
date: 2021-01-23 22:57:33
tags: [Notes,Javascript]
keywords:
description: 介绍ECMA-262定义的两个单例内置对象：Global和Math
photos:
---
任何由EMCAscript实现提供、与宿主环境无关，并在EMCAscript程序开始执行时就存在的对象。前面已经接触了大部分内置对象，包括Object，Array和String。本文介绍EMCA-262 定义的另外两个单例对象：Global和Math。
# Global

Global 对象为一种兜底对象，它所针对的是不属于任何对象的属性和方法。在全局作用域中的变量和方法都会成为Global对象的属性。

## URL编码方法

encodeURI()和encodeURIComponent()方法用于编码统一资源标识符(URI)，以便传给浏览器。
```javascript
/* 编码
    encodeURI()
    encondeURIComponent()
 */
let uri = "https://kinsiy.github.io?name=哈哈哈"
let url_1 = encodeURI(uri);
let url_2 = encodeURIComponent(uri);
console.log(url_1);            // https://kinsiy.github.io?name=%E5%93%88%E5%93%88%E5%93%88
console.log(url_2);            // https%3A%2F%2Fkinsiy.github.io%3Fname%3D%E5%93%88%E5%93%88%E5%93%88

/* 解码
    decodeURI()
    decondeURIComponent()
 */
console.log(decodeURI(url_1));            // https://kinsiy.github.io?name=哈哈哈
console.log(decodeURIComponent(url_2));   // https://kinsiy.github.io?name=哈哈哈
```
## eval()
这个方法就是一个完整的EMCAScript解释器，它接收一个参数，即一个要执行的EMCAScript(Javascript)字符串。
通过eval()定义的任何变量和函数都不会提升。
```javascript
let msg = "hello world!";
eval("console.log(msg)");                           // hello world!
eval("function sayHi() { console.log('Hi'); }");
sayHi();                                            // Hi
eval("var name = 'Kinsiy'");                        // 此处使用let与const声明console.log()均为空，原因不明，作用域原因？
console.log(name);                                  // Kinsiy
```
## Global 属性

| 属性           | 说明                   |
| :------------- | :--------------------- |
| undefined      | 特殊值undefined        |
| NaN            | 特殊值NaN              |
| Infinity       | 特殊值Infinity         |
| Object         | Object的构造函数       |
| Array          | Array的构造函数        |
| Function       | Function的构造函数     |
| Boolean        | Boolean的构造函数      |
| String         | String的构造函数       |
| Number         | Number的构造函数       |
| Date           | Date的构造函数         |
| RegExp         | RegExp的构造函数       |
| Symbol         | Symbol的构造函数       |
| Error          | Errot的构造函数        |
| EvalError      | EvalError的构造函数    |
| RangeError     | RangerError的构造函数  |
| ReferenceError | RefenceError的构造函数 |
| SyntaxError    | SytaxError的构造函数   |
| TypeError      | TypeError的构造函数    |
| URIError       | URIError的构造函数     |

## window 对象
虽然ECMA-262 没有规定直接访问Global对象的方式，但浏览器将window对象实现为Global对象的代理。因此，所有全局作用域声明的变量和函数都变成了window的属性。
```javascript
var color = "red";
function sayColor(){
    console.log(window.color);
}

window.satColor();      // red

/* 另一种获取Global对象的方式 */
let global = function(){
    return this;
}();
```
# Math
## Math对象属性

Math 有一些属性，主要用于保存数学中的一些特殊值。

| 属性         | 说明                |
| :----------- | :------------------ |
| Math.E       | 自然对数的基数e的值 |
| Math.LN10    | In10                |
| Math.LN2     | In2                 |
| Math.LOG2E   | Log<sub>2</sub>e    |
| Math.LOG10E  | Log<sub>10</sub>e   |
| Math.PI      | π                   |
| Math.SQRT1_2 | &radic;(1/2)        |
| Math.SQRT2   | &radic;2            |

## min() && max()
```javascript
let max = Math.max(5,7,48,56.1,54.6,100,1.2,100.5);     // 100.5
let min = Math.min(5,7,48,56.1,54.6,100,1.2,100.5);     // 1.2
console.log(max);
console.log(min);

/* 使用扩展操作符 */
let values = [5, 10, 4, 25, 7, 45];     
console.log(Math.max(...values));           // 45   
```
## 舍入方法
* Math.ceil() 向上取整
* Math.floor() 向下取整
* Math.round() 四舍五入
* Math.fround() 返回数值最接近的单精度(32位)浮点值表示

```javascript
console.log(Math.ceil(25.5));       // 26
console.log(Math.floor(25.5));      // 25
console.log(Math.round(26.6));      // 27
console.log(Math.fround(26.8));     // 26.799999237060547
```
## random()
Math.random()返回一个0~1范围内的随机数，其中包含0不包含1。[0,1)
```javascript
/* 随机数公式
    number = Math.floor(Math.random() * total_number_of_choices + first_possible_value)
 */
 function selectFrom(lowerValue, upperValue){
     let choices = upperValue - lowerValue + 1;
     return Math.floor(Math.random() * choices + lowerValue);
 }

 let i = 0;
 while (i < 5) {
   console.log(selectFrom(5,45));   // [5,45]
   i++;         // 8 17 28 17 11 (5个随机数)
 };
```
## 其他方法

| 方法               | 说明                                                   |
| :----------------- | :----------------------------------------------------- |
| Math.abs(x)        | x的绝对值                                              |
| Math.exp(x)        | e<sup>x</sup>                                          |
| Math.expm1(x)      | e<sup>x</sup>-1                                        |
| Math.log(x)        | Inx                                                    |
| Math.log1p(x)      | 1+Inx                                                  |
| Math.pow(x, power) | x<sup>power</sup>                                      |
| Math.pow(...nums)  | &radic;(a<sup>2</sup>+b<sup>2</sup>+...+n<sup>2</sup>) |
| Math.clz32(x)      | 返回32位整数x的前置零的数量                            |
| Math.sign(x)       | 返回表示x符号的1、0、-0或-1                            |
| Math.trunc(x)      | 取整                                                   |
| Math.sqrt(x)       | &radic;x                                               |
| Math.cbrt(x)       | <sup>3</sup>&radic;x                                   |
| Math.acos(x)       | cos<sup>-1</sup>x                                      |
| Math.acosh(x)      | 反双曲余弦                                             |
| Math.asin(x)       | sin<sup>-1</sup>x                                      |
| Math.asinh(x)      | 反双曲正弦                                             |
| Math.atan(x)       | tan<sup>-1</sup>x                                      |
| Math.atanh(x)      | 反双曲正切                                             |
| Math.atan2(y, x)   | tan<sup>-1</sup>(y/x)                                  |
| Math.sin(x)        | sin(x)                                                 |
| Math.cos(x)        | cos(x)                                                 |
| Math.tan(x)        | tan(x)                                                 |
