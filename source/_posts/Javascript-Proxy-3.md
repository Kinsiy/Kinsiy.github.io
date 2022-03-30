---
title: Javascript-Proxy[代理模式]
date: 2021-03-07 19:28:44
categories: [学习笔记, Javascript]
tags: [JS红宝书, Proxy]
keywords:
description:
photos:
---

## 代理模式

使用代理可以在代码中实现一些有用的编程模式

### 跟踪属性访问

通过捕获 get、set 和 has 等操作，可以知道对象属性什么时候被访问、被查询。把实现相应捕获器的某个对象代理放在应用中，可以监控这个对象何时在何处被访问过。

<!-- more -->

```javascript
const user = {
  name: "Kinsiy",
};

const proxy = new Proxy(user, {
  get(target, property, receiver) {
    console.log(`Getting ${property}`);
    return Reflect.get(...arguments);
  },

  set(target, property, value, receiver) {
    console.log(`Setting ${property} = ${value}`);
    return Reflect.set(...arguments);
  },
});

proxy.name; // Getting name
proxy.age = 23; // Setting age = 23
```



### 隐藏属性

代理的内部实现对外部代码是不可见的，因此要隐藏目标对象上的属性也轻而易举。

```javascript
const hiddenProperties = ["King", "Queen"];
const targetObject = {
  name: "Kinsiy",
  King: "青睐",
  Queen: "Restituo",
};

const proxy = new Proxy(targetObject, {
  get(target, property, receiver) {
    if (hiddenProperties.includes(property)) {
      return undefined;
    } else {
      return Reflect.get(...arguments);
    }
  },
  has(target, property) {
    if (hiddenProperties.includes(property)) {
      return false;
    } else {
      return Reflect.has(...arguments);
    }
  },
});

// get
console.log(proxy.name); // Kinsiy
console.log(proxy.King); // undefined
console.log(proxy.Queen); // undefined

// has
console.log("name" in proxy); // true
console.log("King" in proxy); // false
console.log("Queen" in proxy); // false
```

### 属性验证

因为所有属性操作都会触发 set()捕获器，所以可以根据所赋的值决定是允许还是拒绝

```javascript
const target = {
  onlyNumbersGoHere: 0,
};

const proxy = new Proxy(target, {
  set(target, property, value, receiver) {
    if (typeof value !== "number") {
      return false;
    } else {
      return Reflect.set(...arguments);
    }
  },
});

proxy.onlyNumbersGoHere = 1;
console.log(proxy.onlyNumbersGoHere); // 1
proxy.onlyNumbersGoHere = "2";
console.log(proxy.onlyNumbersGoHere); // 1
```

### 函数与构造函数参数验证

跟保护和验证对象属性类似，也可以对函数和构造函数参数进行审查。比如，可以让函数只接收某种类型的值

```javascript
function median(...nums) {
  return nums.sort()[Math.floor(nums.length / 2)];
}

const proxy = new Proxy(median, {
  apply(target, thisArg, argumentsList) {
    for (const arg of argumentsList) {
      if (typeof arg !== "number") {
        throw new Error("参数中存在非数字！");
      }
    }
    return Reflect.apply(...arguments);
  },
});

console.log(proxy(1, 2, 4, 5)); // 4
console.log(proxy(1, 2, 4, "5")); // Error: 参数中存在非数字！

// 类似地，可以要求实例化时必须给构造函数传参

class Uesr {
  constructor(id) {
    this._id = id;
  }
}

const proxy = new Proxy(User, {
  construct(target, argumentsList, newTarget) {
    if (argumentsList[0] === undefined) {
      throw new Error("用户类必须提供id");
    } else {
      return Reflect.construc(...arguments);
    }
  },
});

new proxy(1);
new proxy(); // Error: 用户类必须提供id
```

### 数据绑定与可观察对象

通过代理可以把运行时原本不相关的部分联系到一起。这样就可以实现各种模式，从而让不同代码互操作。

```javascript
const userList = [];

class User {
  constructor(name) {
    this.name = name;
  }
}
// 还可以把集合绑定到一个事件分派程序，每次插入新实例时都会发送消息
function log(newValue) {
  console.log(newValue);
}

const proxy_1 = new Proxy(userList, {
  set(target, property, receiver) {
    const result = Reflect.set(...arguments);
    if (result && property !== "length") {
      // 这里控制一下新增元素后，数组长度变化的消息无需发送
      log(Reflect.get(target, property, receiver));
    }
    return result;
  },
});

// 可以将被代理的类绑定到一个全局实例集合，让所有创建的实例都被添加到这个集合中
const proxy = new Proxy(User, {
  construct(target, argumentsList, newTarget) {
    const newUser = Reflect.construct(...arguments);
    proxy_1.push(newUser); // 发送消息
    return newUser;
  },
});

new proxy("Kinsiy");
new proxy("King");
new proxy("Queen");

// User {name: "Kinsiy"}
// User {name: "King"}
// User {name: "Queen"}
console.log(userList); // [User, User, User]
```
