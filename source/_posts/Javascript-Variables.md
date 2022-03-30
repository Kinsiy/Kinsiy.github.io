---
title: Javascript-变量、作用域与内存
date: 2021-01-10 14:15:42
description:
categories: [学习笔记, Javascript]
tags: [JS红宝书]
---

相比于其他语言，Javascript 中的变量可谓独树一帜。正如 ECMA-262 所规定的，Javascript 变量是松散类型的，而且变量不过就是特定时间点的一个特定值而已。由于没有规定变量必须包含什么数据类型，变量的值和数据类型在脚本生命期间可以改变。这样的变量很有意思，很强大，当然也有不少问题。本章会剖析错综复杂的变量。

<!-- more -->

## 原始值与引用值

EMCAScript 变量可以包含两种不同类型的数据：原始值和引用值。原始值就是最简单的数据，引用值则是由多个值构成的对象。再把一个值赋给变量是，Javascript 引擎必须确定这个值是原始值还是引用值。《语言基础》一章讨论了 6 种原始值：undefined、Null、Boolean、Number、String、Symbol。保存原始值的变量是按值访问的。引用值是保存在内存中的对象，Javascript 不允许直接访问内存位置，因此也就不能直接操作对象所在的内存空间。在操作对象时，实际上操作的是对该对象的引用。为此保存引用值的变量是按引用访问的。

### 动态属性

```javascript
/* 引用值 */
let person = new Object();
person.name = "Kinsiy";
console.log(person.name); // "Kinsiy"
/* 原始值 */
// 原始值不能有属性，尽管尝试给原始值添加属性不会报错
ler name = "Kinsiy";
name.age = 23;
console.log(name.age); // undefined
```

### 复制值

```javascript
/* 原始值复制 */
// num1 与 num2 完全独立
let num1 = 8;
let num2 = num1;

/* 引用值复制 */
// obj1 与 obj2 指向同一个对象
let obj1 = new Object();
let obj2 = obj1;
obj1.name = "Kinsiy";
console.log(obj2.name); // "Kinsiy"
```

### 传递参数

ECMAscript 中所有函数的参数都是按值传递的。在按值传递参数时，值会被复制到一个局部变量中。在按引用传递参数时，值在内存中的位置会被保存在一个局部变量中，这意味着对本地变量的修改会反映到函数外部。

```javascript
function addTen(num){
    num += 10;
    return num;
}

let count  = 20;
let result = addTen(count);
console.log(count); // 20
console.log(result); // 30

function setName(obj){
    obj.name = "Kinsiy";
    obj = New Object();
    obj.name = "Restituo";
}

let person = new Object();
setName(person);
console.log(person.name); // "Kinsiy"
```

### 确定类型

```javascript
let a = "Kinsiy";
let b = true;
let c = 56;
let d;
let e = null;
let f = new Object();

/* typeof */
console.log(typeof a); //String
console.log(typeof b); //boolean
console.log(typeof c); //number
console.log(typeof d); //undefined
console.log(typeof e); //object
console.log(typeof f); //object

/* instanceof 
    result = variable instanceof constructor
*/
let colors = ["粉色"];
let patter = /.d/;

console.log(f instanceof Object); //true
console.log(colors instanceof Array); //true
console.log(patter instanceof RegExp); //true
```

## 执行上下文与作用域

全局上下文是最外层的上下文。在浏览器中，全局上下文就是我们常说的 window 对象，因此所有通过 var 定义的全局变量和函数都会成为 window 对象的属性和方法。使用 let 和 const 的顶级声明不会定义在全局上下文中，但在作用域链解析上效果是一样的。

```javascript
var color = "bule";
function changeColor(){
    let anotherColor = "red";

    function swapColor(){
        ler tempColor = anotherColor;
        anotherColor = color;
        color = tempColor;

        // 可以访问color、anotherColor、tempColor
    }
    // 可以访问color、anotherColor, 无法访问tempColor
    swapColor();
}

// 只能访问color
changeColor();

/* 内部上下文可以通过作用域链访问外部上下文的一切，但外部上下文无法访问内部上下文中的任何东西。上下文之间的连接是线性的、有序的 */
```

### 作用域增强

```javascript
function testWith() {
  let qs = "?debug=ture";
  with (location) {
    let url = href + qs; // 相当于location.href
  }
  return url; // ReferenceError: url is not defined   使用let定义把url限制在了{}中，换做var可增强作用域
}
console.log(testWith());

/* try/catch 语句的catch块、with语句 会导致在作用域的前端临时添加一个上下文，这个上下文在代码执行后会被删除 */
```

