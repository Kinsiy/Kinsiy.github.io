---
title: Javascript-Object.1[理解对象]
date: 2021-01-24 01:44:54
categories: [学习笔记, Javascript]
tags: [JS红宝书, Object]
keywords:
description:
photos:
---

# 理解对象

Object 是 ECMAScript 中最常用的类型之一。虽然 Object 的实例没有多少功能，但很适合存储和在应用程序间交换数据。

<!-- more -->

```javascript
/* new Object() */
let person = new Object();
person.name = "Kinsiy";
person.age = 23;

/* 对象字面量 */
let people = {
  name: "Meng",
  age: 24,
  5: true,
};

/* 存取 */
console.log(people.name); // Meng
console.log(people["age"]); // 24

let propertyName = "name";
console.log(people[propertyName]); // Meng

people["first name"] = "ZHAO";
console.log(people["first name"]); // ZHAO
```

## 属性的类型

ECMA-262 使用一些内部特性来描述属性的特征。这些特征是由为 Javascript 实现引擎的规范定义的。因此开发者不能在 Javascript 中直接访问这些特征。为了将某个特性标识为内部特性，规范会用两个中括号括起来，比如[[Enumerable]]。

### 数据属性

数据属性包含一个保存数据值的位置。值会从这个位置读取，也会写入到这个位置。数据属性有 4 个特征描述他们的行为。

-   [[Configurable]]: 表示 属性是否可以通过 delete 删除并重新定义，是否可以修改他的特性，以及是否可以把它改为访问器属性。默认情况下，所有直接定义在对象上的属性的这个特性都是 true。
-   [[Enumerable]]: 表示属性是否可以通过 for-in 循环返回。默认情况下，所有直接定义在对象上的属性的这个特性都是 true。
-   [[Writable]]: 表示属性的值是否可以被修改。默认情况下，所有直接定义在对象上的属性的这个属性都是 true。
-   [[Value]]: 包含属性实际的值。这个值默认值为 undefined。

要修改属性的默认特性，就必须用 Object.defineProperty()方法

```javascript
let person = {};
Object.defineProperty(person, "name", {
  writable: false,
  value: "Kinsiy",
});

console.log(person.name); // Kinsiy
person.name = "Queen";
console.log(person.name); // Kinsiy
```

### 访问器属性

访问器属性不包含数据值。相反，它们包含一个获取(getter)函数和一个设置(setter)函数，不过这两个函数不是必须的。在读取访问器属性时，会调用获取函数，这个函数的责任就是返回一个有效的值。在写入访问器属性时，会调用设置函数并传入新值，这个函数必须决定对数据做出什么修改。访问器属性有 4 个特性描述它们的行为。

-   [[Configurable]]: 表示 属性是否可以通过 delete 删除并重新定义，是否可以修改它的特性，以及是否可以把它改为数据属性。默认情况下，所有直接定义在对象上的属性的这个特性都是 true。
-   [[Enumerable]]: 表示属性是否可以通过 for-in 循环返回。默认情况下，所有直接定义在对象上的属性的这个特性都是 true。
-   [[Get]]: 获取函数，在读取属性时调用。默认值为 undefined。
-   [[Set]]：设置函数，在写入属性时调用。默认值为 undefined。

访问器属性是不能直接定义的，必须使用 Object.defineProperty().

```javascript
let book = {
  year_: 2020,
  edition: 1,
};

Object.defineProperty(book, "year", {
  get() {
    return this.year_;
  },

  set(newValue) {
    if (newValue > this.year_) {
      this.year_ = newValue;
      this.edition += newValue - 2020;
    }
  },
});

book.year = 2021;
console.log(book.edition); // 2
console.log(book.year_); // 2021
```

获取函数和设置函数不一定都要定义。只定义获取函数意味着属性是只读的，尝试修改属性会被忽略。

## 定义多个属性

Object.defineProperties()。这个方法可以通过描述符一次性定义多个属性。它接收两个参数: 要为之添加或修改的对象和另一个描述符对象，其属性与要添加或修改的属性一一对应。

