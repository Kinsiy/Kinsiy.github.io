---
title: Javascript-Proxy[捕获器与反射]
date: 2021-03-07 19:26:58
categories: [学习笔记, Javascript]
tags: [JS红宝书, Proxy]
keywords:
description:
photos:
---

## 代理捕获器与反射方法

### get()

get()捕获器会在获取属性值得操作中被调用。对应的反射 API 方法为 Reflect.get()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  get(target, property, receiver) {
    console.log("get()");
    return Reflect.get(...arguments);
  },
});

proxy.id; // get()
```

<!-- more -->

1. 返回值
    - 返回值无限制
2. 拦截的操作
    - proxy.property
    - proxy[property]
    - Object.create(proxy)[property]
    - Reflect.get(proxy, property, receiver)
3. 捕获器处理程序函数
    - target: 目标对象
    - property: 引用的目标对象上的字符串属性
    - receiver：代理对象或继承代理对象的对象
4. 捕获器不变式
    - 如果 target.property 不可写且不可配置，则处理程序返回值必须与 target.property 匹配。
    - 如果 target.property 不可配置且[[Get]]特性为 undefined，处理程序的返回值也必须是 undefined。

### set()

set()捕获器会在设置属性值得操作中被调用。对应的反射 API 方法为 Reflect.set()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  set(target, property, value, receiver) {
    console.log("set()");
    return Reflect.set(...arguments);
  },
});

proxy.id = "Kinsiy"; // set()
```

1. 返回值
    - 返回 true 表示成功；返回 false 表示失败，严格模式下会抛出 TypeError
2. 拦截的操作
    - proxy.property = value
    - proxy[property] = value
    - Object.create(proxy)[property] = value
    - Reflect.set(proxy, property, value, receiver)
3. 捕获器处理程序函数
    - target: 目标对象
    - property: 引用的目标对象上的字符串属性
    - value: 要赋给属性的值
    - receiver：代理对象或继承代理对象的对象
4. 捕获器不变式
    - 如果 target.property 不可写且不可配置，则不能修改目标属性的值
    - 如果 target.property 不可配置且[[Set]]特性为 undefined，则不能修改目标属性的值
    - 在严格模式下，处理程序中返回 false 会抛出 TypeError

### has()

set()捕获器会在 in 操作符中被调用。对应的反射 API 方法为 Reflect.has()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  has(target, property) {
    console.log("has()");
    return Reflect.has(...arguments);
  },
});

console.log("id" in proxy); //has() false
```

1. 返回值
    - has()必须返回布尔值，表示属性是否存在。返回非布尔值会被转型为布尔值
2. 拦截的操作
    - property in proxy
    - property in Object.create(proxy)
    - With(proxy){(property;)}
    - Reflect.has(proxy, property)
3. 捕获器处理程序函数
    - target: 目标对象
    - property: 引用的目标对象上的字符串属性
4. 捕获器不变式
    - 如果 target.property 存在且不可配置，则处理程序必须返回 false
    - 如果 target.property 存在且目标对象不可扩展，则处理程序必须返回 true

### defineProperty()

defineProperty()捕获器会在 Object.defineProperty()中被调用。对应的反射 API 方法为 Reflect.defineProperty()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  defineProperty(target, property, descriptor) {
    console.log("defineProperty()");
    return Reflect.defineProperty(...arguments);
  },
});

Object.defineProperty(proxy, "id", {
  value: "Queen",
}); // defineProperty()

console.log(proxy.id); // Queen
```

1. 返回值
    - defineProperty()必须返回布尔值，表示属性是否成功定义。返回非布尔值会被转型为布尔值
2. 拦截的操作
    - Object.defineProperty(peoxy, property, descriptor)
    - Reflect.defineProperty(peoxy, property, descriptor)
3. 捕获器处理程序函数
    - target: 目标对象
    - property: 引用的目标对象上的字符串属性
    - descriptor: 包含可选的 enumerable、configurable、writable、value、get 和 set 定义的对象。
4. 捕获器不变式
    - 如果目标对象不可扩展，则无法定义属性
    - 如果目标对象有一个可配置的属性，则不能添加同名的不可配置属性
    - 如果目标对象有一个不可配置的属性，则不能添加同名的可配置属性

### getOwnPropertyDescriptor()

