---
title: Javascript-Map&Set
date: 2021-01-29 22:31:45
categories: [学习笔记, Javascript]
tags: [JS红宝书, Map, Set]
keywords:
description: 集合引用类型, Map()、WeakMap()、Set()、WeakSet()的使用
photos:
---

# Map

Map 是一种新的集合类型，为 Javascript 带来了真正的键/值存储机制。Map 的大多数特性都可以通过 Object 类型实现，但二者之间还是存在一些细微的差异。具体实践中使用哪一个值得细细甄别。

## 基本 API

创建 Map 实例并初始化实例，可以给 Map() 构造函数传入一个可迭代对象，需要包含键/值对数组。可迭代对象的每个键/值对都会按照迭代顺序插入到新映射实例中

```javascript
const m_1 = new Map(); // 创建空映射

const m_2 = new Map([
  ["key_1", "val_1"],
  ["key_2", "val_2"],
  ["key_3", "val_3"],
]); // 创建的同时初始化实例
console.log(m_2.size); // 3

const m_3 = new Map([[]]);
console.log(m_3.has(undefined)); // true
console.log(m_3.get(undefined)); // undefined
```

初始化后，可以使用 set()方法在添加键/值对。另外，可以使用 get()和 has()进行查询，可以通过 size 属性获取映射中的键/值对的数量，还可以使用 delete()和 clear()删除值。delete()返回一个布尔值，表示集合中是否存在要删除的键.

```javascript
const m_4 = new Map([
  ["firstName", "Kinsiy"],
  ["lastName", "Restituo"],
]);

m_4.set("age", "23");
console.log(m_4); // {"firstName" => "Kinsiy", "lastName" => "Restituo", "age" => "23"}
console.log(m_4.has("firstName")); // true
console.log(m_4.get("lastName")); // Restituo
console.log(m_4.size); // 3

console.log(m_4.delete("age")); // true 只删除这一个键/值对
console.log(m_4); // {"firstName" => "Kinsiy", "lastName" => "Restituo"}

m_4.clear(); // 清除这个映射实例中的所有键/值对
console.log(m_4); // {}
```

在映射中用作键和值得对象及其他"集合"类型，在自己的内容或属性被修改时仍然保持不变：

```javascript
const m_5 = new Map();
const obj_key = {},
  obj_val = {},
  arr_key = [],
  arr_val = [];

m_5.set(obj_key, obj_val).set(arr_key, arr_val);

obj_key.foo = "foo";
obj_val.bar = "bar";
arr_key.push("Kinsiy");
arr_val.push("Restituo");

console.log(m_5.get(obj_key)); // {bar: "bar"}
console.log(m_5.get(arr_key)); // ["Restituo"]
```

## 顺序与迭代

与 Object 类型的一个主要的差异是，Map 实例会维护键/值对的插入顺序，因此可以按照插入顺序执行迭代操作。

```javascript
const m_6 = new Map([
  ["one", "Javascript"],
  ["two", "Java"],
  ["three", "Python"],
  ["four", "C"],
]);
console.log(m_6.entries === m_6[Symbol.iterator]); // true

/* entries(),keys(),values() */
for (let [key, value] of m_6.entries()) {
  console.log(`key: ${key}    value: ${value}`);
}
// key: one    value: Javascript
// key: two    value: Java
// key: three    value: Python
// key: four    value: C
for (let key of m_6.keys()) {
  console.log(key); // one two three four
}
for (let value of m_6.values()) {
  console.log(value); // Javascript Java Python C
}

console.log([...m_6]); //[["one", "Javascript"], ["two", "Java"], ["three", "Python"], ["four", "C"]]
```

键和值在迭代器遍历时时可以修改的，但映射内部的引用则无法修改。当然，这并不妨碍修改作为键或值的对象内部的属性。因为这样并不英雄他们在映射实例中的身份。

```javascript
const obj_key = { id: 1 };
const m_7 = new Map([[obj_key, "Kinsiy"]]);

for (let key of m_7.keys()) {
  key.id = 8;
  console.log(key); // {id: 8}
  console.log(m_7.get(key)); // Kinsiy
}
```

## 选择 Object 还是 Map

1. <b>内存占用。</b> 给定固定大小的内存，Map 大约可以比 Object 多存储 50%的键/值对。
2. <b>插入性能。</b> 两者消耗大致相当，不过 Map 在所有浏览器中一般会稍微快一点儿。
3. <b>查找速度。</b> 从大型 Object 和 Map 中查找键/值对的性能差异极小，但如果只包含少量键值对,则 Object 有时候速度更快。如果代码涉及大量查找操作，那么某些时候可能选择 Object 更好一些
4. <b>删除性能。</b> 对大多浏览器引擎来说，Map 的 delete()操作都比插入和查找更快。如果代码涉及大量删除操作，那么毫无疑问应该选择 Map。

# WeakMap

WeakMap 是 Map 的"兄弟"类型，其 API 也是 Map 的子集。WeakMap 中的"Weak" 描述的是 Javascript 垃圾回收程序对待"弱映射"中键的方式。

## 基本 API

弱映射的键只能是 Object 或继承自 Object 的类型。尝试使用非对象设置键会抛出 TypeError，值得类型没有限制。API 包括 set()、has()、get()、delete()

```javascript
const obj_1 = { id: 1 },
  obj_2 = { id: 2 },
  obj_3 = { id: 3 };
/* 初始化是全有或全无的操作
    只要有一个键无效就会抛出错误，导致整个初始化失败
 */
const m_1 = new WeakMap([
  [obj_1, "value_1"],
  ["Str_obj_2", "value_2"],
  [obj_3, "value_3"],
]); // TypeError: Invalid value used as weak map key
```

## 弱键

