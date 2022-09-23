---
title: Javascript-Array
date: 2021-01-24 14:47:28
categories: [学习笔记, Javascript]
tags: [JS红宝书, Array]
keywords:
description: 
photos:
---

> 数组是一种类列表对象，它的原型中提供了遍历和修改元素的相关操作。JavaScript 数组的长度和元素类型都是非固定的。因为数组的长度可随时改变，并且其数据在内存中也可以不连续，所以 JavaScript 数组不一定是密集型的，这取决于它的使用方式
>
> ——MDN. Array

<!-- more -->

## 数组基础

### 创建数组

创建数组时可以给构造函数传入一个值，如果这个值是数值，会创建一个长度为指定数值的数组。如果这个值是其他类型的，会创建一个只包含该特定值的数组。

```javascript
/* 使用Array构造函数 */
let colors_1 = new Array(20); // 创建一个初始length为20的数组
let colors_2 = new Array("red", "pink", "blue");

/* 使用数组字面量 */
let colors_3 = ["green", "yellow", "black"];

/* 静态方法from()、of()
    from()用于将类数组结构转换为数组实例
    of()用于将一组参数转换为数组实例
*/

/* Array.from()的第一个参数是一个类数组对象，即任何可迭代的结构，或者有一个length属性和可索引元素的结构。 */
console.log(Array.from("Kinsiy")); // ["K","i","n","s","i","y"]

const m = new Map().set(1, 3).set(2, 4);
const s = new Set().add(5).add(6).add(8);
console.log(Array.from(m)); // [[1,3], [2,4]]
console.log(Array.from(s)); // [5, 6, 8]

// 浅复制
const arr_1 = [1, 2, 3, 4];
const arr_2 = Array.from(arr_1);
console.log(arr_2); // [1, 2, 3, 4]
console.log(arr_1 === arr_2); // false

// 转换arguments 对象
function getArgsArray() {
  return Array.from(arguments);
}

console.log(getArgsArray(8, 9, 4, 5)); // [8, 9, 4, 5]

// Array.from()可以接收第二个可选的映射函数参数。
// 还可以接收第三个可选参数, 用于指定映射函数中this的值。(在剪头函数中不适用)
const arr_3 = [1, 2, 3, 4];
const arr_4 = Array.from(arr_3, (x) => {
  return x ** 2;
});
const arr_5 = Array.from(
  arr_3,
  function (x) {
    return x ** this.exponent;
  },
  { exponent: 3 }
);
console.log(arr_3); // [1, 2, 3, 4]
console.log(arr_4); // [1, 4, 9, 16]
console.log(arr_5); // [1, 8, 27, 64]

/* Array.of() 可以把一组参数转换为数组 */
console.log(Array.of("K", "i", "n", "s", "i", "f")); // ["K","i","n","s","i","f"]
console.log(Array.of(undefined)); // [undefined]
```

### 数组空位

使用数组字面量初始化数组时，可以使用一串逗号来创建空位(hole).ECMAScript 会将逗号之间相应的索引位置的值当成空位。ES6 新增方法普遍将这些空位当成存在的元素，只不过值是 undefined

```javascript
const options = [5, , , , 4];
for (const val of Array.from(options)) {
  console.log(val === undefined); // false true true true false
}
```

由于行为不一致和存在性能隐患，因此实践中要避免使用数组空位。如果确实需要空位，则可以显式地用 undefined 值替代

### 数组索引

数组中最后一个元素的索引始终是 length-1，因此下一个新增槽位的索引就是 length。每次在数组最后一个元素后面新增一项，数组的 length 属性都会自动更新。注意 length 不是只读的，可以通过修改 length 的值删除数组元素

```javascript
let arr_1 = [5, 7, 9, 3, 15, 88];
console.log(arr_1); // [5,7,9,3,15,88]

arr_1.length = 5;
console.log(arr_1); // [5,7,9,3,15]

arr_1[arr_1.length] = 99;
console.log(arr_1); // [5,7,9,3,15,99]
```