getOwnPropertyDescriptor()捕获器会在 Object.getOwnPropertyDescriptor()中被调用。对应的反射 API 方法为 Reflect.getOwnPropertyDescriptor()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  getOwnPropertyDescriptor(target, property) {
    console.log("getOwnPropertyDescriptor()");
    return Reflect.getOwnPropertyDescriptor(...arguments);
  },
});

Object.getOwnPropertyDescriptor(proxy, "id"); // getOwnperpertyDescriptor()
```

1. 返回值
    - getOwnPropertyDescriptor()必须返回对象，或者属性不存在时返回 undefined。
2. 拦截的操作
    - Object.getOwnPropertyDescriptor(peoxy, property)
    - Reflect.getOwnPropertyDescriptor(peoxy, property)
3. 捕获器处理程序函数
    - target: 目标对象
    - property: 引用的目标对象上的字符串属性
4. 捕获器不变式
    - 如果自有的 target.property 存在且不可配置，则处理程序必须返回一个表示该属性存在的对象
    - 如果自有的 target.property 存在且可配置，则处理程序必须返回表示该属性可配置的对象
    - 如果自有的 target.property 存在且 target 不可扩展，则处理程序必须返回一个表示该属性存在的对象
    - 如果自有的 target.property 不存在，则处理程序不能返回表示该属性可配置的对象

### deleteProperty()

deleteProperty()捕获器会在 delete 操作符中被调用。对应的反射 API 方法为 Reflect.deleteProperty()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  deleteProperty(target, property) {
    console.log("deleteProperty()");
    return Reflect.deleteProperty(...arguments);
  },
});

delete proxy.id; // deleteProperty()
```

1. 返回值
    - defineProperty()必须返回布尔值，表示删除属性是否成功。返回非布尔值会被转型为布尔值。
2. 拦截的操作
    - delete proxy.property
    - delete proxy[property]
    - Reflect.deleteProperty(peoxy, property)
3. 捕获器处理程序函数
    - target: 目标对象
    - property: 引用的目标对象上的字符串属性
4. 捕获器不变式
    - 如果自有的 target.property 存在且不可配置，则处理程序不能删除这个属性

### ownKeys()

ownKeys()捕获器会在 Object.keys() 及类似方法中被调用。对应的反射 API 方法为 Reflect.ownKeys()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  ownKeys(target) {
    console.log("ownKeys()");
    return Reflect.ownKeys(...arguments);
  },
});

Object.keys(proxy); // ownKeys()
```

1. 返回值
    - ownKeys()必须返回包含字符串或符号的可枚举对象
2. 拦截的操作
    - Object.getOwnPropertyNames(proxy)
    - Object.getOwnPropertySymbols(proxy)
    - Object.keys(proxy)
    - Reflect.ownKeys(peoxy)
3. 捕获器处理程序函数
    - target: 目标对象
4. 捕获器不变式
    - 返回的可枚举对象必须包含 target 的所有不可配置的自有属性
    - 如果 target 不可扩展，则返回可枚举对象必须准确地包含自有属性键

### getPrototypeOf()

getPrototypeOf()捕获器会在 Object.getPrototypeOf() 中被调用。对应的反射 API 方法为 Reflect.getPrototypeOf()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  getPrototypeOf(target) {
    console.log("getPrototypeOf()");
    return Reflect.getPrototypeOf(...arguments);
  },
});

/**
 * Object.getPrototypeOf() 返回指定对象的原型，内部[[prototype]]属性的值
 */
Object.getPrototypeOf(proxy); // getPrototypeOf()
```

1. 返回值
    - getPrototypeOf()必须返回对象或 null。
2. 拦截的操作
    - Object.getPrototypeOf(proxy)
    - Reflect.getPrototypeOf(proxy)
    - proxy._\_proto_\_
    - Object.prototype.isPrototypeOf(proxy)
    - proxy instanceof Object
3. 捕获器处理程序函数
    - target: 目标对象
4. 捕获器不变式
    - 如果 target 不可扩展，则 Object.getPrototypeOf(proxy)唯一有效的返回值就是 Object.getPrototypeOf(target)的返回值

### setPrototypeOf()

setPrototypeOf()捕获器会在 Object.setPrototypeOf() 中被调用。对应的反射 API 方法为 Reflect.setPrototypeOf()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  setPrototypeOf(target, prototype) {
    console.log("setPrototypeOf()");
    return Reflect.setPrototypeOf(...arguments);
  },
});