WeakMap 中"weak"表示弱映射的键时"弱弱地拿着"的。意思就是，这些键不属于正式的引用，不会阻止垃圾回收。但要注意的是，弱映射中值的引用可不是"弱弱地拿着"的。只要键存在，键值对就会存在于映射中，并被当作值得引用，因此就不会被当作垃圾回收。

```javascript
/*  由于没有指向这个对象({})的其他引用，这行代码执行完之后，这个对象键就会被当作垃圾回收。
    键值对被破坏，值也会成为垃圾回收的目标
*/
const wm_1 = new WeakMap([[{}, "value_1"]]);

/* container 维护着一个对弱映射键的引用，这个键不会成为垃圾回收的目标
    如果调用了remove_Reference，就会摧毁对象的最后一个引用，垃圾回收程序会把这个键值对清理掉
 */
const container = {
  key: {},
};
const wm_2 = new WeakMap([[container.key, "value_2"]]);

function remove_Reference() {
  container.key = null;
}
```

## 使用弱映射

### 私有变量

(有些不理解，待研究)

```javascript
const User = (() => {
  const wm = new WeakMap();

  class User {
    constructor(id) {
      this.idProperty = Symbol("id");
      this.setId(id);
    }

    setPrivate(property, value) {
      const privateMemers = wm.get(this) || {};
      privateMemers[property] = value;
      wm.set(this, privateMembers);
    }

    getPrivate(property) {
      return wm.get(this)[property];
    }

    setId(id) {
      this.setPrivate(this.idProperty, id);
    }

    getId(id) {
      return this.getPrivate(this.idProperty);
    }
  }
  return User;
})();

const user = new User(123);
console.log(user.getId()); // 123
```

### DOM 节点元数据

```javascript
const wm_1 = new WeakMap();
const OneDom = document.querySelector("#login");
wm.set(OneDom, { disabled: true });

// 当节点从DOM树删除后，垃圾回收程序就可以立即释放其内存(架设没有其他地方引用这个对象)。
```

# Set

集合数据结构。Set 在很多方面都像是加强的 Map，这是因为他们的大多数 API 和行为都是共有的。

## 基本 API

与 Map 相比，Set 的 API 基本与 Map 一致，不过 Map 的 set()方法，在 Set 这边变为 add(),功能类似,均为向集合中新增元素。

```javascript
const s_1 = new Set(["val_1", "val_2", "val_3"]);

console.log(s_1); // {"val_1", "val_2", "val_3"}
s_1.add("val_4").add("val_5");
console.log(s_1); // {"val_1", "val_2", "val_3", "val_4", "val_5"}
```

## 顺序与迭代

与 Map 相比，Set 的迭代方法也大都一致。Set 的 entries()方法返回一个迭代器，可以按照插入顺序产生包含两个元素的数组，这两个元素是集合中每个值得重复出现。

```javascript
const s_2 = new Set(["Kinsiy", "Restituo", "Supreman"]);

console.log(s_2); // {"Kinsiy", "Restituo", "Supreman"}
for (let v of s_2.entries()) {
  console.log(v); // ["Kinsiy", "Kinsiy"] ["Restituo", "Restituo"] ["Supreman", "Supreman"]
  v = "Super"; // 修改集合中值得属性不会影响其作为集合值得身份
}
console.log(s_2); // {"Kinsiy", "Restituo", "Supreman"}
```

# WeakSet

与 Map 和 WeakMap 的关系类似，WeakSet 也是 Set 的"兄弟"类型，其 API 也是 Set 的子集。"weak"描述的是 Javascript 垃圾回收程序对待"弱集合"中值得方式。弱集合中的值只能是 Object 或者继承自 Object 的类型。

## 基本 API

包括 add()、has()、delete()

```javascript
const obj_1 = { id: 2 };
const s_3 = new WeakSet();
s_3.add(obj_1);
console.log(s_3.has(obj_1)); // true
s_3.delete(obj_1);
console.log(s_3.has(obj_1)); // false
```

## 不可迭代值

因为 WeakSet 中的值任何时候都可能被销毁，所以没必要提供迭代其值得能力。当然，也用不着向 clear()这样一次性销毁所有值得方法。

## 使用弱集合

打标签

```javascript
const disabledElements = new WeakSet();
const loginButton = document.querySelector("#login");
disabledElements.add(loginButton);
// 只要WeakSet 中任何元素从DOM树中被删除，垃圾回收程序就可以忽略其存在，而立即释放其内存(假设没有其他地方引用这个对象)
```

# 迭代与扩展操作

有四种原生集合定义了默认迭代器：

-   Array
-   所有的定型数组
-   Map
-   Set

这意味着上述所有类型都支持顺序迭代，都可以传入 for-of 循环。这也意味着这些类型都兼容扩展操作符。扩展操作符在对可迭代对象执行浅复制时特别有用，只需简单的语法就可以复制整个对象：

```javascript
let arr_1 = [1, 2, 3, 4];
let arr_2 = [...arr_1];
console.log(arr_1 === arr_2); // false

/* 用于构建数组的部分元素 */
let arr_3 = [5, 2, ...arr_1, 9, 8]; // [5, 2, 1, 2, 3, 4, 9, 8]
console.log(arr_3);

/* 对于期待可迭代对象的构造函数，只要传入一个可迭代对象就可以实现复制 */
let map_1 = new Map([
  ["key_1", "1"],
  ["key_2", "2"],
]);
let map_2 = new Map(map_1);
console.log(map_2); // {"key_1" => "1", "key_2" => "2"}

/* 浅复制意味着只会复制对象的引用 */
let arr_4 = [{}];
let arr_5 = [...arr_4];
arr_4[0].name = "Kinsiy";
console.log(arr_5[0]); // {name: "Kinsiy"}
```