```javascript
let book = {};
Object.defineProperties(book, {
  year_: {
    /* 只设置value，属性不可写。省缺值均为false */
    configurable: true,
    enumerable: true,
    writable: true,
    value: 2020,
  },

  edition: {
    configurable: true,
    enumerable: true,
    writable: true,
    value: 1,
  },

  year: {
    /*
        value 和 get() 是同一个作用，只能同时用一个。
        writable 和 set() 是同一个作用，用一个
        */
    configurable: true,
    enumerable: true,
    get() {
      return this.year_;
    },
    set(newValue) {
      if (newValue > 2020) {
        this.year_ = newValue;
        this.edition += newValue - 2020;
      }
    },
  },
});

book.year = 2022;
console.log(book.edition); // 3
console.log(book.year_); // 2022
```

## 读取属性中的特性

使用 Object.getOwnPropertyDescriptor()方法可以取得指定属性的属性描述符。这个方法接收两个参数：属性所在的对象和要取得其描述符的属性名。返回值是一个对象，对于访问器属性包含 configurable、enumerable、get 和 set 属性，对于数据属性包含 configurable、enumerable、writable 和 value 属性。

```javascript
// ...
let descript = Object.getOwnPropertyDescriptor(book, "year_");
console.log(descript.configurable); // true
console.log(descript.enumerable); // true
console.log(descript.writable); // true
console.log(descript.value); // 2022
console.log(typeof descript.get); // undefined
let descript_1 = Object.getOwnPropertyDescriptor(book, "year");
console.log(typeof descript_1.get); // function

/* getOwnPropertyDescriptors */
console.log(Object.getOwnPropertyDescriptors(book));
/*
    {
      year_: {
        value: 2022,
        writable: true,
        enumerable: true,
        configurable: true
      },
      edition: {
        value: 3,
        writable: true,
        enumerable: true,
        configurable: true
      },
      year: {
        enumerable: true,
        configurable: true
        get: ƒ get()
        set: ƒ set(newValue)
      }
    }
*/
```

## 合并对象

ECMAScript6 专门为合并对象提供了 Object.assign()方法。这个方法接收一个目标对象和一个或多个源对象作为参数，然后将每个源对象中可枚举(Object.propertyIsEnumerable()返回 true)和自用(Object.hasOwnProperty()返回 true)属性复制到目标对象。以字符串和符号为键的属性会被复制。对每个符合条件的属性，这个方法会使用源对象上的[[Get]]取得属性的值，然后谁用目标对象上的[[Set]]设置属性的值。

```javascript
let desc = {};

/**
 *  Object.assign()实际上对每个源对象执行的是浅复制。
 *  如果多个源对象都有相同的属性，则使用最后一个复制的值
 */
let result = Object.assign(dest, { id: "K" }, { id: "I" }, { name: "Kinsiy" });

console.log(desc === result); // true
console.log(desc); // {id: "I", name: "Kinsiy"}

/**
 *  如果赋值期间出错，则操作会中止并退出，同时抛出错误。
 *  Object.assign()没有"回滚"之前赋值的概念，因此它是一个尽力而为、可能只会完成部分复的方法
 */

let dest = {};
let scr = {
  a: "Queen",
  get b() {
    throw new Error();
  },
  c: "Kinsiy",
};

try {
  Object.assign(dest, scr);
} catch (e) {}

console.log(dest); // {a: "Queen"}
```

## 对象标识及相等判定

```javascript
console.log(Object.is(true, 1)); // false
console.log(Object.is({}, {})); // false
console.log(Object.is("2", 2)); // false

console.log(Object.is(+0, -0)); // false
console.log(Object.is(+0, 0)); // true
console.log(Object.is(-0, 0)); // false

console.log(Object.is(NaN, NaN)); // true

/**
 *  检查超过两个值
 */

function recursivelyCheckEaual(x, ...rest) {
  return Object.is(x, rest[0]) && (rest.length < 2 || recursivelyCheckEaual(...rest));
}
```

