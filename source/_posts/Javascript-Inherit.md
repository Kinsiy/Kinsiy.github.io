---
title: Javascript - 继承方式
date: 2021-03-02 19:54:11
categories: [学习笔记, Javascript]
tags: [Classes]
keywords:
description:
photos:
---

## 原型链

ECMA-262 把原型链定义为 ECMAScript 的主要继承方式。其基本思想就是通过原型继承多个引用类型的属性和方法。

```javascript
// 后续代码以次段为基础
function SuperType() {
  this.property = true;
}

SuperType.prototype.getSuperValue = function () {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}

SubType.prototype = new SuperType(); // SubType 的原型是另一个类型的实例

SubType.prototype.getSubValue = function () {
  return this.subproperty;
};

let instance = new SubType();

console.log(instance.getSuperValue()); // true
```

<!--more-->

### 默认原型

实际上，原型链中还有一环。默认情况下，所有引用类型都继承自 Object，这也是通过原型链实现的。任何函数的默认原型都是一个 Object 的实例，这意味着这个实例有一个内部指针指向 Object.prototype。这也是为什么自定义类型能够继承包括 toString(),valueOf()在内的所有默认的方法的原因。

### 原型与继承关系

原型与实例的关系可以通过两种方式来确定。第一种方式是使用 instanceof 操作符，如果一个实例的原型链中出现过相应的构造函数，则 instanceof 返回 true。
第二种方法是使用 isPrototypeOf()方法。原型链中的每个原型都可以调用这个方法，只要原型链中包含这个原型，这个方法就返回 true。

```javascript
// 实例的原型链中出现过相应的构造函数
console.log(instance instanceof Object); // true
console.log(instance instanceof SuperType); // true
console.log(instance instanceof SubType); // true

// 只要原型链中包含这个原型
console.log(Object.prototype.isPrototypeOf(instance)); // true
console.log(SuperType.prototype.isPrototypeOf(instance)); // true
console.log(SubType.prototype.isPrototypeOf(instance)); // true

// 直接调用返回false SubType.isPrototypeOf(instance) false
```

### 关于方法

子类有时候需要覆盖父类的方法，或者增加父类没有的方法。为此，这些方法必须在原型赋值之后再添加到原型上。

```javascript
// 覆盖已有的方法
SubType.prototype.getSuperValue = function () {
  return false;
};

let instance = new SubType();

console.log(instance.getSuperValue()); // false
```

以对象字面量方式创建原型方法会破坏之前的原型链，这相当于重写了原型链。

```javascript
SubType.prototype = {
  getSubValue() {
    return this.subproperty;
  },

  someOtherMethod() {
    return false;
  },
};

let instance = new SubType();

console.log(instance.getSuperValue()); // TypeError
```

### 原型的问题

-   原型中包含的引用值会在所有实例间共享
-   子类型在实例化时不能给父类型的构造函数传参

## 盗用构造函数

"盗用构造函数"有时也称作"对象伪装"或"经典继承"。基本思路很简单：在子类构造函数中调用父类构造函数。因为毕竟函数就是在特地上下文中执行代码的简单对象，所以可以使用 apply()和 call()方法以新创建的对象为上下文执行的构造函数。

```javascript
function SuperType() {
  this.language = ["Python", "Javascript", "C"];
}

function SubType() {
  SuperType.call(this);
}

let instance_1 = new SubType();
let instance_2 = new SubType();

instance_1.language.push("Java");
console.log(inatance_1); // ["Python", "Javascript", "C", "Java"]
console.log(inatance_2); // ["Python", "Javascript", "C"]
```

### 传递参数

相比于使用原型链，盗用构造函数的一个优点就是可以在子类构造函数中向父类构造函数传参。

```javascript
function SuperType(v) {
  this.value = v;
}

function SubType(v) {
  SuperType.call(this, this.v);
  this.sex = "girl";
}

let instance_1 = new SubType("Kinsiy");
let instance_2 = new SubType("Restituo");

console.log(`value: ${instance_1.value}     sex: ${instance_1.sex}`);
console.log(`value: ${instance_2.value}     sex: ${instance_2.sex}`);

//value: Kinsiy     sex: girl
//value: Restituo     sex: girl
```

### 盗用构造函数的问题

盗用构造函数的主要缺点，也是使用构造函数模式自定义类型的问题：必须在构造函数中定义方法，因此函数不能重用。此外，子类也不能访问父类原型上定义的方法，因此所有类型只能使用构造函数模式。

