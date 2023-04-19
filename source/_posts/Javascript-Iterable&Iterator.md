---
title: Javascript-迭代器与生成器
date: 2021-02-03 01:13:43
categories: [学习笔记, Javascript]
tags: [ Iterator]
keywords:
description:
photos:
---

## 理解迭代

在 Javascript 中计数循环就是一种最简单的迭代。循环是迭代的基础，这是因为它可以指定迭代的次数，以及每次迭代要执行什么操作。每次循环都会在下一次迭代开始之前完成，而每次迭代的顺序都是事先定义好的。
迭代会在一个有序集合上进行。("有序"可以理解为集合中所有的项都可以按照既定的顺序被遍历到，特别是开始和结束都有明确的定义。)数组是 Javascript 中有序集合的最典型的例子。

<!-- more -->

```javascript
/* 最简单的迭代，计数循环 */
for (let i = 1; i <= 10; i++){
    console.log(i);     // 1,2,...,10
}

let arr = ["Python","Javascript","Java"]；
for (let index = 0; index < arr.length; i++){
    console.log(arr[index]);    // "Python" "Javascript" "Java"
}

/* ES5 新增了Array.prototype.forEach()方法，想通用迭代需求迈进了一步(但仍不够理想) */
arr.forEach((item) => console.log(item))    // // "Python" "Javascript" "Java"
```

## 迭代器模式

迭代器模式(特别是 ECMAScript 这个语境下)描述了一个方案，即可以把有些结构称为“可迭代对象”(iterable)，因为它们实现了正式的 Iterable 接口，而且可以通过迭代器 Iterator 消费。
可迭代对象是一种抽象的说法。基本上，可以把可迭代对象理解成数组或集合这样的集合类型的对象。它们包含的元素都是有限的，而且都具有无歧义的遍历顺序。迭代器无需了解与其关联的可迭代对象的结构，只需知道如何取得连续的值。

### 可迭代协议

实现 Iterale 接口(可迭代协议)要求同时具备两种能力：支持迭代的自我识别能力和创建实现 Iterator 接口的对象的能力。在 ECMAScript 中，这意味着必须暴露一个属性作为"默认迭代器"，而且这个属性必须使用特殊的 Symbol.iterator 作为键。这个默认迭代器属性必须引用一个迭代器工厂函数，调用这个工厂函数必须返回一个新迭代器。很多内置类型都实现了 Iterable 接口：

-   字符串
-   数组
-   映射
-   集合
-   arguments 对象
-   NodeList 等 DOM 集合类型

```javascript
/* 检查是否存在默认迭代器属性可以暴露这个工厂函数 */
let num = 8;
let obj = {};
let str = "Kinsiy";
let arr = ["Python", "Javascript"];
let map = new Map().set("a", 1).set("b", 2);
let set = new Set().add("c", 3).add("d", 4);
let els = document.querySelectorAll("div");

console.log(num[Symbol.iterator]); // undefined
console.log(obj[Symbol.iterator]); // undefined
console.log(str[Symbol.iterator]); // ƒ [Symbol.iterator]() { [native code] }
console.log(arr[Symbol.iterator]); // ƒ values() { [native code] }
console.log(map[Symbol.iterator]); // ƒ entries() { [native code] }
console.log(set[Symbol.iterator]); // ƒ values() { [native code] }
console.log(els[Symbol.iterator]); // ƒ values() { [native code] }

/* 调用工厂函数 */
console.log(str[Symbol.iterator]()); // StringIterator {}
console.log(arr[Symbol.iterator]()); // Array Iterator {}
console.log(map[Symbol.iterator]()); // MapIterator {"a" => 1, "b" => 2}
console.log(set[Symbol.iterator]()); // SetIterator {"c", "d"}
console.log(els[Symbol.iterator]()); // Array Iterator {}
```

实际写代码过程中,不需要显式调用这个工厂函数来生成迭代器。实现可迭代协议的所有类型都会自动兼容接收可迭代对象的任何语言特性。接收可迭代对象的原生语言特性包括：