## 增强的对象语法

### 属性值简写

```javascript
let name = "Kinsiy",
  age = 23;

let person = {
  name,
  age,
};

console.log(`name: ${person.name}   age: ${person.age}`); // name: Kinsiy   age: 23
```

### 可计算属性

可以在对象字面量中完成动态属性赋值。中括号包围的对象属性键告诉运行时将其作为 Javascript 表达式而不是字符串来求值

```javascript
const nameKey = "name";
const ageKey = "age";
const jobKey = "job";
let uniqueToken = 0;

function getUniqueKey(key) {
  return `${key}_${uniqueToken++}`;
}

let person = {
  [getUniqueKey(nameKey)]: "Kinsiy",
  [getUniqueKey(ageKey)]: "23",
  [getUniqueKey(jobKey)]: "Software engineer",
};

console.log(person); // {name_0: "Kinsiy", age_1: "23", job_2: "Software engineer"}
```

### 简写方法名

```javascript
let person = {
  name: "Kinsiy",
  sayName() {
    console.log(this.name);
  },
};

person.sayName(); // Kinsiy

/**
 *  与可计算属性键相互兼容
 */
const methodKey = "sayAge";
let person_2 = {
  [methodKey](age) {
    console.log(age);
  },
};

person_2.sayAge("23"); // 23
```

## 对象解构

```javascript
let person = {
  name: "Kinsiy",
  age: 23,
};

let { name: myName, age, job = "Software engineer" } = person;

console.log(myName); // Kinsiy
console.log(age); // 23
console.log(job); // Software engineer

/**
 *  事先声明变量时
 *  赋值表达式必须包含在一对括号中
 */

let OtherName, newAge;
({ name: OtherName, age: newAge } = person);
console.log(OtherName); // Kinsiy
console.log(newAge); // 23
```

### 嵌套解构

```javascript
let person = {
  name: "Kinsiy",
  age: 23,
  job: {
    title: "Software engineer",
  },
};
let otherPerson = {};

({ name: otherPerson.name, age: otherPerson.age, job: otherPerson.job } = person);
person.job.title = "Hacker";

console.log(otherPerson); // {name: "Kinsiy", age: 23, job: {Hacker}}
console.log(person); // {name: "Kinsiy", age: 23, job: {Hacker}}

let {
  job: { title: newTitle },
} = person;
console.log(newTitle); // Hacker

// 在外层属性没有定义的情况下不能使用嵌套解构。无论对象还是目标对象都是一样
let personCopy = {};
// foo在源对象上是undefined
({foo: { bar: personCopy.bar }} = person); // Uncaught TypeError: Cannot read property 'bar' of undefined

// job在目标对象上是undefined
({job: { titilt: personCopy.job.title }} = person); // Uncaught TypeError: Cannot set property 'title' of undefined
```

### 部分解构

涉及多个属性的解构赋值时一个输出无关非顺序化操作。如果一个解构表达式涉及多个赋值，开始的赋值成功而后面的赋值出错，则整个解构赋值只会完成一部分。

```javascript
let person = {
  name: "Kinsiy",
  age: 23,
};

let name, bar, age;
try {
  ({
    name: name,
    job: { bar: bar },
    age: age,
  } = person);
} catch (e) {}

console.log(name, bar, age); // Kinsiy undefined undefined
```

### 参数上下文匹配

在函数参数列表也可以进行解构赋值。对参数的解构赋值不会影响 arguments 对象，但可以在函数签名中声明函数体内使用局部变量：

```javascript
let person = {
  name: "Kinsiy",
  age: 23,
};

function getPersonInfo(foo, { name, age }, bar) {
  console.log(arguments);
  console.log(name, age);
}

getPersonInfo("1st", person, "2st");

// ["1st", {name: "Kinsiy", age: 23}, "2st"]
// Kinsiy 23
```
