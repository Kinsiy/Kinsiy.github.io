---
title: Typescript - Classes
mathjax: false
date: 2022-03-28 23:17:51
tags: [Classes]
categories: [学习笔记, TypeScript]
description:
photos:
---

Ts完全支持Es2015中的{% label primary@class %}关键字.

{% note info%}

学习ES6中的类[Javascript - Class](https://kinsiy.github.io/Javascript-Class/?highlight=%E7%B1%BB)、[MDN. Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

本文参考Typescript官网[Docs - Handbook - Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)自翻译而来, 仅供个人参考学习使用.一切请以官网为准!

{% endnote %}

## 类成员

来看示例(--strictPropertyInitialization)

```typescript
class Point {
  x: number
  y						// [1] Error: Member 'y' implicitly has an 'any' type.
  y: number 	// [2] Property 'y' has no initializer and is not definitely assigned in the constructor
  z = 0
  i: string = "kinsiy"

  constructor(){
    this.x = 1
    this.j = 2	// [3] Error: Property 'j' does not exist on type 'Point'.
  }
}

let point = new Point()

point.z = "3" // [4] Error: Type 'string' is not assignable to type 'number'.
point.j = "xxx" // [3] Error: Property 'j' does not exist on type 'Point'.

```

<!--more-->

通过示例中的四个错误, 学习 Ts 类

1. Ts中对于未声明类型的成员, 会隐式的得到一个{% label primary@any %}类型
2. 成员变量必须初始化(配置strictPropertyInitialization)
3. 不能对未声明的变量进行初始化
4. 与普通的变量声明一样, 具有[类型推断](https://kinsiy.github.io/Typescript-TypeGuards/)

对于第二个错误, 必须在定义成员时或**构造方法中**进行初始化,才不会报错,Ts不会检查构造方法调用的方法,因为这些方法可能会被派生类重新.从而导致未初始化. 如果你确定该变量一定会被初始化可以使用非空断言进行声明

```typescript
class OKGreeter {
  name!: string // ok
}
```

### 访问修饰符

#### readonly

只读成员,通过readonly修饰符声明. 只读成员在任何地方可读

```typescript
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";	// Error: Cannot assign to 'name' because it is a read-only property.
  }
}
const g = new Greeter();
g.name = "also not ok";	// Error: Cannot assign to 'name' because it is a read-only property.
```

#### public

公有成员,通过public修饰符声明.默认修饰符. 公有成员在任何地方都可以被读/写

```typescript
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```

#### protected

保护成员,通过protected修饰符声明. 保护成员**只能在自身及其派生类内部读/写**(实例不可见)

```typescript
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }

  _getName(){
    console.log(this.getName())
  }
}
 
class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g._getName()
g.getName();	// Error: Property 'getName' is protected and only accessible within class 'Greeter' and its subclasses.
```

#### private

私有成员, 通过private修饰符声明. 私有成员**只能在该类内部读/写**(实例不可见)

```typescript
// 对实例不可见
class Base {
  private x = 0
}
const b = new Base()
console.log(b.x)	// Error: Property 'x' is private and only accessible within class 'Base'.

// 对派生类不可见
class Derived extends Base {
  showX() {
    console.log(this.x) // Error:Property 'x' is private and only accessible within class 'Base'.
  }
}

// 不能被派生类改变可见性
class _Derived extends Base {
  x = 1
  // Error: Class '_Derived' incorrectly extends base class 'Base'.
  //				  Property 'x' is private in type 'Base' but not in type '_Derived'.
}
```

Ts与Java/C#/C++/Swift/PHP一样可以跨实例访问私有成员

```typescript
// 能够跨实例访问
class A {
  private x = 10;
  public sameAs(other: A) {
    return other.x === this.x;	// ok
  }
}
```

#### 注意

上述修饰符仅在类型检查期间起作用,在编译成 Js 运行时你还是能够在对应实例上找到被{% label primary@protected %}/{% label primary@private %}的成员.被{% label primary@readonly %}修饰的成员也能在运行时进行写操作.

并且在Ts类型检查阶段, 对于{% label primary@protected %}/{% label primary@private %}修饰的成员,可以通过中括号(obj[key])的方式逃过类型检查({% label primary@readonly %}不行)

```typescript
class MySafe {
  private secretKey = 12345;
  protected kinsiy = 2212
  readonly qinglai = 333
}

const s = new MySafe()

console.log(s.secretKey) // Error: Property 'secretKey' is private and only accessible within class 'MySafe'.
 
console.log(s["secretKey"]) // OK

console.log(s["kinsiy"]) // OK

console.log(s.kinsiy)

s["qinglai"] = 3333 // Error: Cannot assign to 'qinglai' because it is a read-only property.
```

要想实现硬私有,可以直接使用JS中的私有成员修饰符**(#)**

```typescript
class Dog {
  #barkAmount = 0;
  personality = "happy";
 
  constructor() {}
}
```

### 构造函数

构造函数声明与一般函数声明几乎一样.只有两个小小的不同.

- 不能使用泛型, 类的泛型在class关键字上声明,后面会介绍
- 不能有返回类型注解, 构造函数永远返回类实例

```typescript
class Point {
  x: number;
  y: number;
 
  // 重载
  constructor(x: number, y: string);
  constructor(x: number, y: number)
  constructor(s: string);
  
  constructor(x: any, y?: any)  {
    x = Number(x)
    y = Number(y)
    this.x = x;
    this.y = y;
  }
}

// 还可以把成员声明放在构造函数中
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3)
console.log(a.x) // (property) Params.x: number

console.log(a.z) // Property 'z' is private and only accessible within class 'Params'.
```

{% note info%}

与Js一样,在类构造函数中,不能在调用super()之前引用this

{% endnote %}

### 方法

Ts中声明方法与Js中声明方法除类型注解外,其余部分完全一样.

```typescript
class Point {
  x = 10;
  y = 10;
 
  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

### Getters/Setters

设置类成员的Getters/Setters时,需要注意几点

- 仅设置getters, 成员自动变为仅读成员
- 如果未注解setters的参数类型,则其类型与getters返回类型一致
- Getters/Setters必须拥有一样的成员可见性

```typescript
class Thing {
  _size = 0

  get size(): number {
    return this._size
  }

  set size(value: string | number | boolean) {
    let num = Number(value)

    // Don't allow NaN, Infinity, etc

    if (!Number.isFinite(num)) {
      this._size = 0
      return
    }

    this._size = num
  }
}
```

### 索引签名

与Interface等对象类型一样,类也支持索引签名

```typescript
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);
 
  check(s: string) {
    return this[s] as boolean;
  }
}
```

## 继承/实现

### implements

使用implements实现一个或多个接口.需要注意的是接口仅检查类是否可以被视为接口类型,不会更改类声明的类型.举例来说

```typescript
interface _Checkable{
  _check(name: string): boolean
}

interface Checkable {
  check(name: string): boolean
}
 
class NameChecker implements Checkable, _Checkable {
  _check(s) {	// [1] Error: Parameter 's' implicitly has an 'any' type.
    return s.toLowerCase() === "ok"
  }
  
  check(n: number) {	// [2] Error
    let s = String(n)
    return s.toLowerCase() === "ok"
  }
}

/**
Property 'check' in type 'NameChecker' is not assignable to the same property in base type 'Checkable'.
  Type '(n: number) => boolean' is not assignable to type '(name: string) => boolean'.
    Types of parameters 'n' and 'name' are incompatible.
      Type 'string' is not assignable to type 'number'.
*/
```

注意看[1\][2]两个报错, [1\]报的是‘s’是any类型而不是[2\]的类型不符,也就是说_check()是符合接口约束的,但是参数’s’的类型并不会被接口改写为要求的{% label primary@string %}  

### extends

与Js一样,Ts使用extends实现继承. 除类型注解外.差异不大

```typescript
class Base {
  greet(name?: string):void
  greet() {
    console.log("Hello, world!");
  }
}
 
class Derived extends Base {
  greet(name: string) {
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
```

派生类重写方法需要注意方法签名要与基类一致

#### declare

当编译目标大于等于ES2022或--useDefineForClassFields时, 派生类声明的任何成员都会覆盖基类声明的值,当我们仅想在派生类中更具体化成员类型时.会直接覆盖基类初始值.这时可以使用{% label primary@declare %}表面派生类声明不会产生任何运行时影响

```typescript
interface Animal {
  dateOfBirth: any;
}
 
interface Dog extends Animal {
  breed: any;
}
 
class AnimalHouse {
  resident: Animal;
  constructor(animal: Animal) {
    this.resident = animal;
  }
}
 
class DogHouse extends AnimalHouse {
  // Does not emit JavaScript code,
  // only ensures the types are correct
  declare resident: Dog;
  constructor(dog: Dog) {
    super(dog);
  }
}
```

#### 初始化顺序

1. 基类成员初始化
2. 执行基类构造函数
3. 派生类成员初始化
4. 执行派生类构造函数

```typescript
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + this.name);
  }
}
 
