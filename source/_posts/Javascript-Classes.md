---
title: Javascript - Classes
date: 2021-03-04 21:07:28
categories: [学习笔记, Javascript]
tags: [Classes]
keywords:
description: 类(class)是 ECMAScript 中新的基础性语法糖结构。虽然 ECMAScript6 类表面上看起来可以支持正式的面向对象编程，但实际上它背后使用的仍然是原型和构造函数的概念。
photos:
---

## 类定义

与函数类型相似，定义类也有两种主要方式：类声明和类表达式。这两种方式都使用 class 关键字加大括号。与函数表达式类似，类表达式在它们被求值前也不能引用。

不过，与函数定义不同的是:

- 虽然函数声明可以提升，但类定义不能。
- 函数受函数作用域限制，而类受块作用域限制。

```javascript
// 类声明
class Person {}

// 类表达式
const Animal = class {};

console.log(ClassDeclaration); // ReferenceError: Cannot access 'ClassDeclaration' before initialization
class ClassDeclaration {}
console.log(ClassDeclaration); // class ClassDeclaration {}

{
  function FunctionDeclaration() {}
  class ClassDeclaration {}
}

console.log(FunctionDeclaration); // ƒ FunctionDeclaration() {}
console.log(ClassDeclaration); // ReferenceError: ClassDeclaration is not defined
```

### 类的构成

类可以包含构造函数方法、实例方法、获取函数、设置函数、和静态类方法。但这些都不是必需的。空的类定义照样有效。默认情况下，类定义中的代码都在严格模式下执行。类表达式的名称是可选的。在把类表达式赋值给变量后，可以通过 name 属性取得类表达式的名称字符串。但不能在类表达式作用域外部访问这个标识符。

```javascript
let Person = class PersonName {
  identify() {
    console.log(Person.name, PersonName.name);
  }
};

let p = new Person();

p.identify(); // PersonName PersonName

console.log(Person.name); // PersonName
console.log(PersonName); // ReferenceError: PersonName is not defined
```

## 类构造函数

constructor 关键字用于在类定义块内部创建类的构造函数。方法名 constructor 会告诉解释器在使用 new 操作符创建类的新实例时，应该调用这个函数。构造函数不是必须的，不定义构造函数相当于将构造函数定义为空函数。

### 实例化

使用 new 操作符实例化 Person 的操作等于使用 new 调用其构造函数。唯一可感知的不同之处就是，Javascript 解释器知道使用 new 和类意味着应该使用 constructor 函数进行实例化。使用 new 调用类的构造函数会执行如下操作。

1. 在内存中创建一个新对象
2. 这个新对象内部的[[Prototype]]指针被赋值为构造函数的 prototype 属性。
3. 构造函数内部的 this 被赋值为这个新对象(即 this 指向新对象)
4. 执行构造函数的代码(给新对象添加属性)
5. 如果构造函数返回非空对象，则返回改对象；否则，返回刚创建的新对象。

```javascript
class Animal {}
class Person {
  constructor() {
    console.log("person ctor");
  }
}

class Vegetable {
  constructor() {
    this.color = "orange";
  }
}

let a = new Animal();
let b = new Person(); // person ctor
let c = new Vegetable();
console.log(c.color); // orange
```

类实例化时传入的参数会用作构造函数的参数。如果不需要参数，则类名后面的括号也是可选的。

默认情况下，类构造函数会在执行之后返回 this 对象。构造函数返回的对象会被用作实例化的对象，如果没用引用新创建的 this 对象，那么这个对象会被销毁。不过，如果返回的不是 this 对象，而是其他对象，那么这个对象不会通过 instanceof 操作符监测出跟类有关联，因为这个对象的原型指针没有被修改。

```javascript
class Person {
  constructor(override) {
    this.foo = "foo";
    if (override) {
      return { bar: "bar" };
    }
  }
}

let p1 = new Person();
let p2 = new Person(true);

console.log(p1); // Person {foo: "foo"}
console.log(p2); // {bar: "bar"}
console.log(p1 instanceof Person); // true
console.log(p2 instanceof Person); // false
```
{% note primary %}
 #### 类构造函数与构造函数的主要区别
 调用类构造函数必须使用new操作符。而普通构造函数如果不使用new调用，那么就会以全局的this(通常是window)作为内部对象。调用类构造函数时如果忘了使用new则会抛出错误