### 检测数组

通用方法 Array.isArray()。在之有一个网页(只有一个全局作用域)的情况下，可以使用 instanceof。

```javascript
if (Array.isArray(value){
    // doSomething
})

if (value instanceof Array){
    // doSomething
}
```

## API

### Array.prototype.keys()

返回数组索引的**迭代器**

```JavaScript
const arr_1 = ["Java", "Javascript", "Python", "C", "C++"];

const arr_1Key = Array.from(arr_1.keys());
console.log(arr_1Key); // [0,1,2,3,4]
```

### Array.prototype.values()

返回数组元素的**迭代器**

```JavaScript
const arr_1Values = Array.from(arr_1.values());

console.log(arr_1Values); // ["Java","Javascript","Python","C","C++"]
```

### Array.prototype.entries()

返回索引/值的**迭代器**

```javascript
const arr_1Entries = Array.from(arr_1.entries());

console.log(arr_1Entries); // [[0,"Java"], [1,"Javascript"], [2,"Python"], [3,"C"], [4,"C++"]]

for (const [i, e] of arr_1.entries()) {
  console.log(`i: ${i}    e: ${e}`);
}
// i: 0    e: Java
// i: 1    e: Javascript
// i: 2    e: Python
// i: 3    e: C
// i: 4    e: C++
```

### Array.prototype.fill()

{% label primary@fill() %} 方法用一个固定值填充一个数组中从起始索引到终止索引内的全部元素。不包括终止索引。返回修改后的数组

{% note info %}

arr.fill(value, [start=0], [end=array.length])

{% endnote %}

**value(*):**  用来填充数组元素的值

**[start=0\](number):**  起始索引，如果是负数，将从末尾开始计算

**[end=array.length\](number):**  终止索引(不包含)，如果是负数，将从末尾开始计算

```javascript
const zeroes = [0, 0, 0, 0, 0, 0];

zeroes.fill(8, 2, -2);
console.log(zeroes); // [0, 0, 8, 8, 0, 0]
zeroes.fill(0);

zeroes.fill(1, 4, 2); // 反向索引，忽略
console.log(zeroes); // [0, 0, 0, 0, 0, 0]
```

### Array.prototype.copyWithin()

{% label primary@copyWithin() %}会按照指定范围浅复制数组中的部分内容，然后将他们插入到指定索引开始的位置，并返回修改后的数组。

{% note info %}

arr.copyWithin(target, [start=0], [end=array.length]])

{% endnote %}

**target(number):** 复制序列到该位置，如果是负数，将从末尾开始计算。

如果 {% label primary@target %} 大于等于 {% label primary@arr.length %}，将会不发生拷贝。如果 {% label primary@target %} 在 {% label primary@start %} 之后，复制的序列将被修改以符合 {% label primary@arr.length %}

**[start=0\](number):** 复制起始索引，如果是负数，将从末尾开始计算

**[end=array.length\](number):** 终止索引(不包含)，如果是负数，将从末尾开始计算

```javascript
let inst,
  reset = () => (inst = [9, 8, 7, 6, 5]);
reset();

// 在源索引或目标索引到达数组边界时停止
inst.copyWithin(4); // 从inst中复制从0开始的内容插入到索引为4开始的位置
console.log(inst); // [9, 8, 7, 6, 9]
reset();

inst.copyWithin(0, 1); // 从inst 中复制从1开始的内容插入到从索引0开始的位置
console.log(inst); // [8, 7, 6, 5, 5]
reset();

inst.copyWithin(2, 0, -3); // 从inst中复制从0开始到索引1结束的内容到索引2开始的位置
console.log(inst); // [9, 8, 9, 8, 5]
reset();
```

### Array.prototype.join()