### 变量声明

使用 var 的函数作用域声明<br>
在使用 var 声明变量时，变量会被自动添加到最接近的上下文。在函数中，最接近的上下文就是函数的局部上下文。在 with 语句中最接近的上下文也是函数上下文。如果变量未经声明就被初始化了那么它就会自动添加到全局上下文

```javascript
/* var */
function add(num1, num2) {
  var sum = num1 + num2; //若省略var，下方console.log不会报错
  return sum;
}

let result = add(10, 20);
console.log(sum); //ReferenceError: sum is not defined
/* 变量提升 */
console.log(name); //不会报错，而是undefined
var name = "Kinsiy";
```

使用 let 的快级作用域声明<br>
ES6 新增的 let 关键字与 var 很相似，但它作用域是块级的，这也是 Javascript 中的新概念。块级作用域由最近的一对花括号{}界定

```javascript
if (true) {
  let a;
}

{
  let b;
}

console.log(a); //ReferenceError: a is not defined
console.log(b); //ReferenceError: b is not defined

/* 不能重复声明 */
let c;
let c; //SyntaxError
```

使用 const 的常量声明<br>
使用 const 声明的变量必须同时初始化为某个值。一经声明，在其生命周期的任何时候都不能再重新赋予新值、

```javascript
const a; // SyntaxError
const b = 3;
console.log(b); // 3
b = 4; // TypeError
/* cosnt除了要遵守以上规则，其它方面与let声明一样
    const声明只应用到顶级原语或对象，换句话说，赋值为对象的const变量不能再被重新赋值为其他引用值，但对象的键则不收限制
    若想让整个对象都不能修改，可以使用Object.freeze(),这样再给属性赋值时虽然不会报错，但会静默失败
*/

const o1 = {};
o1 = {}; //TypeError

const o2 = {};
o2.name = "Kinsiy";
console.log(o2.name); //Kinsiy

const o3 = Object.freeze({});
o3.name = "Restituo";
console.log(o3.name); //undefined
```

### 标识符查找

```javascript
var color = "red";

function getColor() {
  let color = "pink";
  {
    let color = "green";
    return color;
  }
}

console.log(getColor()); //green
```

## 垃圾回收

Javascript 最常用的垃圾回收策略是标记清理。<br>
垃圾回收程序在运行的时候，会标记内存中存储的所有变量(标记方法有很多种)。然后，它会将所有在上下文中的变量，以及被在上下文中的变量引用的变量的标记去掉。在此之后再被加上标记的变量就是待删除的了，原因是任何在上下文中的变量都访问不到它们了。随后垃圾回收程序做一次内存清理，销毁带标记的所有值并回收它们的内存。

### 内存管理

将内存占用量保持在一个较小的值可以让页面性能更好。优化内存占用的最佳手段就是保证在执行代码时只保存必要的数据。如果数据不在必要，那么把它设置为 null，从而释放其引用。这也可以叫做解除引用。这个建议最适合全局变量和全局对象。局部变量在超出作用域后会被自动解除引用

```javascript
function createPerson(name){
    let locatPerson = new Object();
    locatPerson.name = name;
    return localPerson;
}

let grobalPerson = createPerson("Kinsiy");
//  ....

grobalPerson = null;        //解除grobalPerson 对值得引用

/* 通过const和let 声明提升性能
    因为const和let 都以块(而非)函数为作用域，所以相比于var，使用这两个新关键字可能会更早的让垃圾回收程序介入,尽早的回收应该回收的内存
*/


/* 隐藏类和删除操作
    V8 Javascript 引擎
    运行期间，V8 会将创建的对象和隐藏类关联起来，以跟踪它们的属性特征。能够共享相同隐藏类的对象性能会更好，V8 会针对这种情况进行优化，但不一定总能够做到
*/

function Article(opt_authour){
    this.title = "V8 Javascript";
    this.authour = opt_authour;     //避免“先创建再补充”式的动态属性赋值，并在构造函数中一次性声明所有属性
}

let a1 = new Article("someone");
let a2 = new Article("otherSomeone");

// delete a1.author                 //避免使用delete ,最佳实践是把不想要的属性设置为null
a1.author = null;

/* 内存泄露 */
function setName(){
    name = "Kinsiy";                //最常见，解释器会把变量name当做window属性来创建。
}

let name = "Kinsiy";
setInterval(() => {
    console.log(name);
}, 100;)                            //定时器导致内存泄露

let outer = function(){
    let name = "Kinsiy";
    return function(){
        return name;
    };
};                                  //闭包导致内存泄露

/* 静态分配与对象池 */

//略
```