类构造函数没有什么特殊之处，实例化之后，它会成为普通的实例方法，但作为类构造函数，仍然要使用new调用
{% endnote%}

```javascript
p1.constructor(); // TypeError: Class constructor Person cannot be invoked without 'new

let p3 = new p1.constructor();

console.log(p3 instanceof Person); // true
```

### 把类当成特殊函数

ECMAScript 中没有正式的类这个类型。从各方面来看，ECMAScript 类就是一种特殊函数。声明一个类后，通过 typeof 操作符检测类标识符，表面他是一个函数。类标签符有 prototype 属性，而这个原型也有一个 constructor 属性指向类自身。

在类的上下文中，类本身在使用 new 调用时就会被当成构造函数。重点在于，类中定义的 constructor 方法不会被当成构造函数，在对它使用 instanceof 操作符会返回 false。但是，如果在创建实例时直接将类构造函数当成普通构造函数来使用，那么 instanceof 操作符的返回值会反转。

```javascript
class Person {}

let p_1 = new Person();
console.log(p_1.constructor === Person); // true
console.log(p_1 instanceof Person); // true
console.log(p_1 instanceof Person.constructor); // false

let p_2 = new Person.constructor();
console.log(p_2.constructor === Person); // false
console.log(p_2 instanceof Person); // false
console.log(p_2 instanceof Person.constructor); // true
```

类可以像其他对象或函数引用一样把类作为参数传递。与立即调用函数表达式相似，类也可以立即实例化。

## 实例、原型和类成员

类的语法可以非常方便地定义应该存在于实例上的成员，应该存在于原型上的成员，以及应该存在于类本身的成员。

### 实例成员

每个实例都对应一个唯一的成员对象，这意味着所有成员都不会在原型上共享。

```javascript
class Person {
  constructor() {
    this.name = new String("Kinsiy");
    this.sayName = () => console.log(this.name);
    this.nickNames = ["King", "Queen"];
  }
}

let p_1 = new Person();
let p_2 = new Person();

p_1.sayName(); // String {"Kinsiy"}
p_2.sayName(); // String {"Kinsiy"}

console.log(p_1.name === p_2.name); // false
console.log(p_1.sayName === p_2.sayName); // false
console.log(p_1.nickNames === p_2.nickNames); // false

p_1.name = p_1.nickNames[0];
p_2.name = p_2.nickNames[1];

p_1.sayName(); // King
p_2.sayName(); // Queen
```

### 原型方法与访问器

为了在实例间共享方法，类定义语法把类块中定义的方法作为原型方法。

```javascript
class Person {
  constructor() {
    // 添加到this的所有内容都会存在于不同的实例上
    this.locate = () => console.log("instance");
  }

  // 在类块中定义的所有内容都会定义在类的原型上
  locate() {
    console.log("prototype");
  }
}

let p = new Person();

p.locate(); // instance
Person.prototype.locate(); // prototype
```

{% note info %}

可以把方法定义在类构造函数中或者类块中，但不能在类块中给原型添加原始值或对象作为成员数据.

类方法等同于对象属性，因此可以使用字符串、符号或计算的值作为键

类也支持获取和设置访问器。语法与行为跟普通对象一样

{% endnote %}

```javascript
class Car {
  set name(newName) {
    this.name_ = newName;
  }

  get name() {
    return this.name_;
  }
}

let p_2 = new Person();

p_2.name = "Kinsiy";
console.log(p_2.name); // Kinsiy
```

### 静态类方法

可以在类上定义静态方法。这些方法通常用于执行不特定与实例的操作，也不要求存在类的实例。与原型成员类似，每个类只能有一个静态成员。静态类方法非常适合作为实例工厂

