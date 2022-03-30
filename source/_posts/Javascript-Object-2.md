---
title: Javascript-Object.2[创建对象]
date: 2021-02-19 22:22:00
categories: [学习笔记, Javascript]
tags: [JS红宝书, Object]
keywords:
description:
photos:
---

# 概述

ECMAscript 5.1 并没有正式支持面向对象的结构，比如类或继承。ECMAscript 6 开始正式支持类和继承。ES6 的类旨在完全涵盖之前规范设计的基于原型的继承模式。不过，无论从那个方面看，ES6 的类都仅仅是封装了 ES5.1 构造函数加原型链继承的语法糖而已。

<!-- more -->

# 工厂模式

```javascript
function createPerson(name, age, job) {
  let o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function () {
    console.log(this.name);
  };
  return o;
}

let person_1 = createPerson("Kinsiy", 23, "Software Engineer");
let person_2 = createPerson("Greg", 27, "Doctor");
```

工厂模式解决创建多个类似对象的问题，但没有解决对象标识问题(即创建的对象是什么类型)。

# 构造函数模式

```javascript
/* 按照惯例，构造函数名称的首字母是要大写的，非构造函数则以小写字母开头 */
function Person(name, age, job) {
  // 使用函数表达式亦可 let Person = function(){ ... }
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function () {
    console.log(this.Name);
  };
}

let person_1 = new Person("Kinsiy", 23, "Software Engineer");
let person_2 = new Person("Greg", 27, "Doctor");

console.log(person_1.constructor == Person); // true
console.log(person_1 instanceof Object); // true     所有自定义对象都继承自Object
console.log(person_1 instanceof Person); // true

console.log(person_1.sayName == person_2.sayName); // false
```

实际上，Person()内部的代码跟 createPerson()基本是一样的，只是有如下区别

-   没有显式的创建对象
-   属性和方法直接赋值给了 this
-   没有 return

要创建 Person 的实例，应使用 new 操作符。以这种方式调用构造函数会执行如下操作

-   在内存中创建一个新对象
-   这个新对象内部的[[Prototype]] 特性被赋值为构造函数的 prototype 属性
-   构造函数内部的 this 被赋值为这个新对象(即 this 指向新对象)
-   执行构造函数内部的代码(给新对象添加属性)
-   如果构造函数返回非空对象，则返回该对象；返回刚创建的新对象

## 构造函数也是函数

构造函数与普通函数唯一的区别就是调用方式不同。

```javascript
/* ... */

// 作为构造函数
let person = new Person("Kinsiy", 23, "Software Engineer");
person.sayName();

// 作为函数调用
Person("Greg", 27, "Doctor"); // 添加到window对象
window.sayName();

// 在另一个对象的作用域中调用
let o = new Object();
Person.call(o, "Type", 20, "Nurse");
o.sayName();
```

## 构造函数的问题

构造函数定义的方法会在每个实例上都创建一遍。对前面的例子而言，person_1 和 person_2 都有 sayName()的方法，但这两个方法不是同一个 Function 实例。逻辑上讲，这个构造函数实际上是这样的：

```javascript
function Person(name, age, job) {
  // 使用函数表达式亦可 let Person = function(){ ... }
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = new Function("console.log(this.name)"); // 逻辑等价
}
```

因此不同实例上的函数虽然同名却不相等。因为都是做一样的事，所以没必要定义两个不同的 Function 实例。况且，this 对象可以把函数与对象的绑定推迟到运行时。要解决这个问题，可以吧函数定义转移到构造函数外部：

```javascript
function Person(name, age, job) {
  // 使用函数表达式亦可 let Person = function(){ ... }
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = sayName;
}

function sayName() {
  console.log(this.name);
}

let person_1 = new Person("Kinsiy", 23, "Software Engineer");
let person_2 = new Person("Greg", 27, "Doctor");

person_1.sayName(); // Kissiy
person_2.sayName(); // Greg
console.log(person_1.sayName == person_2.sayName); // true
```

# 原型模式