-   for - of 循环
-   数组解构
-   扩展操作符
-   Array.from()
-   创建集合
-   创建映射
-   Promise.all()接收由期约组成的可迭代对象
-   Promise.race()接收由期约组成的可迭代对象
-   yield\* 操作符，在生成器中使用

这些原生语言结构会在后台调用提供的可迭代对象的这个工厂函数，从而创建一个迭代器：

```javascript
arr = ["Python", "C", "Javascript"];
// 数组结构
let [a, b, c] = arr;
console.log(a, b, c); // Python C Javascript

// Map 构造函数
let pairs = arr.map((x, i) => [x.toUpperCase(), i]);
console.log(pairs); // [["PYTHON", 0], ["C", 1], ["JAVASCRIPT", 2]]
let m = new Map(pairs);
console.log(m); // {"PYTHON" => 0, "C" => 1, "JAVASCRIPT" => 2}

/* 如果对象原型链上的父类实现了Iterable接口，那这个对象也就实现是这个接口 */
class KinsiyArray extends Array {}
let KinsiyArr = new KinsiyArray("Html", "Xml", "Hxml");
for (let el of KinsiyArr) {
  console.log(el); // Html Xml Hxml
}
```

### 迭代器协议

迭代器是一种一次性使用的对象，用于迭代与其关联的可迭代对象。迭代器 API 使用 next()方法在可迭代对象中遍历数据。每次成功调用 next()，都会返回一个 IteratorResult 对象，其中包含迭代器返回的下一个值。若不调用 next()，则无法知道迭代器的当前位置。

```javascript
// 可迭代对象
let arr = ["Javascript", "C", "Python"];

// 迭代器工厂函数
console.log(arr[Symbol.iterator]); // ƒ values() { [native code] }

// 迭代器
let iter = arr[Symbol.iterator]();
console.log(iter); // Array Iterator {}

// 执行迭代
console.log(iter.next()); // {value: "Javascript", done: false}
console.log(iter.next()); // {value: "C", done: false}
console.log(iter.next()); // {value: "Python", done: false}
console.log(iter.next()); // {value: undefined, done: true}

/* done: true状态称为"耗尽"。只要迭代器到达done: true状态，后续调用next()就一直返回同样的值了 */
console.log(iter.next()); // {value: undefined, done: true}

// 不同迭代器的实例相互之间没有联系
let iter_1 = arr[Symbol.iterator]();
let iter_2 = arr[Symbol.iterator]();
console.log(iter_1.next()); // {value: "Javascript", done: false}
console.log(iter_2.next()); // {value: "Javascript", done: false}

// 可迭代对象在迭代期间被修改了，迭代器也会反映相应的变化
arr.splice(1, 0, "Java");
console.log(iter_1.next()); // {value: "Java", done: false}
```

### 自定义迭代器

```javascript
class Kinsiy {
  constructor(limit) {
    this.limit = limit;
  }

  [Symbol.iterator]() {
    let count = 1,
      limit = this.limit;
    return {
      next() {
        if (count < limit) {
          return { done: false, value: count++ };
        } else {
          return { done: true, value: undefined };
        }
      },
    };
  }
}

let me = new Kinsiy(3);

for (let i of me) {
  console.log(i); // 1, 2
}
```

### 提前终止迭代器

可选的 return()方法用于指定迭代器提前关闭时执行的逻辑。执行迭代的结构在想让迭代器知道它不想遍历到可迭代对象耗尽时，就可以关闭迭代器。可能的情况包括：

-   for - of 循环通过 break, continue, return, thorw 提前退出
-   解构操作并未消费所有值

```javascript
let counter = ['A','B','C','D'];

try {
    for (let chat of counter){
        if (chat == "B"){
            throw "err"
        }
        console.log(chat);      // A    提前退出
    }
}catch(){};
```

## 生成器

生成器是 EMCAScript 6 新增的一个极为灵活的结构，拥有在一个函数块内暂停和恢复代码执行的能力。

### 生成器基础

生成器的形式是一个函数，函数名称前面加一个星号(\*)表示它是一个生成器。只要是可以定义函数的地方，就可以定义生成器