```javascript
class Person {
  constructor(age) {
    this.age_ = age;
  }

  sayAge() {
    console.log(this.age_);
  }

  static create() {
    return new Person(Math.floor(Math.random() * 100));
  }
}

console.log(Person.create()); // Person {age_: 50}
```

### 非函数原型和类成员

虽然类定义并不显式支持在原型或类上添加成员数据，但在类定义外部，可以手动添加

```javascript
class Person {
  sayName() {
    console.log(`${Person.greeting} ${this.name}`);
  }
}

// 在类上定义数据成员。无法通过this引用
Person.greeting = "My name is";

// 在原型上定义数据成员
Person.prototype.name = "Kinsiy";

let me = new Person();
me.sayName(); // My name is Kinsiy
```

### 迭代器与生成器方法

类定义语法支持在原型和类本身上定义生成器方法

```Javascript
class Person{
  constructor(){
    this.nicknames= ["Kinsiy","Restituo","Type57"]
  }

  // 在原型上定义生成器方法
  *createNicknameIterator(){
    yield "Kinsiy"
    yield "King"
    yield "Queen"
  }

  // 在类上定义生成器方法
  static *createJobIterator(){
    yield "Butcher"
    yield "Baker"
    yield "Killer"
  }

  // 添加一个默认的迭代器，把类实例变成可迭代对象
  *[Symbol.iterator](){
    yield *this.nicknames.entries();
  }
}

let p = new Person;
for (let i of p){
  console.log(i)
}
// [0, "Kinsiy"] [1, "Restituo"] [2, "Type57"]
```

## 继承

ECMAScript 新增特性中最出色的一个就是原生支持了类继承机制。虽然类继承使用的是新语法，但背后依旧使用的是原型链。

### 继承基础

ES6 类支持单继承。使用 extends 关键字，就可以继承任何拥有[[Construct]]和原型的对象。很大程度上，这意味着不仅可以继承一个类，也可以继承普通的构造函数(保持向后兼容)

类和原型上定义的方法都会带到派生类。this 的值会反映调用相应方法的实例或者类

```javascript
class Vehicle {
  identifyPrototype(id) {
    console.log(id, this);
  }

  static identifyClass(id) {
    console.log(id, this);
  }
}

class Bus extends Vehicle {}

let v = new Vehicle();
let b = new Bus();

console.log(b instanceof Bus); // true
console.log(b instanceof Vehicle); // true

v.identifyPrototype("vehicle"); // vehicle Vehicle {}
b.identifyPrototype("bus"); // bus Bus {}

Vehicle.identifyClass("Vehicle"); // Vehicle class Vehicle...
Bus.identifyClass("Bus"); // Bus class Bus extends Vehicle {}
```

### 构造函数、HomeObject 和 Super()

派生类的方法可以通过 super 关键字引用它们的原型。这个关键字只能在派生类中使用，而且仅限于类构造函数、实例方法和静态方法内部。在类构造函数中使用 super 可以调用父类构造函数。

ES6 给类构造函数和静态方法添加了内部特性[[HomeObject]]，这个特性是一个指针，指向定义该方法的对象。这个指针是自动赋值的，而且只能在 Javascript 引擎内部访问，super 始终定义为[[HomeObject]]的原型。

```javascript
// super只能在派生类构造函数和静态方法中使用
class Vehicle {
  constructor() {
    super();
    // SyntaxError: 'super' keyword unexpected here
  }
}

// 不能单独引用super关键字，要么用它调用构造函数，要么用它引用静态方法
class A {}

class B extends A {
  constructor() {
    console.log(super);
    // SyntaxError: 'super' keyword unexpected here
  }
}

// 调用super()会调用父类构造函数，并将返回的实例赋值给this
class C {}

class D extends C {
  constructor() {
    super();
    console.log(this instanceof C);
  }
}

let d = new D(); // true

// super()的行为如同调用构造函数，如果需要给父类构造函数传参，则需要手动传入。
class E {
  constructor(v) {
    this.flag = v;
  }
}

class F extends E {
  constructor(subV) {
    super(subV);
  }
}

console.log(new F("Kinsiy")); // F {flag: "Kinsiy"}

// 如果没有定义派生类的构造函数，在实例化派生类时会调用super()，而且会传入所有传给派生类的参数
class G extends E {}

console.log(new G("Type57")); // G {flag: "Type57"}

// 在类构造函数中，不能在调用super()之前引用this
class H extends E {
  constructor() {
    console.log(this);
  }
}

new H(); // ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor

// 如果在派生类中显示定义了构造函数，则要么必须在其中调用super()，要么必须在其中返回一个对象
class I extends E {
  constructor() {
    return {}; // 这个对象不会通过 instanceof 操作符监测出跟类有关联
  }
}

let i = new I();
console.log(i instanceof I); // fasle
```

