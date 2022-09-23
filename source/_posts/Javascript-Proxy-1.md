---
title: Javascript-Proxy[代理基础]
date: 2021-03-07 10:14:03
categories: [学习笔记, Javascript]
tags: [JS红宝书, Proxy]
keywords:
description:
photos:
---

## 代理基础

代理是目标对象的抽象。从很多方面看，代理类似 C++指针，因为它可以用作目标对象的替身，但又完全独立于目标对象。目标对象即可以直接被操作，也可以通过代理来操作。

### 创建空代理

代理是使用 Proxy 构造函数来创建的。这个构造函数接收两个参数：目标对象和处理程序对象、缺少其中任何一个参数都会抛出 TypeError。

<!-- more -->

```javascript
const target = {
  id: "Kinsiy",
};

const handler = {};

const proxy = new Proxy(target, handler);

// id属性会访问同一个值
console.log(`target: ${target.id} proxy: ${proxy.id}`); // target: Kinsiy proxy: Kinsiy

// 给目标及代理属性赋值会反映在两个对象上
target.key = "King";
proxy.value = "Queen";

console.log(target.value); // Queen
console.log(proxy.key); // King

// hasOwnProperty()方法在两个地方
console.log(target.hasOwnProperty("id")); // true
console.log(proxy.hasOwnProperty("id")); // true

// Proxy.prototype 是undefined 不能使用instanceof
console.log(proxy instanceof Proxy); // TypeError: Function has non-object prototype 'undefined' in instanceof check

// 严格相等可以用来区分代理和目标
console.log(proxy === target); // false
```

### 定义捕获器

使用代理的主要目的是可以定义捕获器(trap)。捕获器就是在处理程序对象中定义的“基本操作的拦截器”。每个处理程序可以包含零个或多个捕获器，每个捕获器都对应一种基本操作，可以直接或间接在代理对象上调用。每次在捕获器上调用这些基本操作时，代理可以在这些操作传播到目标对象之前先调用捕获器函数，从而拦截并修改相应的行为。

```javascript
const target = {
  language: "Javascript",
};

const handler = {
  get() {
    return "handler override";
  },
};

const proxy = new Proxy(target, handler);

console.log(proxy.language); // handler override
console.log(target.language); // Javascript
```

上面定义的 get()捕获器不是 ECMAScript 对象可以调用的方法。这个操作在 Javascript 代码中可以通过多种形式触发并被 get()捕获器拦截到。proxy[prototype]、proxy.property 或 Object.create(proxy)[property]等操作都会触发基本的 get()操作以获取属性。

### 捕获器参数和反射 API

所有捕获器都可以访问相应的参数，基于这些参数可以重建被捕获方法的原始行为。比如 get()捕获器会接收到目标对象、要查询的属性和代理对象三个参数。
所有捕获器都可以基于自己的参数重建原始操作，但并非所有捕获器行为都像 get()那么简单。因此，通过手动写码如法炮制的想法是不现实的。实际上，开发者不需要手动重建原始行为，而是可以通过调用全局 Reflect 对象上(封装了原始行为)的同名方法来轻松重建。
处理程序对象中所有可以捕获的方法都有对应的反射(Reflect)API 方法。这些方法与捕获器拦截的方法具有相同的名称和函数签名，而且也具有被拦截方法相同的行为。

```javascript
const target = {
  id: "Kinsiy",
  info: "man never cay!",
};

const handler = {
  get(trapTarget, property, receiver) {
    let decoration = "";
    if (property === "id") {
      decoration = " may cry!";
    }
    return Reflect.get(...arguments) + decoration;
  },
};

const proxy = new Proxy(target, handler);

console.log(proxy.id); // Kinsiy may cry!
console.log(proxy.info); // man never cay!
```

### 捕获器不变式

使用捕获器几乎可以改变所有基本方法的行为，但也不是没有限制。根据 ECMAScript 规范，每个捕获的方法都知道目标对象上下文，捕获函数签名，而捕获处理程序的行为必须遵守"捕获器不变式"(trap invariant)。捕获器不变式因方法不同而异，但通常都会防止捕获器定义出现过于反常的行为。比如，如果目标对象有一个<b>不可配置且不可写</b>的数据属性，那么在捕获器返回一个<b>与该属性不同的值</b>时，会抛出 TypeError。

### 可撤销代理

有时候可能需要中断代理对象与目标对象之间的联系。对于 new Proxy()创建的普通代理来说，这种联系会在代理对象的生命周期内一直持续存在。
Proxy 也暴露了 revocable()方法，这个方法支持撤销代理对象与目标对象之间的关联。撤销代理的操作是不可逆的。而且，撤销函数(revoke())是幂等的，调用多少次的结果都是一样的。撤销代理之后再调用代理会抛出 TypeError。
撤销函数和代理对象是在实例化时同时生成的