每个函数都会创建一个 prototype 属性，这个属性时一个对象，包含应该由特定引用类型的实例共享的属性和方法。实际上这个对象就是通过调用构造函数创建的对象的原型。使用原型的好处是，在它上面定义的属性和方法可以被对象实例共享。原来在构造函数中直接赋给对象实例的值，可以直接赋值给他们的原型。

```javascript
let Person = function () {};
Person.prototype.name = "Kinsiy";
Person.prototype.age = 24;
Person.prototype.jonb = "Software Engineer";
Person.prototype.sayName = function () {
  console.log(this.name);
};

let person_1 = new Person();
person_1.sayName(); // Kinsiy

let person_2 = new Person();
person_2.sayName(); // Kinsiy

console.log(person_1.sayName == person_2.sayName); // true
```

## 理解原型

无论何时，只要创建一个函数，就会按照特定的规则为这个函数创建一个 prototype 属性(指向原型对象)。默认情况下，所有原型对象自动获得一个名为 constructor 的属性，指回与之关联的构造函数。对前面的例子而言，Person.prototype.constructor 指向 Person。
在自定义构造函数时，原型对象默认只会获得 constructor 属性，其他的所有方法都继承自 Object。每次调用构造函数创建一个新实例，这个实例的内部[[Prototype]]指针就会被赋值为构造函数的原型对象。脚本中没有访问这个[[Prototype]]特性的标准方式，但 Firefox、Safari 和 Chrome 会在每个对象上暴露**proto**属性，通过这个属性可以访问对象的原型。

```javascript
function Person() {}

console.log(typeof Person.prototype); // object
console.log(Person.prototype);

// {
//      constructor: ƒ Person()
//      __proto__: Object}
// }

console.log(Person.prototype.constructor == Person); // true

// 正常的原型链都会终止与Object的原型对象

console.log(Person.prototype.__proto__ === Object.prototype); // true
console.log(Person.prototype.__proto__.constructor === Object); // true
console.log(Person.prototype.__proto__.__proto__ === null); // true

/**
 * 实例通过__proto__链接到原型对象，
 * 它实际上指向隐藏特性[[Prototype]]
 *
 * 构造函数通过prototype 属性链接到原型对象
 *
 * 实例与构造函数没有直接联系，与原型对象有直接联系
 */
person_1 = new Person();
console.log(person_1.__proto__ === Person.prototype); // true
console.log(person_1.__proto__.constructor === Person); // true
```

可以使用 isPrototypeOf()方法确定两个对象之间的[[Prototype]]关系。本质上 isPrototypeOf()会在传入参数的[[Prototype]]指向调用它的对象室返回 true。

```javascript
function Person() {}
person_1 = new Person();
console.log(Person.prototype.isPrototypeOf(person_1)); // true

/* ECMAScript 的Object类型有一个方法叫Object.getPrototypeOf()，返回参数的内部特性[[Prototype]]的值 */
console.log(Object.getPrototypeOf(person_1) == Person.prototype); // true

/* Object类型还有一个setPrototyprOf()方法，可以向实例的[[Prototype]]写入一个新值。这样可以重写一个对象的原型继承关系 */
let object_1 = {
  num: 2,
};

let person_2 = {
  name: "kinsiy",
};

Object.setPrototypeOf(person_2, object_1);
console.log(person_2.num); // 2
console.log(Object.getPrototypeOf(person_2) === object_1); // true

/* Object.setPrototyprOf()可能会严重影响代码性能，可以通过Object.create()来创建一个新对象，同时指定为原型 */
let object_2 = {
  num: 5,
};

let person_3 = Object.create(object_2);
person_3.name = "Kinsiy";

console.log(person_3.num); // 5
console.log(Object.getPrototypeOf(person_3) === object_2); // true
```

## 原型层级

在通过对象访问属性时，会按照这个属性的名称开始搜索。搜索开始于对象实例本身。如果在这个实例上发现了给定的名称，则返回该名称对应的值。如果没有找到这个属性，则搜索会沿着指针进入原型对象，然后在原型对象上找到属性后，再返回对应的值。
虽然可以通过实例原型读取原型对象上的值，但不可能通过实例重写这些值。如果在实例上添加了一个与原型对象中同名的属性，那就会在实例上创建这个属性，这个属性会遮蔽原型对象上的属性。

