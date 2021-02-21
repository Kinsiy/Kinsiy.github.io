---
title: 集合引用类型(Object.2)-创建对象
author: Kinsiy
avatar: 'https://cdn.jsdelivr.net/gh/Kinsiy/cdn/img/custom/avatar.jpg'
authorLink: 'https://kinsiy.github.io'
authorAbout: I'm True
authorDesc: I'm True
categories: SKILL
comments: true
date: 2021-02-19 22:22:00
tags: [Notes,Javascript]
keywords:
description: 创建对象(Object)
photos:
---
# 概述
ECMAscript 5.1 并没有正式支持面向对象的结构，比如类或继承。ECMAscript 6 开始正式支持类和继承。ES6的类旨在完全涵盖之前规范设计的基于原型的继承模式。不过，无论从那个方面看，ES6的类都仅仅是封装了ES5.1构造函数加原型链继承的语法糖而已。
# 工厂模式
```javascript
function createPerson(name, age, job){
    let o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function(){
        console.log(this.name);
    };
    return o;
}

let person_1 = createPerson("Kinsiy",23,"Software Engineer");
let person_2 = createPerson("Greg",27,"Doctor");
```
工厂模式解决创建多个类似对象的问题，但没有解决对象标识问题(即创建的对象是什么类型)。
# 构造函数模式
```javascript
/* 按照惯例，构造函数名称的首字母是要大写的，非构造函数则以小写字母开头 */
function Person(name, age, job){            // 使用函数表达式亦可 let Person = function(){ ... }   
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){
        console.log(this.Name);
    }
}

let person_1 = new Person("Kinsiy",23,"Software Engineer");
let person_2 = new Person("Greg",27,"Doctor");

console.log(person_1.constructor == Person);    // true
console.log(person_1 instanceof Object);        // true     所有自定义对象都继承自Object
console.log(person_1 instanceof Person);        // true

console.log(person_1.sayName == person_2.sayName);  // false
```
实际上，Person()内部的代码跟createPerson()基本是一样的，只是有如下区别
* 没有显式的创建对象
* 属性和方法直接赋值给了this
* 没有return

要创建Person的实例，应使用new操作符。以这种方式调用构造函数会执行如下操作
* 在内存中创建一个新对象
* 这个新对象内部的[[Prototype]] 特性被赋值为构造函数的prototype属性
* 构造函数内部的this被赋值为这个新对象(即this指向新对象)
* 执行构造函数内部的代码(给新对象添加属性)
* 如果构造函数返回非空对象，则返回该对象；返回刚创建的新对象