```javascript
const target = {
  id: "king",
};

const handler = {
  get() {
    return "Queen";
  },
};

const { proxy, revoke } = Proxy.revocable(target, handler);

console.log(target.id); // king
console.log(proxy.id); // Queen

revoke();

console.log(proxy.id); // TypeError: Cannot perform 'get' on a proxy that has been revoked
```

### 实用反射 API

#### 反射 API 与对象 API

1. 反射 API 并不限于捕获处理程序
2. 大多数反射 API 方法在 Object 类型上有对应的方法

通常，Object 上的方法适用于通用程序，而反射方法适用于细粒度的对象控制与操作

#### 标记状态

很多反射方法返回称作"状态标记"的布尔值，表示意图执行的操作是否成功。有时候，状态标记比那些返回修改后的对象或者抛出错误(取决于方法)的反射 API 更有用。以下反射方法都会提供状态标记

-   Reflect.defineProperty()
-   Reflect.preventExtensions()
-   Reflect.setPrototypeOf()
-   Reflect.set()
-   Reflect.deleteProperty()

```javascript
const o = {};

try {
  Object.defineProperty(o, "id", "King");
  console.log("success");
} catch {
  console.log("failure");
}

// 重构
if (Reflect.defineProperty(o, "age", { value: 23 })) {
  console.log("success");
} else {
  console.log("failure");
}
```

#### 用一等函数替代操作符

以下方法提供只有通过操作符才能完成的操作

-   Reflect.get(): 可以替代对象属性访问的操作符
-   Reflect.set(): 可以替代 = 赋值操作符
-   Reflect.has(): 可以替代 in 操作符或 with()
-   Reflect.deleteProperty(): 可以替代 delete 操作符
-   Reflect.construct(): 可以替代 new 操作符

#### 安全地应用函数

在通过 apply()方法调用函数时，被调用的函数可能也定义自己的 apply 属性(虽然可能性极小)。为绕过这个问题，可以使用定义在 Function 原型上的 apply 方法。

```javascript
Function.prototype.apply.call(myFunc, thisVal, argumentList);

// 这种可拍的代码完全可以使用Reflect.apply来避免
Reflect.apply(myFunc, thisVal, argumentList);
```

### 代理另一个代理

代理可以拦截反射 API 的操作，而这意味着完全可以创建一个代理，通过它去代理另一个代理。这样就可以在一个目标对象之上构建多层拦截网

```javascript
const target = {
  id: 1,
  name: "Kinsiy",
};

const firstProxy = new Proxy(target, {
  get() {
    console.log("first proxy");
    return Reflect.get(...arguments) + " first";
  },
});

const secondProxy = new Proxy(firstProxy, {
  get() {
    console.log("second proxy");
    return Reflect.get(...arguments) + " second.";
  },
});

console.log(secondProxy.name);

// second proxy
// first proxy
// Kinsiy first second.
```

### 代理的问题与不足

代理是 ECMAScript 现有基础之上构建起来的一套新 API，因此其实已经尽力做到最好了。很大程度上，代理作为对象的虚拟层可以正常使用。但在某些情况下，代理也不能与现在的 ECMAScript 机制很好地协作。

#### 代理中的 this

代理潜在的一个问题来源是 this 值。

```javascript
const target = {
  thisValEqualsProxys() {
    return this === proxy;
  },
};

const proxy = new Proxy(targer, {});

console.log(target.thisValEqualsProxys()); // false
console.log(proxy.thisValEqualsProxys()); // true
```

调用代理上的任何方法，比如 proxy.outerMethod()，而这个方法进而又会调用另一个方法，比如 this.innerMethod()，实际上都会调用 proxy.innerMethod()。多数情况下，这是符合预期的行为。可是，如果目标对象依赖于对象标识，那就可能碰到意料之外的问题

#### 代理与内部槽位

代理与内置引用类型(比如 Array)的实例通常可以很好地协同，但有些 ECMAScript 内置类型可能会依赖代理无法控制的机制，结果导致在代理上调用某些方法会出错。
一个典型的例子就是 Date 类型。根据 ECMAScript 规范，Date 类型方法的执行依赖 this 值上的内部槽位[[NumberDate]]。代理对象上不存在这个内部槽位，而且这个内部槽位的值也不能通过普通的 get()和 set()操作访问到，于是代理拦截后本应转发给目标对象的方法会抛出 TypeError。

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