```Javascript
/* 标识生成器函数的星号不受两侧空格影响 */
// 生成器函数声明
function* generatorFn(){};

// 生成器函数表达式
let generatorF = function* (){};

// 作为对象字面量方法的生成器函数
let foo = {
    * generator_f(){};
}

// 作为类实例方法的生成器函数
class foo = {
    * generator_Fn(){};
}

/* 箭头函数不能用来定义生成器函数 */
```

调用生成器函数会产生一个生成器对象。生成器对象一开始处于暂停执行(suspended)的状态。与迭代器相似，生成器对象也实现了 Iterator 接口，因此具有 next()方法。调用这个方法会让生成器开始或恢复执行。

```javascript
function* generatorFn() {}

const g = generatorFn();

console.log(g); // generatorFn {<suspended>}
console.log(g.next); // ƒ next() { [native code] }

/* next()方法的返回值类似于迭代器，有一个done属性和一个value属性。
    函数体为空的生成器函数中间不会停留，调用一次next()就会让生成器达到done: true 状态
    value属性是生成器函数的返回值，默认值为undefined，可以通过生成器函数的返回值指定
 */
console.log(g.next()); // {value: undefined, done: true}

function* generatorFn_1() {
  return "foo";
}

let g_1 = generatorFn_1();
console.log(g_1); // generatorFn_1 {<suspended>}
console.log(g_1.next()); // {value: "foo", done: true}

/* 生成器函数只会在初次调用next()方法后开始执行 */
function* generatorFn_2() {
  console.log("running");
}

// 初次调用生成器函数并不会打印日志
let g_2 = generatorFn_2();
g_2.next(); // running

/* 生成器对象实现了Iterator 接口，他们默认的迭代器是自引用的 */
console.log(generatorFn); // ƒ* generatorFn(){}
console.log(generatorFn()[Symbol.iterator]); // ƒ [Symbol.iterator]() { [native code] }
console.log(g === g[Symbol.iterator]()); // true
```

### 通过 yield 中断执行

yield 关键字可以让生成器停止和开始执行，也是生成器最有用的地方。生成器函数在遇到 yield 关键字之前会正常执行。遇到这个我关键字后，执行会停止，函数作用域的状态会被保留。停止执行的生成器函数只能通过在生成器对象上调用 next()方法来恢复执行：

```javascript
/* 通过yield 关键字退出的生成器函数会处在done: false 状态; 通过return关键字退出的生成器函数会处于done: true 状态 */
function* generatorFn() {
  yield "Kinsiy";
  yield "Hey";
  return "Live";
}

let g = generatorFn();

console.log(g.next()); // {value: "Kinsiy", done: false}
console.log(g.next()); // {value: "Hey", done: false}
console.log(g.next()); // {value: "Live", done: true}

/* 在一个生成器对象上调用next()不会影响其他生成器
    yield 关键字必须直接位于生成器函数定义中，出现在嵌套的非生成器函数中会抛出语法错误
 */
// 有效
function* gen_1() {
  yield;
}
// 无效
function* gen_2() {
  function a() {
    yield;
  }
}
// [ chrome(88.0.4324.146)未见报错，但yield 确实无效 ]
```

#### 生成器对象作为可迭代对象

```javascript
function* genertorFn() {
  for (let i = 0; i < 3; i++) {
    yield i;
  }
}

for (let x of genertorFn()) {
  console.log(`x is : ${x}`);
}
// x is : 0
// x is : 1
// x is : 2
```

#### 使用 yield 实现输入输出

除了可以作为函数的中间返回语句使用，yield 关键字还可以作为函数的中间参数使用。上一次让生成器函数暂停的 yield 关键字会接收到传给 next()方法的第一个值。
第一次调用的 next()传入的值不会被使用，因为这一次调用是为了开始执行生成器函数：