## 构造函数也是函数
构造函数与普通函数唯一的区别就是调用方式不同。
```javascript
/* ... */

// 作为构造函数
let person = new Person("Kinsiy",23,"Software Engineer");
person.sayName();

// 作为函数调用
Person("Greg",27,"Doctor");     // 添加到window对象
window.sayName();

// 在另一个对象的作用域中调用
let o = new Object();
Person.call(o,"Type",20,"Nurse");
o.sayName()
```
## 构造函数的问题
构造函数定义的方法会在每个实例上都创建一遍。对前面的例子而言，person_1和person_2都有sayName()的方法，但这两个方法不是同一个Function实例。逻辑上讲，这个构造函数实际上是这样的：
```javascript
function Person(name, age, job){            // 使用函数表达式亦可 let Person = function(){ ... }   
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = new Function("console.log(this.name)");  // 逻辑等价
}
```
因此不同实例上的函数虽然同名却不相等。因为都是做一样的事，所以没必要定义两个不同的Function实例。况且，this对象可以把函数与对象的绑定推迟到运行时。要解决这个问题，可以吧函数定义转移到构造函数外部：
```javascript
function Person(name, age, job){            // 使用函数表达式亦可 let Person = function(){ ... }   
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = sayName;
}

function sayName(){
    console.log(this.name);
}

let person_1 = new Person("Kinsiy",23,"Software Engineer");
let person_2 = new Person("Greg",27,"Doctor");

person_1.sayName();     // Kissiy
person_2.sayName();     // Greg
console.log(person_1.sayName == person_2.sayName);      // true
```
# 原型模式
每个函数都会创建一个prototype属性，这个属性时一个对象，包含应该由特定引用类型的实例共享的属性和方法。实际上这个对象就是通过调用构造函数创建的对象的原型。使用原型的好处是，在它上面定义的属性和方法可以被对象实例共享。原来在构造函数中直接赋给对象实例的值，可以直接赋值给他们的原型。
```javascript
let Person = function() {};
Person.prototype.name = "Kinsiy";
Person.prototype.age = 24;
Person.prototype.jonb = "Software Engineer";
Person.prototype.sayName = function(){
    console.log(this.name);
};

let person_1 = new Person();
person_1.sayName();     // Kinsiy

let person_2 = new Person();
person_2.sayName();     // Kinsiy

console.log(person_1.sayName == person_2.sayName);      // true
```
## 理解原型
无论何时，只要创建一个函数，就会按照特定的规则为这个函数创建一个prototype属性(指向原型对象)。默认情况下，所有原型对象自动获得一个名为constructor的属性，指回与之关联的构造函数。对前面的例子而言，Person.prototype.constructor 指向Person。
在自定义构造函数时，原型对象默认只会获得constructor 属性，其他的所有方法都继承自Object。每次调用构造函数创建一个新实例，这个实例的内部[[Prototype]]指针就会被赋值为构造函数的原型对象。脚本中没有访问这个[[Prototype]]特性的标准方式，但Firefox、Safari和Chrome会在每个对象上暴露__proto__属性，通过这个属性可以访问对象的原型。
```javascript
function Person() {};

console.log(typeof Person.prototype);       // object
console.log(Person.prototype);

// {
//      constructor: ƒ Person()
//      __proto__: Object}
// }

console.log(Person.prototype.constructor == Person);  // true

// 正常的原型链都会终止与Object的原型对象

console.log(Person.prototype.__proto__ === Object.prototype);    // true
console.log(Person.prototype.__proto__.constructor === Object);  // true
console.log(Person.prototype.__proto__.__proto__ === null)       // true

/**
 * 实例通过__proto__链接到原型对象，
 * 它实际上指向隐藏特性[[Prototype]]
 * 
 * 构造函数通过prototype 属性链接到原型对象
 * 
 * 实例与构造函数没有直接联系，与原型对象有直接联系
 */
person_1 = new Person();
console.log(person_1.__proto__ === Person.prototype);       // true
console.log(person_1.__proto__.constructor === Person);     // true
```
可以使用isPrototypeOf()方法确定两个对象之间的[[Prototype]]关系。本质上isPrototypeOf()会在传入参数的[[Prototype]]指向调用它的对象室返回true。
```javascript
function Person() {};
person_1 = new Person;
console.log(Person.prototype.isPrototypeOf(person_1));      // true

/* ECMAScript 的Object类型有一个方法叫Object.getPrototypeOf()，返回参数的内部特性[[Prototype]]的值 */
console.log(Object.getPrototypeOf(person_1) == Person.prototype);   // true

/* Object类型还有一个setPrototyprOf()方法，可以向实例的[[Prototype]]写入一个新值。这样可以重写一个对象的原型继承关系 */
let object_1 = {
    num: 2
};

let person_2 = {
    name: "kinsiy"
};

Object.setPrototypeOf(person_2,object_1);
console.log(person_2.num);      // 2
console.log(Object.getPrototypeOf(person_2) === object_1);      // true

/* Object.setPrototyprOf()可能会严重影响代码性能，可以通过Object.create()来创建一个新对象，同时指定为原型 */
let object_2 = {
    num: 5
};

let person_3 = Object.create(object_2);
person_3.name = "Kinsiy";

console.log(person_3.num);      // 5
console.log(Object.getPrototypeOf(person_3) === object_2);      // true
```
## 原型层级