### 抽象基类

有时候可能需要爱定义这样一个类，它可供其他类继承，但本身不会被实例化。虽然 ECMAScript 没有专门支持这种类的语法，但通过 new.target 也容易实现。

new.target 保存通过 new 关键字调用的类或函数。另外在抽象基类构造函数中进行检查，可以要求派生类必须定义某个方法。因为原型方法在调用类构造函数之前就已经存在了，所以可以通过 this 关键字来检查相应的方法

```javascript
// 抽象基类
class Vehicle {
  constructor() {
    console.log(new.target); // class Bus extends Vehicle {}
    if (new.target === Vehicle) {
      throw new Error("Vehicle 不能被实例化！");
    }

    if (!this.foo) {
      throw new Error("Vehicle 派生类必须实现foo()");
    }
  }
}

// 派生类
class Bus extends Vehicle {
  foo() {}
}
class Biclcye extends Vehicle {}

new Bus();
new Vehicle(); // Error: Vehicle 不能被实例化！
new Biclcye(); // Error: Vehicle 派生类必须实现foo()
```

### 继承内置类型

ES6 类为继承内置引用类型提供顺畅的机制，开发者可以方便地扩展内置类型

```javascript
class SuperArray extends Array {
  // 洗牌算法
  shuffle() {
    for (let i = this.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [this[i], this[j]] = [this[j], this[i]];
    }
  }
}

let a = new SuperArray(1, 2, 3, 4, 5);
console.log(a instanceof Array); // true

console.log(a); // SuperArray(5) [1, 2, 3, 4, 5]
a.shuffle();
console.log(a); // SuperArray(5) [2, 3, 5, 4, 1]

// 有些内置类型的方法会返回新实例。默认情况下，返回的实例类型与原型实例的类型是一致的
// 如果想覆盖这个默认行为，则可以覆盖Symbol.species访问器

let a_1 = a.filter((x) => !!(x % 2));
console.log(a_1); // SuperArray(3) [1, 3, 5]
console.log(a_1 instanceof SuperArray); // true

class SuperArray_1 extends SuperArray {
  static get [Symbol.species]() {
    return Array;
  }
}

let b = new SuperArray_1(1, 2, 3, 4, 5);
let b_1 = b.filter((x) => !!(x % 2));
console.log(b_1); // [1, 3, 5]
console.log(b_1 instanceof SuperArray); // false
console.log(b_1 instanceof SuperArray_1); // false
console.log(b_1 instanceof Array); // true
```

### 类混入

把不同类的行为集中到一个类是一种常见的 Javascript 模式。虽然 ES6 没有显式支持多类继承，但通过现有特性可以轻松地模拟这种行为。

```Javascript
class Vehicle{}

let FooMixin = (SuperClass) => class extends SuperClass{
  foo(){
    console.log("foo")
  }
}

let BarMixin = (SuperClass) => class extends SuperClass{
  bar(){
    console.log("bar")
  }
}

let BzaMixin = (SuperClass) => class extends SuperClass{
  baz(){
    console.log("baz")
  }
}

function mix(Baseclass,...Mixins){
  return Mixins.reduce((accumulator, current) => current(accumulator),Baseclass)
}

class Bus extends mix(Vehicle,FooMixin,BarMixin,BzaMixin){}

let b = new Bus
b.foo();  // foo
b.bar();  // bar
b.baz();  // baz
```

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)