```javascript
function Person() {}

Person.prototype.name = "Kinsiy";
Person.prototype.age = 23;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function () {
  console.log(this.name);
};

let person_1 = new Person();
let person_2 = new Person();

person_1.name = "Restituo";
console.log(person_1.name); // "Restituo"
console.log(person_2.name); // "Kinsiy"

/// hasOwnProperty() 方法
console.log(person_1.hasOwnProperty("name")); // true
console.log(person_2.hasOwnProperty("name")); // false

/// delete 操作符
delete person_1.name;
console.log(person_1.name); // "Kinsiy"

console.log(person_1.hasOwnProperty("name")); // false
console.log(person_2.hasOwnProperty("name")); // false
```

只要给对象实例添加了一个属性，这个属性就会遮蔽(shadow)原型对象上的同名属性，也就是虽然不会修改它，但会屏蔽对它的访问。即使在实例上把这个属性设置为 null，也不会恢复它和原型的联系。不过使用 delete 操作符可以完全删除实例上的这个属性，从而让标识符解释过程能够继续搜索原型对象。
hasOwnProperty()方法用于确定某个属性在实例上还是在原型对象上。这个方法是继承自 Object 的，会在属性存在于调用他的对象实例上时返回 true。

## 原型和 in 操作符

有两种方式使用 in 操作符：单独使用和在 for-in 循环中使用。在单独使用时，in 操作符会在可以通过对象访问指定属性时放回 true，无论该属性是在实例上还是在原型上。

```javascript
...

/**
 * @description: 检查属性是否在原型上未被遮蔽
 * @param {*} object
 * @param {*} name
 * @return {*} boolean
 */
function hasPrototyProperty(object, name) {
    return !object.hasOwnProperty(name) && name in object;
}

console.log(hasPrototyProperty(person_1, "name"));      // "false"
console.log(hasPrototyProperty(person_2, "name"));      // "true"
```

在 for-in 循环中使用 in 操作符时，可以通过对象访问且可以被枚举的属性都会返回，包括实例属性和原型属性。遮蔽原型中不可枚举([[Enumerable]]特性被设置为 false)属性的实例属性也会在 for-in 循环中返回，因为默认情况下开发者定义的属性都是可以枚举的。
要获取对象上所有可枚举的实例属性，可以使用 Object.keys()方法。这个方法接收一个对象作为参数，返回包含该对象所有可枚举属性名称的字符串数组。
如果想列出所有实例属性，无论是否可以枚举，都可以使用 Object.getOwnPropertyNames()

```javascript
...
for (const key in person_1) {
    console.log(key);   // name age job sayName
}

/// Object.keys()

let keys = Object.keys(Person.prototype);
console.log(keys);  // ["name", "age", "job", "sayName"]
keys = Object.keys(person_1);
console.log(keys);  // ["name"]

/// Object.getOwnPropertyNames()
keys = Object.getOwnPropertyNames(Person.prototype);
console.log(keys);  //  ["constructor", "name", "age", "job", "sayName"]
```

在 ECMAscript 6 新增符号类型之后，相应的出现了一个 Object.getOwnPropertyNames()的兄弟方法的需求，因为以符号为键的属性没有名称的概念。因此，Object.getOwnPropertySymbols()方法就出现了，这个方法与 Object.getOwnPropertyNames()类似，只是针对符号而已。

## 属性枚举顺序

for-in 循环、Object.keys()、Object.getOwnPropertyNames()、Object.getOwnPropertySymbols()以及 Object.assgin()在属性枚举顺序方面有很大区别。for-in 循环和 Object.keys()的枚举顺序是不确定的，取决于 Javascript 引擎，可能因浏览器而异。
Object.getOwnPropertyNames()、Object.getOwnPropertySymbols()以及 Object.assgin()的枚举顺序是确定的。先以升序枚举数值键，然后以插入顺序枚举字符串和符号键。