/**
 * Object.setPrototypeOf() 向实例的私有特性[[Prototype]]写入一个新值
 */
Object.getPropertypeOf(proxy, Object); // setPrototypeOf()
```

1. 返回值
    - setPrototypeOf()必须返回布尔值，表示值原型赋值是否成功。返回非布尔值会被转型为布尔值。
2. 拦截的操作
    - Object.setPrototypeOf(proxy, prototype)
    - Reflect.setPrototypeOf(proxy, prototype)
3. 捕获器处理程序函数
    - target: 目标对象
    - prototype: target 的替代原型，如果是顶级原型则为 null
4. 捕获器不变式
    - 如果 target 不可扩展，则唯一有效的 prototype 参数就是 Object.getPrototypeOF(target)的返回值

### isExtensible()

isExtensible()捕获器会在 Object.isExtensible() 中被调用。对应的反射 API 方法为 Reflect.isExtensible()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  isExtensible(target) {
    console.log("isExtensible()");
    return Reflect.isExtensible(...arguments);
  },
});

/**
 * Object.isExtensible() 方法判断一个对象是否是可扩展的（是否可以在它上面添加新的属性）。
 * Object.preventExtensions，Object.seal 或 Object.freeze 方法都可以标记一个对象为不可扩展（non-extensible）。
 */
Object.isExtensible(proxy); // isExtensible()
```

1. 返回值
    - isExtensible()必须返回布尔值，表示 target 是否可扩展。返回非布尔值会被转型为布尔值。
2. 拦截的操作
    - Object.isExtensible(proxy)
    - Reflect.isExtensible(proxy)
3. 捕获器处理程序函数
    - target: 目标对象
4. 捕获器不变式
    - 如果 target 可扩展，则处理程序必须返回 true
    - 如果 target 不可扩展，则处理程序必须返回 false

### preventExtensions()

preventExtensions()捕获器会在 Object.preventExtensions() 中被调用。对应的反射 API 方法为 Reflect.preventExtensions()

```javascript
const myTarget = {};

const proxy = new Proxy(myTarget, {
  preventExtensions(target) {
    console.log("preventExtensions()");
    return Reflect.preventExtensions(...arguments);
  },
});

/**
 * Object.preventExtensions()方法让一个对象变的不可扩展，也就是永远不能再添加新的属性。
 */
Object.preventExtensions(proxy); // preventExtensions()
```

1. 返回值
    - preventExtensions()必须返回布尔值，表示 target 是否已经不可扩展。返回非布尔值会被转型为布尔值。
2. 拦截的操作
    - Object.preventExtensions(proxy)
    - Reflect.preventExtensions(proxy)
3. 捕获器处理程序函数
    - target: 目标对象
4. 捕获器不变式
    - 如果 Object.isExtensible(proxy)是 false，则处理程序必须返回 true

### apply()

apply()捕获器会在调用函数时被调用。对应的反射 API 方法为 Reflect.apply()

```javascript
const myTarget = () => {};

const proxy = new Proxy(myTarget, {
  apply(target, thisArg, argumentsList) {
    console.log("apply()");
    return Reflect.apply(...arguments);
  },
});

proxy(); // apply()
```

1. 返回值
    - 无限值
2. 拦截的操作
    - proxy(...argumentsList)
    - Function.prototype.apply(thisArg, argumentsList)
    - Function.prototype.call(thisArg, ...argumentsList)
    - Reflect.apply(target, thisArgument, argumentsList)
3. 捕获器处理程序函数
    - target: 目标对象
    - thisArg: 调用函数时的 this 参数
    - argumentsList: 调用函数时的参数列表
4. 捕获器不变式
    - target 必须是一个函数对象

### construct()

construct()捕获器会在 new 操作符中被调用。对应的反射 API 方法为 Reflect.construct()

```javascript
const myTarget = function () {};

const proxy = new Proxy(myTarget, {
  construct(target, argumentsList, newTarget) {
    console.log("construct()");
    return Reflect.construct(...arguments);
  },
});

new proxy(); // construct()
```

1. 返回值
    - construct()必须返回一个对象
2. 拦截的操作
    - new proxy(...argumentsList)
    - Reflect.construct(target, argumentsList, newTarget)
3. 捕获器处理程序函数
    - target: 目标对象
    - argumentsList: 调用函数时的参数列表
    - newTarget: 最初被调用的构造函数
4. 捕获器不变式
    - target 必须可以用作构造函数

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)