{% label primary@join() %} 方法将一个数组（或一个[类数组对象](https://developer.mozilla.org/zh-CN_docs/Web/JavaScript/Guide/Indexed_collections#working_with_array-like_objects)）的所有元素连接成一个字符串并返回这个字符串。如果数组只有一个项目，那么将返回该项目而不使用分隔符

{% note info %}

arr.join([separator])

{% endnote %}

```Javascript
let colors = ["red", "blue", "pink", "yellow"];
console.log(colors.join("^")); // red^blue^pink^yellow
```

### Array.prototype.push()

{% label primary@push() %}方法接收任意数量的参数，并将它们添加到数组末尾，返回数组的最新长度。

{% note info %}

arr.push(element1, ..., elementN)

{% endnote %}

```javascript
let colors = [];
let count = colors.push("pink", "yellow");
console.log(`colors: ${colors}      count: ${count}`); // colors: pink,yellow      count: 2
```

### Array.prototype.pop()

{% label primary@pop() %}方法用于删除数组的最后一项，同时减少数组的 length 值，返回被删除的项。

{% note info %}

arr.pop()

{% endnote %}

```Javascript
count = colors.pop();
console.log(`colors: ${colors}      count: ${count}`); // colors: pink      count: yellow
```

### Array.prototype.shift()

{% label primary@shift() %}方法删除数组的第一项并返回它，然后数组长度减一

{% note info %}

arr.shift()

{% endnote %}

```javascript
let colors = [];
let count = colors.push("pink", "black", "bule");
console.log(`colors: ${colors}      count: ${count}`); // colors: pink,black,bule      count: 3

count = colors.shift();
console.log(`colors: ${colors}      count: ${count}`); // colors: black,bule      count: pink
```

### Array.prototype.unshift()

{% label primary@unshift() %}在数组开头添加任意多个值，然后返回新的数组长度, 使用unshift()和pop()可以在相反方向上模拟队列

{% note info %}

arr.unshift(element1, ..., elementN)

{% endnote %}

```javascript
count = colors.unshift("yellow", "green");
console.log(`colors: ${colors}      count: ${count}`); // colors: yellow,green,black,bule      count: 4

count = colors.pop();
console.log(`colors: ${colors}      count: ${count}`); // colors: yellow,green,black      count: bule
```

### Array.prototype.reverse()

{% label primary@reverse() %} 方法将数组中元素的位置颠倒，并返回调用它的数组的引用

{% note info %}

 arr.reverse()

{% endnote %}

```javascript
let arr_1 = [7, 5, 6, 1, 4, 12];
console.log(arr_1.reverse()); // [12, 4, 1, 6, 5, 7]     仅是反向，不会比较
```

### Array.prototype.sort()

{% note info %}

arr.sort([compareFunction])

{% endnote %}

{% label primary@sort() %} 方法用[原地算法](https://en.wikipedia.org/wiki/In-place_algorithm)对数组的元素进行排序，并返回调用它的数组的引用。sort()会把数组转换为字符串再进行比较，在比较数值时不合适为此sort()可以接收一个比较方法比较函数接收两个参数，如果value_1 < value_2，返回负值，value_1 = value_2，返回0，value_1 > value_2, 返回正值。若需降序，交换正负值即可

```javascript
let arr_1 = [7, 5, 6, 1, 4, 12];
console.log(arr_1.sort()); // [1, 12, 4, 5, 6, 7]     默认转字符串后按升序排序

function compare(value_1, value_2) {
  switch (true) {
    case value_1 < value_2:
      return -1;
    case value_1 > value_2:
      return 1;
    default:
      return 0;
  }
}

console.log(arr_1.sort(compare)); // [1, 4, 5, 6, 7, 12]
```

### Array.prototype.concat()

{% label primary@concat() %} 方法可以在现有数组全部元素基础上创建一个新数组。

{% note info %}

var new_array = old_array.concat([value])

{% endnote %}

```javascript
let colors_1 = ["red", "pink", "blue"];
let newColors = ["green", "black"];
let colors_2 = colors_1.concat("yellow", newColors);

console.log(colors_1); // ["red", "pink", "blue"]
// 默认打平数组
console.log(colors_2); // ["red", "pink", "blue", "yellow", "green", "black"]
// 强制不打平数组
newColors[Symbol.isConcatSpreadable] = false;
let colors_3 = colors_1.concat("purple", newColors);
console.log(colors_3); // ["red", "pink", "blue", "purple", ["green","black"]]
```

### Array.prototype.slice()

{% label primary@slice() %}用于数组切片，接收一个或两个参数：切片的开始索引与结束索引。支持负索引，不包含结束索引元素

{% note info %}

arr.slice([start=0], [end=array.length])

{% endnote %}

**[start=0\](number):** 起始索引，如果是负数，将从末尾开始计算

**[end=array.length\](number):** 终止索引(不包含)，如果是负数，将从末尾开始计算

```javascript
let colors_1 = ["red", "pink", "black", "yellow", "green"];
let colors_2 = colors_1.slice(1, -1);
console.log(colors_2); // ["pink", "black", "yellow"]
```

### Array.prototype.splice()

{% label primary@splice() %}的主要目的是在数组中间插入元素，但有 3 种不同的方式使用这个方法。

-   <b>删除。</b> 需要给 splice() 传 2 个参数：要删除的第一个元素的位置和要删除的元素数量。可以从数组中删除任意多个元素，比如 splice(0,2)会删除前两个元素。
-   <b>插入。</b> 需要给 splice() 传 3 个参数： 开始位置，0(要删除的元素数量)和要插入的元素，可以在数组中删除指定的位置插入元素。第三个元素之后还可以传第四个、第五个参数，乃至任意多个要插入的元素。比如，splice(2,0,"pink","purple")会从数组位置 2 开始插入字符串"pink"和"purple"
-   <b>替换。</b> splice()在删除元素的同时可以在指定位置插入新元素，同样需要传入 3 个参数: 开始位置、要删除元素的数量和要插入的任意多个元素。要插入的元素数量不一定要跟删除的元素数量一致。比如，splice(2, 1, "red", "green")会在位置 2 删除一个元素，然后从该位置开始向数组中插入"red"和"green"

{% note info %}

array.splice(start[, deleteCount[, item1[, item2[, ...]]]])

{% endnote %}

**start[Number]:** 指定修改的开始位置（从0计数）。如果超出了数组的长度，则从数组末尾开始添加内容；如果是负值，则表示从数组末位开始的第几位（从-1计数，这意味着-n是倒数第n个元素并且等价于{% label primary@array.length-n %}）；如果负数的绝对值大于数组的长度，则表示开始位置为第0位。

**deleteCount[Number]:** 整数，表示要移除的数组元素的个数。

**item1, item2, ...:** 要添加进数组的元素,从{% label primary@start %} 位置开始。如果不指定，则 {% label primary@splice() %} 将只删除数组元素

返回由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组

```javascript
let colors = ["red", "pink", "yellow", "blue", "purple"];
// 删除
let result = colors.splice(0, 1);
console.log(`colors: ${colors}     result: ${result}`);
// colors: pink,yellow,blue,purple     result: red

// 插入
result = colors.splice(1, 0, "black", "greed");
console.log(`colors: ${colors}     result: ${result}`);
// colors: pink,black,greed,yellow,blue,purple     result:

// 替换
result = colors.splice(4, 1, "orange", "white");
console.log(`colors: ${colors}     result: ${result}`);
// colors: pink,black,greed,yellow,orange,white,purple     result: blue
```

### Array.prototype.indexOf()

{% label primary@indexOf() %}方法返回在数组中可以找到一个给定元素的第一个索引，如果不存在，则返回-1。

{% note info %}

arr.indexOf(value, [fromIndex=0])

{% endnote %}

**value (*):** 要查找的元素

**[fromIndex=0] (number):** 开始查找的位置。如果该索引值大于或等于数组长度，意味着不会在数组里查找，返回-1。如果参数中提供的索引值是一个负值，则将其作为数组末尾的一个抵消

```javascript
let num = [1, 2, 3, 4, 5, 4, 3, 2, 1];

console.log(num.indexOf(4)); // 3
console.log(num.indexOf(4, 4)); // 5

let person = { name: "Kinsiy" };
let people = [{ name: "Kinsiy" }];
let otherPeople = [person];

console.log(people.indexOf(person)); // -1
console.log(otherPeople.indexOf(person)); // 0
```

### Array.prototype.lastIndexOf()

{% label primary@lastIndexOf() %} 方法返回指定元素（也即有效的 JavaScript 值或变量）在数组中的最后一个的索引，如果不存在则返回 -1。从数组的后面向前查找，从 {% label primary@fromIndex %} 处开始。

{% note info %}

arr.lastIndexOf(value, [fromIndex=arr.length-1])

{% endnote %}

**value (*):** 要查找的元素

**[fromIndex=arr.length-1] (number):** 从此位置开始逆向查找. 如果为负值，将其视为从数组末尾向前的偏移

```Javascript
let num = [1, 2, 3, 4, 5, 4, 3, 2, 1];

console.log(num.lastIndexOf(4)); // 5
```

### Array.prototype.includes()

{% label primary@includes() %}返回布尔值，表示是否至少找到一个与指定元素匹配的项。在比较第一个参数跟数组每一项时，会使用全等(===)比较，也就是说两者必须严格相等。

{% note info %}

arr.includes(valueToFind, [fromIndex=0])

{% endnote %}

**valueToFind(*):** 需要查找的元素值

**[fromIndex=0\](Number):** 从fromIndex 索引处开始查找 valueToFind。如果为负值，则按升序从 array.length + fromIndex 的索引开始搜 （即使从末尾开始往前跳 fromIndex 的绝对值个索引，然后往后搜寻）

```javascript
let person = { name: "Kinsiy" };
let people = [{ name: "Kinsiy" }];
let otherPeople = [person];

console.log(people.indexOf(person)); // -1
console.log(otherPeople.indexOf(person)); // 0
console.log(people.includes(person)); // false
console.log(otherPeople.includes(person)); // true
```

### find() & findIndex()

ECMAScript 也允许按照定义的断言函数搜索数组，每个索引都会调用这个函数。断言函数的返回值决定了相应索引的元素是否被认为匹配。
断言函数接收 3 个参数：元素、索引和数组本身。其中元素是数组中当前搜索的元素，索引是当前元素的索引，而数组就是正在搜索的数组。断言函数返回真值，表示匹配。
find()和 findIndex()方法使用了断言函数。这两个方法都是从数组的最小索引开始，find()返回第一个匹配的元素，findIndex()返回第一个匹配元素的索引。这两个方法也可都可以接收第二个可选的参数用于指定断言函数内部 this 的值。

```javascript
const people = [
  {
    name: "Kinsiy",
    age: 18,
  },
  {
    name: "Restituo",
    age: 22,
  },
];

console.log(people.find((e, i, arr) => e.age > 20)); // {name: "Restituo", age: 22}
console.log(people.findIndex((e, i, arr) => e.age > 20)); // 1

// 找到匹配项后，这两个方法都不在继续搜索
const arr_1 = [2, 3, 4];
arr_1.find((e, i, arr) => {
  console.log(e);
  console.log(i);
  console.log(arr);
  return e === 3;
});
// 2
// 0
// [2, 3, 4]
// 3
// 1
// [2, 3, 4]
```

### every() & filter() & forEach() & map() & some()

ECMAScript 为数组定义了 5 个迭代方法。每个方法接收两个参数：每一项为参数运行的函数，以及可选的作为函数运行上下文的作用域对象。传个每个方法的函数接收 3 个参数：数组元素、元素索引和数组本身。

-   <b>every():</b> 对数组每一项都运行传入的函数，如果对每一项函数都返回 true，则这个方法返回 true。
-   <b>filter():</b> 对数组每一项都运行传入的函数，函数返回 true 的项会组成数组之后返回。
-   <b>forEach():</b> 对数组每一项都运行传入的函数，没有返回值
-   <b>map():</b> 对数组每一项都运行传入的函数，返回由每次函数调用的结果构成的数组
-   <b>some():</b> 对数组每一项都运行传入的函数，如果有一项函数返回 true，则这个方法返回 true。

这些方法都不改变调用他们的数组。

```javascript
let num = [1, 2, 3, 4, 5, 6, 7, 8, 9];
let result = num.every((e, i, a) => e > 5);
console.log(result); // false

result = num.some((e, i, a) => e > 5);
console.log(result); // true

result = num.filter((e, i, a) => e > 5);
console.log(result); // [6, 7, 8, 9]

result = num.map((e, i, a) => e ** 2);
console.log(result); // [1, 4, 9, 16, 25, 36, 49, 64, 81]

num.forEach((e, i, a) => {
  console.log(`i: ${i}    e: ${e}`);
});
// i: 0    e: 1
// ...
// i: 8    e: 9
```

### Array.prototype.reduce()

ECMAScript 为数组提供两个归并方法：reduce()和 reduceRight()。这两个方法都会迭代数组的所有项，并在此基础上构建一个最终返回值。reduce()从第一项归并到最后一项,reduceRight()反之。

这两个方法都接收两个参数：对每一项都会运行的归并函数，以及可选的以之为归并起点的初始值。传给 reduce()和 reduceRight()的函数接收四个参数：上一个归并值、当前值、当前值得索引、和数组本身。这个函数返回的任何值都会作为下一次调用同一个函数的第一个参数。如果没有给这两个函数传入可选的第二个参数(作为归并起点值)，则第一次迭代将从数组的第二项开始，因此传给归并函数的第一个参数是数组的第一项，第二个参数是数组的第二项。

```javascript
let values = [1, 2, 3, 4, 5, 7, 8];
let sum = values.reduce((prev, cur, index, array) => prev + cur);
console.log(sum); // 30

sum = values.reduceRight((prev, cur, index, array) => prev + cur, -3);
console.log(sum); // 27
```

{% note warning %}

第二个值不是索引值，并不是说从索引处开始归并，而是归并初始值。比如要求和，是在求和前先加这个初始值。

{% endnote %}
### Object.prototype.valueof()

{% label primary@valueof() %}返回数组本身。

{% note info %}

obj.valueOf()

{% endnote %}

```javascript
let colors = ["red", "blue", "pink", "yellow"];
console.log(colors.valueOf()); // ["red","blue","pink","yellow"]
```

### Object.prototype.toString()

{% label primary@toString() %}返回由数组中每个值得等效字符串拼接而成的一个逗号分隔的字符串。

{% note info %}

obj.toString()

{% endnote %}

```Javascript
let colors = ["red", "blue", "pink", "yellow"];
console.log(colors.toString()); // red,bule,pink,yellow
alert(colors); // red,bule,pink,yellow     alert()期待字符串，后台调用toString()

let person_1 = {
  toLocaleString() {
    return "Kinsiy";
  },

  toString() {
    return "Kinsiy";
  },
};

let person_2 = {
  toLocaleString() {
    return "Restituo";
  },

  toString() {
    return "Type57";
  },
};

let people = [person_1, person_2];
alert(people); // Kinsiy Type57
console.log(people.toString()); // Kinsiy Type57
console.log(people.toLocaleString()); // Kinsiy Restituo
```
## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)