class Derived extends Base {
  name = "derived";
}
 
// Prints "base", not "derived"
const d = new Derived();
```

## 静态成员

使用{% label primary@static %}修饰符可声明静态成员.其性质与JS原生实现别无二致. 并且可以与{% label primary@public %},{% label primary@protected %},{% label primary@private %}同时使用

```typescript
class MyClass {
  static x = 0
  private static y = 1	// 与 private 同时使用
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);	// 0
MyClass.printX();	// 0

console.log(MyClass.y) // Error: Property 'y' is private and only accessible within class 'MyClass'.


// 可被继承
class _MyClass extends MyClass {
  _printX = MyClass.printX
}

let mc = new _MyClass()

mc._printX() // 0
```

{% note info %}

由于在JS中,类就是一个特殊的函数.所以在声明静态类成员时,需要注意,不允许覆盖{% label primary@Function %}原型链上的属性,比如{% label primary@name %},{% label primary@length %},{% label primary@call %}...等

{% endnote %}

## 静态块*

可以在类中声明静态块,可以访问所以的类内部成员

{% note info %}

暂不理解,这么做的目的是什么? 但确实可以这么干

{% endnote %}

```typescript
class Foo {
  static #count = 0 // 注意这里是硬私有哦

  get count() {
    return Foo.#count;
  }