```javascript
let k1 = Symbol("k1");
let k2 = Symbol("k2");

let o = {
  1: 1,
  first: "first",
  [k1]: "sym1",
  second: "second",
  0: 0,
};
o[k2] = "sym2";
o[3] = 3;
o.third = "third";
o[2] = 2;

console.log(Object.getOwnPropertyNames(o)); // ["0", "1", "2", "3", "first", "second", "third"]
console.log(Object.getOwnPropertySymbols(o)); // [Symbol(k1), Symbol(k2)]
```

# 对象迭代

在 Javascript 有史以来的大部分时间内，迭代对象属性都是一个难题。ECMAscript 2017 新增了两个静态方法，用于将对象内容转换为序列化的——更重要的是可迭代的——格式。这两个静态方法 Object.values()和 Object.entries()接收一个对象，返回它们内容的数组。Object.values()返回对象值得数组，Object.entries()方法返回键/值对的数组。

```javascript
const o = {
  foo: "bar",
  baz: 1,
  qux: {},
};

console.log(Object.values(o)); // ["bar", 1, {…}]
console.log(Object.entries(o)); // [["foo", "bar"], ["baz", 1], ["qux", {…}]]
```

注意，非字符串属性会被转换为字符串输出。另外，这两个方法执行对象的浅复制。符号属性会被忽略。

## 其他原型语法

在前面的例子中，每次定义一个属性或方法都会把 Person.prototype 重写一遍。为了减少代码冗余，也为了从视觉上更好的封装原型功能，直接通过一个包含所有属性和方法的对象字面量来重写原型成为了一种常见的做法。

```javascript
function Person() {};

Person.prototype = {
    name: "Kinsiy",
    age: 23,
    sayName(){
        console.log(this.name);
    }
}

/**
* 这样重写之后，Person.property的constructor属性就不指向Person了。
*/
let person_1 = new Person;
console.log(person_1 instanceof Object);
console.log(person_1 instanceof Person);

console.log(person_1.constructor == Person);
console.log(person_1.constructor == Object);


/**
* 如果constructor的值很重要，则可以在重写原型对象时专门设置一下它的值
* Person.prototype = {
*    constructor: Person,
*    name: "Kinsiy",
*    age: 23,
*    sayName(){
*        console.log(this.name);
*    }
* }
* 但要注意，以这种方式恢复的constructor属性会创建一个[[Enumerable]]为true的属性。而原生constructor属性默认是不可枚举的
* 可以改为使用Object.defineProperty()方法来定义constructor 属性
*/

Object.defineProperty(Person.property,"contructor"){
    enumerable: false,
    value: Person
};
```

## 原型的动态性

因为从原型上搜索值得过程是动态的，所以即使实例在修改原型之前已经存在，任何时候对原型对象所做的修改也会在实例上反映出来。

```javascript
function Person() {}

let person_1 = new Person();

Person.prototype.sayHi = function () {
  console.log("Hi");
};

person_1.sayHi(); // "Hi"
```

这与重写这个原型是两回事。实例的[[prototype]]指针是在调用构造函数时自动赋值的，这个指针即使把原型修改为不同的对象也不会变。重写整个原型会切断最初原型与构造函数的联系，但实例引用的仍然是最初的原型。

```javascript
function Person() {}

let person_1 = new Person();

Person.prototype = {
  sayHi() {
    console.log("Hi");
  },
};

person_1.sayHi(); // TypeError
```

## 原生对象原型

所有原生引用类型的构造函数(包括 Object、Array、String 等)都在原型上定义了实例方法。比如，数组实例的 sort 方法就是 Array.prototype 上定义的。

```javascript
String.prototype.Kinsiy = function () {
  console.log("Kinsiy");
};

const str = "abc";

str.Kinsiy();
```

## 原型的问题

```javascript
function Person() {}
Person.prototype.friends = ["Kinsiy", "Restituo"];

let person_1 = new Person();
let person_2 = new Person();

person_1.friends.push("Type57");

console.log(person_1.friends); // ["Kinsiy", "Restituo", "Type57"]
console.log(person_2.friends); // ["Kinsiy", "Restituo", "Type57"]
```