## 组合继承

组合继承(有时候也叫伪经典继承)综合了原型链和盗用构造函数，将两者的优点集中了起来。其基本思想是使用原型链继承原型上的属性和方法，而通过盗用构造函数来继承实例属性。
组合继承拟补了原型链和盗用构造函数的不足，是 Javascript 中使用最多的继承模式。而且组合继承也保留了 instanceof 操作符和 isPrototypeOf()方法识别合成对象的能力。

```javascript
function SuperType(v) {
  this.value = v;
  this.language = ["Java", "Python", "Go"];
}

SuperType.prototype.sayName = function () {
  console.log(this.value);
};

function SubType(v, n) {
  SuperType.call(this, v);
  this.age = n;
}

SubType.prototype = new SuperType();

SubType.prototype.sayAge = function () {
  console.log(this.age);
};

let instance_1 = new SubType("Kinsiy", 24);
let instance_2 = new SubType("Restituo", 23);

instance_1.language.push("Rust");
console.log(instance_1.language); // ["Java", "Python", "Go", "Rust"]
instance_1.sayName(); // Kinsiy
instance_1.sayAge(); // 24

console.log(instance_2.language); // ["Java", "Python", "Go"]
instance_2.sayName(); // Restituo
instance_2.sayAge(); // 23
```

## 原型式继承

```javascript
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
```

这个 object()函数会创建一个临时构造函数，将传入的对象赋值给这个构造函数的原型，然后返回这个临时类型的一个实例。本质上，object()是对传入的对象执行了一次浅复制。
原型式继承是适用于这种情况：你有一个对象，想在它的基础上再创建一个新对象。你需要把这个对象先传给 object()，然后再对返回的对象进行适当修改。
EMCAScript5 通过增加 Object.create()方法将原型式继承的概念规范化了。这个方法接收两个参数：作为新对象原型的对象，以及给新对象定义额外属性的对象(第二个可选)。在只有一个参数时，Object.create()与这里的 object()方法效果相同。

```javascript
let person = {
  name: "Kinsiy",
  friends: ["Restituo", "Type57"],
};

let anotherPerson = Object.create(person);
anotherPerson.name = "QingLai";
anOtherPerson.friends.push("King");

let yetAnthorPerson = Object.create(person);
anotherPerson.name = "BaiLiXu";
anOtherPerson.friends.push("witcher");

console.log(yetAnthorPerson.friends); //  ["Restituo", "Type57", "King", "witcher"]

let person_3 = Object.create(person, {
  name: {
    value: "Queen",
  },
});

console.log(person_3.name); // Queen
```

## 寄生式继承

与原型式继承比较接近的是一种继承方式是寄生式继承。寄生式继承背后的思路类似于寄生构造函数和工厂模式：创建一个实现继承的函数，以某种方式增强对象，然后返回这个对象。基本的寄生继承模式如下：

```javascript
function createAnother(original) {
  let clone = Object.create(original);
  clone.sayHi = function () {
    console.log("Hi");
  };

  return clone;
}

let person = {
  name: "Kinsiy",
  language: ["Javascript", "Python"],
};

let someone = createAnother(person);
someone.sayHi(); // Hi
```

## 寄生式组合继承

组合继承其实也有存在效率问题。最主要的效率问题就是父类构造函数始终会被调用两次：一次是在创建子类原型时调用，另一次是在子类构造函数中调用。本质上，子类原型最终是要包含超类对象的所有实例属性，子类构造函数只要在执行时重写自己的原型就行了。

```javascript
function inheritPrototype(SubType, SuperType) {
  let prototype = Object.create(SuperType.prototype);
  prototype.constructor = SubType;
  SubType.prototype = prototype;
}

function SuperType(name) {
  this.name = name;
  this.colors = ["red", "green", "blue"];
}

SuperType.prototype.sayName = function () {
  console.log(this.name);
};

function SubType(v, n) {
  SuperType.call(this, v);
  this.age = n;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function () {
  console.log(this.age);
};

let me = new SubType("Kinsiy", 23);
console.log(me instanceof SuperType); // true
me.sayName(); // Kinsiy
```

这里只调用了一次 SuperType 构造函数，避免了 SubType.prototype 上不必要也用不到的属性，因此可以说是这个例子的效率更高。而且，原型键仍然保持不变，因此 instanceof 操作符合 isPrototypeOf()方法正常有效。寄生式组合继承可以算是引用类型继承的最佳模式。