```javascript
function* genertorFn(initial) {
  console.log(initial);
  console.log(yield);
  console.log(yield);
}

let g = genertorFn("Chinese");

console.log(g.next("Test")); // Chinese
console.log(g.next("English")); // English

/* 同时用于输入和输出 */
function* genertorFn_1() {
  return yield "Kinsiy";
}

let g_1 = genertorFn_1();
console.log(g_1.next()); // {value: "Kinsiy", done: false}
console.log(g_1.next("Queen")); // {value: "Queen", done: true}
```

#### 产生可迭代对象

可以使用星号增强 yield 的行为，让它能够迭代一个可迭代对象，从而一次产出一个值：

```javascript
function* genertorFn_1() {
  yield* [6, 7];
  return 8;
}

/* yield* 的值是关联迭代器返回done: true 时的value的值。
    使用console.log(yield*)的原因是 for-of 循环等内置语言结构会忽略状态为done: true 的IteratorObject 内部返回的值 
 */
function* genertorFn_2() {
  yield* [1, 2, 3, ,];
  console.log(yield* [4, 5]); // 打印yield*
  console.log(yield* genertorFn_1()); // 打印yield*
}

for (let x of genertorFn_2()) {
  console.log(x);
}
// 1
// 2
// 3
// undefined
// 4
// 5
// undefined        对于普通迭代器来说，这个值是undefined
// 6
// 7
// 8                对于生成器函数产生的迭代器来说，这个值就是生成器函数返回的值
```

#### 使用 yield\* 实现递归算法

```javascript
function* nTimes(n) {
  if (n > 0) {
    yield* nTimes(n - 1);
    yield n - 1;
  }
}

for (let x of nTimes(3)) {
  console.log(x); // 0 1 2
}
```

遍历图数据结构示例(暂略)。

### 生成器作为默认迭代器

因为生成器对象实现了 Iterator 接口，而且生成器函数和默认迭代器被调用之后都产生迭代器，所以生成器格外适合作为默认迭代器。

```Javascript
class Foo {
    constructor(){
        this.values = [1, 2, 3];
    }

    * [Symbol.iterator](){
        yield* this.values;
    }
}

const f = new Foo();
for (let x of f){
    console.log(x); // 1 2 3
}
```

### 提前终止生成器

与迭代器类似，生成器也支持“可关闭”的概念。一个实现 Iterator 接口的对象一定有 next()方法，还有一个可选的 return() 方法用于提前终止迭代器。生成器对象除了有这两个方法，还有第三个方法 throw()。

#### return()

retrun()方法会强制生成器进入关闭状态。提供给 return()方法的值，就是终止迭代器对象的值。

```javascript
function* genertorFn() {
  yield* [1, 2, 3, 4, 5];
}

const g = genertorFn();
console.log(g); // genertorFn {<suspended>}
console.log(g.next()); // {value: 1, done: false}
console.log(g.return("Kinsiy")); // {value: Kinsiy, done: true}
console.log(g.next()); // {value: undefined, done: true}
console.log(g); // genertorFn {<closed>}
```

#### throw()

throw()方法会在暂停的时候将一个提供的错误注入到生成器对象中。如果错误未被处理，生成器就会关闭。

```javascript
function* genertorFn_1() {
  yield* [1, 2, 3];
}

const g = genertorFn_1();
console.log(g); // genertorFn_1 {<suspended>}
try {
  g.throw("Kinsiy");
} catch (e) {
  console.log(e); // Kinsiy
}
console.log(g); // genertorFn_1 {<closed>}
```

不过，假如生成器函数内部处理了这个错误，那么生成器就不会关闭，而且还可以恢复执行。错误处理会跳过对应的 yield

```javascript
function* genertorFn_2() {
  for (let x of [4, 5, 6]) {
    try {
      yield x;
    } catch (e) {
      // yield e
    }
  }
}

const g = genertorFn_2();
console.log(g.next()); // {value: 4, done: false}
g.throw("Kinsiy");
console.log(g.next()); // {value: 6, done: false}

/* console.log(g.throw("Kinsiy"));

    genertorFn_2() 不yield e 
    {value: 5, done: false}
    反之 yield e 
    {value: "Kinsiy", done: false}
 */
```

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