  static {
    try {
      const lastInstances = loadLastInstances()
      Foo.#count += lastInstances.length;
    }
    catch {}
  }

  static {
    Foo.#count ++
  }
}
```

## 泛型类

正如前面构造函数提到的,类的泛型在{% label primary@class %}关键字上声明.

```typescript
class Box<Type> {
  contents: Type
  constructor(value: Type) {
    this.contents = value
  }
}
 
const b = new Box("hello!") // const b: Box<string>
b.contents // (property) Box<string>.contents: string
```

{% note info %}

不要在静态成员上使用泛型, 谨记一点, 在编译后所有的类型注解都会去掉. 我们知道JS中的类是基于原型链实现继承的, 修改原型链上的任何属性都会影响到所有实例.在静态类上使用泛型{% label danger@Box&lt;string&gt;%}与{% label danger@Box&lt;number&gt;%}在编译后其实并无差别,更改他们中的任意一个的原型链都会影响到另一个.

{% endnote %}

## this 参数

我们知道,在JS中,{% label primary@this %}的值是动态绑定的.比如,下面的例子

```typescript
class MyClass {
  name = "MyClass"
  getName() {
    return this.name
  }
}
const c = new MyClass()
const obj = {
  name: "obj",
  getName: c.getName,
};
 
// Prints "obj", not "MyClass"
console.log(obj.getName())
```

这并不是一种错误的行为,当我们清楚我们的在做什么时,可以使用箭头函数,bind等绑定回 MyClass 实例.

但这样还是有一点小小的隐患,可能会出现误用.有没有办法在类型检查阶段就限制该方法只能在该类实例中调用?这就用到了Ts中特有的{% label primary@this %}参数

```typescript
class MyClass {
  name = "MyClass";
  getName(this: MyClass) { // 注意这里这个this参数
    return this.name;
  }
}
const c = new MyClass();

c.getName() // OK

const g = c.getName;
console.log(g()); // Error: The 'this' context of type 'void' is not assignable to method's 'this' of type 'MyClass'.
```

可以当直接调用getName()时,Ts检查到{% label primary@this %}类型不同会给出错误.

{% label primary@this %}参数在编译成Js时会被去掉

```typescript
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
  /* ... */
}

// JavaScript output
function fn(x) {
  /* ... */
}
```

## 抽象类

在JS中我们无法直接声明抽象类,但是很容易可以通过 new.target 来达到目的.Ts中则是直接支持使用{% label primary@abstract %}关键字声明抽象类

```typescript
abstract class Base {
  abstract getName(): string;
 
  printName() {
    console.log("Hello, " + this.getName())
  }
}
 
const b = new Base() // Error: Cannot create an instance of an abstract class.

// 新建派生类实现抽象类
class Derived extends Base {
  getName() {
    return "world"
  }
}
 
const d = new Derived() // ok
d.printName()	// ok
```

{% note info %}

注意: 在类型检查中,不要直接检查类型是否与抽象类一致,而是检查构造函数返回的类型是否属于该抽象类,举例来说

```typescript
function greet(ctor: new () => Base) {	// 不要写成  Base 或 typeof Base
  const instance = new ctor();
  instance.printName();
}
greet(Derived)
greet(Base) // Error

/**
Argument of type 'typeof Base' is not assignable to parameter of type 'new () => Base'.
  Cannot assign an abstract constructor type to a non-abstract constructor type.
*/
```

{% endnote %}

## 鸭子类型

Ts的类型检查是一种鸭子类型的检查.只要两个类的结构相同,我们完全可以对这两种类型的变量进行相互赋值.

```typescript
class Point1 {
  x = 1
  y = 2
}
 
class Point2 {
  x = 3
  y = 4
  z = 5
}
 
// OK
const p: Point1 = new Point2()	// 反过来不行
```

## 参考

[1\][Typescriptlang.org. Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)
