---
title: 集合引用类型(Array)
author: Kinsiy
avatar: 'https://cdn.jsdelivr.net/gh/Kinsiy/cdn/img/custom/avatar.jpg'
authorLink: 'https://kinsiy.github.io'
authorAbout: I'm True
authorDesc: I'm True
categories: SKILL
comments: true
date: 2021-01-24 14:47:28
tags:
keywords:
description: Array 的使用
photos:
---
数组中的每个槽位可以存储任意类型的数据。ECMAScript 数组是动态大小的，会随着数据添加而自动增长。
# 创建数组
创建数组时可以给构造函数传入一个值，如果这个值是数值，会创建一个长度为指定数值的数组。如果这个值是其他类型的，会创建一个只包含该特定值的数组。
```javascript 
/* 使用Array构造函数 */
let colors_1 = new Array(20);  // 创建一个初始length为20的数组
let colors_2 = new Array("red","pink","blue");

/* 使用数组字面量 */
let colors_3 = ["green","yellow","black"];

/* 静态方法from()、of()
    from()用于将类数组结构转换为数组实例
    of()用于将一组参数转换为数组实例
*/

/* Array.from()的第一个参数是一个类数组对象，即任何可迭代的结构，或者有一个length属性和可索引元素的结构。 */
console.log(Array.from("Kinsiy"));                  // ["K","i","n","s","i","y"]

const m = new Map().set(1,3).set(2,4);
const s = new Set().add(5).add(6).add(8);
console.log(Array.from(m));                         // [[1,3], [2,4]]
console.log(Array.from(s));                         // [5, 6, 8]

// 浅复制
const arr_1 = [1,2,3,4];
const arr_2 = Array.from(arr_1);
console.log(arr_2);                                 // [1, 2, 3, 4]
console.log(arr_1 === arr_2);                       // false

// 转换arguments 对象
function getArgsArray(){
    return Array.from(arguments);
}

console.log(getArgsArray(8,9,4,5));                 // [8, 9, 4, 5]

// Array.from()可以接收第二个可选的映射函数参数。
// 还可以接收第三个可选参数, 用于指定映射函数中this的值。(在剪头函数中不适用)
const arr_3 = [1,2,3,4];
const arr_4 = Array.from(arr_3,(x)=>{return x ** 2});
const arr_5 = Array.from(arr_3,function(x) {return x ** this.exponent}, {exponent: 3});
console.log(arr_3);                                 // [1, 2, 3, 4]
console.log(arr_4);                                 // [1, 4, 9, 16]
console.log(arr_5);                                 // [1, 8, 27, 64]

/* Array.of() 可以把一组参数转换为数组 */
console.log(Array.of("K","i","n","s","i","f"));     // ["K","i","n","s","i","f"]
console.log(Array.of(undefined));                   // [undefined]
```
# 数组空位
使用数组字面量初始化数组时，可以使用一串逗号来创建空位(hole).ECMAScript 会将逗号之间相应的索引位置的值当成空位。
ES6新增方法普遍将这些空位当成存在的元素，只不过值是undefined
```javascript
const options = [5,,,,4];
for (const val of Array.from(options)){
    console.log(val === undefined);         // false true true true false 
}
```
由于行为不一致和存在性能隐患，因此实践中要避免使用数组空位。如果确实需要空位，则可以显式地用undefined值替代
# 数组索引
数组中最后一个元素的索引始终是length-1，因此下一个新增槽位的索引就是length。每次在数组最后一个元素后面新增一项，数组的length属性都会自动更新。注意length不是只读的，可以通过修改length的值删除数组元素
```javascript
let arr_1 = [5,7,9,3,15,88];
console.log(arr_1);             // [5,7,9,3,15,88]

arr_1.length = 5;
console.log(arr_1);             // [5,7,9,3,15]

arr_1[arr_1.length] = 99;
console.log(arr_1);             // [5,7,9,3,15,99]
```
# 检测数组
通用方法Array.isArray()。在之有一个网页(只有一个全局作用域)的情况下，可以使用instanceof。
```javascript
if (Array.isArray(value){
    // doSomething
})

if (value instanceof Array){
    // doSomething
}
```
# 迭代器方法
在ES6 中，Array 的原型上暴露了3个用于检索数组内容的方法：keys()、values()、和entries()。keys()返回数组索引的迭代器，value()返回数组元素的迭代器，而entries()返回索引/值得迭代器。
```javascript
const arr_1 = ["Java","Javascript","Python","C","C++"];

const arr_1Key = Array.from(arr_1.keys());
const arr_1Values = Array.from(arr_1.values());
const arr_1Entries = Array.from(arr_1.entries());
console.log(arr_1Key);                // [0,1,2,3,4]
console.log(arr_1Values);             // ["Java","Javascript","Python","C","C++"]
console.log(arr_1Entries);            // [[0,"Java"], [1,"Javascript"], [2,"Python"], [3,"C"], [4,"C++"]]

for (const [i,e] of arr_1.entries()){
    console.log(`i: ${i}    e: ${e}`);
}
// i: 0    e: Java
// i: 1    e: Javascript
// i: 2    e: Python
// i: 3    e: C
// i: 4    e: C++

```
# 复制和填充方法
填充数组方法fill(),批量复制方法copyWithin()。这两个方法的函数签名类似，都需要指定既有数组实例上的一个范围，包含开始索引，不包含结束结束索引。使用这个方法创建的数组不能缩放。
使用fill()方法可以向一个已有的数组中插入全部或部分相同的值。开始索引用于指定开始填充的位置，它是可选的。如果不提供结束索引，则一直填充到数组末尾。支持负索引，不支持反向索引。
```javascript
const zeroes = [0,0,0,0,0,0];

zeroes.fill(8,2,-2);
console.log(zeroes);    // [0, 0, 8, 8, 0, 0]
zeroes.fill(0);

zeroes.fill(1,4,2);     // 反向索引，忽略
console.log(zeroes);    // [0, 0, 0, 0, 0, 0]
```
copyWithin()会按照指定范围浅复制数组中的部分内容，然后将他们插入到指定索引开始的位置。开始索引和结束索引与fill()使用相同的计算方法。支持负索引，不支持反向索引。
```javascript
let inst, reset = () => inst = [9, 8, 7, 6, 5];
reset();

// 在源索引或目标索引到达数组边界时停止
inst.copyWithin(4);         // 从inst中复制从0开始的内容插入到索引为4开始的位置
console.log(inst);          // [9, 8, 7, 6, 9]
reset();

inst.copyWithin(0,1);       // 从inst 中复制从1开始的内容插入到从索引0开始的位置
console.log(inst);          // [8, 7, 6, 5, 5]
reset();

inst.copyWithin(2,0,-3);     // 从inst中复制从0开始到索引1结束的内容到索引2开始的位置
console.log(inst);           // [9, 8, 9, 8, 5]
reset();
```
# 转换方法
valueof()返回的是数组本身。而toString()返回由数组中每个值得等效字符串拼接而成的一个逗号分隔的字符串。
```javascript
let colors = ["red","blue","pink","yellow"];
console.log(colors.toString());         // red,bule,pink,yellow
console.log(colors.valueOf());          // ["red","blue","pink","yellow"]
alert(colors);                          // red,bule,pink,yellow     alert()期待字符串，后台调用toString()

let person_1 = {
    toLocaleString(){
        return "Kinsiy";
    },

    toString(){
        return "Kinsiy";
    }
}

let person_2 = {
    toLocaleString(){
        return "Restituo";
    },

    toString(){
        return "Type57"
    }
}

let people = [person_1, person_2];
alert(people);                          // Kinsiy Type57
console.log(people.toString());         // Kinsiy Type57
console.log(people.toLocaleString());   // Kinsiy Restituo

/* join()
    join()方法接收一个参数，即字符串分隔符，返回包含所有项的字符串    
 */
console.log(colors.join("^"));          // red^blue^pink^yellow
```
# 栈方法
ECMAScript 数组提供push()和pop()方法，以实现类似栈的方法。
push()方法接收任意数量的参数，并将它们添加到数组末尾，返回数组的最新长度。pop()方法则用于删除数组的最后一项，同时减少数组的length值，返回被删除的项。
```javascript
let colors = [];
let count = colors.push("pink","yellow");
console.log(`colors: ${colors}      count: ${count}`);      // colors: pink,yellow      count: 2

count = colors.pop();
console.log(`colors: ${colors}      count: ${count}`);      // colors: pink      count: yellow
```
# 队列方法
shift()方法删除数组的第一项并返回它，然后数组长度减一。使用shift()和push(),可以把数组当成队列来使用。
```javascript
let colors = [];
let count = colors.push("pink","black","bule");
console.log(`colors: ${colors}      count: ${count}`);      // colors: pink,black,bule      count: 3

count = colors.shift();
console.log(`colors: ${colors}      count: ${count}`);      // colors: black,bule      count: pink

/* unshift()
    在数组开头添加任意多个值，然后返回新的数组长度。使用unshift()和pop()可以在相反方向上模拟队列
 */
count = colors.unshift("yellow","green");
console.log(`colors: ${colors}      count: ${count}`);      // colors: yellow,green,black,bule      count: 4

count = colors.pop();
console.log(`colors: ${colors}      count: ${count}`);      // colors: yellow,green,black      count: bule
```
# 排序方法
数组有两个方法可以用来对元素重新排序：reverse()和sort();
这两个方法都返回调用他了门的数组的引用。
```javascript
let arr_1 = [7, 5, 6, 1, 4, 12];
console.log(arr_1.reverse());       // [12, 4, 1, 6, 5, 7]     仅是反向，不会比较
console.log(arr_1.sort());          // [1, 12, 4, 5, 6, 7]     默认转字符串后按升序排序

/* sort()会把数组转换为字符串再进行比较，在比较数值时不合适
    为此sort()可以接收一个比较方法
    比较函数接收两个参数，如果value_1 < value_2，返回负值，value_1 = value_2，返回0，value_1 > value_2, 返回正值。
    若需降序，交换正负值即可
 */
function compare(value_1,value_2){
    switch(true){
        case(value_1 < value_2):
            return -1;
        case(value_1 > value_2):
            return 1;
        default:
            return 0
    }
}

console.log(arr_1.sort(compare));       // [1, 4, 5, 6, 7, 12]
```
# 操作方法
## concat()
concat() 方法可以在现有数组全部元素基础上创建一个新数组。
```javascript
let colors_1 = ["red","pink","blue"];
let newColors = ["green","black"]
let colors_2 = colors_1.concat("yellow",newColors);

console.log(colors_1);                              // ["red", "pink", "blue"]
// 默认打平数组
console.log(colors_2);                              // ["red", "pink", "blue", "yellow", "green", "black"]
// 强制不打平数组
newColors[Symbol.isConcatSpreadable] = false;
let colors_3 = colors_1.concat("purple",newColors); 
console.log(colors_3);                              // ["red", "pink", "blue", "purple", ["green","black"]]
```
## slice()
slice()用于数组切片，接收一个或两个参数：切片的开始索引与结束索引。支持负索引，不包含结束索引元素
```javascript
let colors_1 = ["red", "pink", "black", "yellow", "green"];
let colors_2 = colors_1.slice(1,-1);
console.log(colors_2);          // ["pink", "black", "yellow"]
```
## splice()
splice()的主要目的是在数组中间插入元素，但有3种不同的方式使用这个方法。
* <b>删除。</b> 需要给splice() 传2个参数：要删除的第一个元素的位置和要删除的元素数量。可以从数组中删除任意多个元素，比如splice(0,2)会删除前两个元素。
* <b>插入。</b> 需要给splice() 传3个参数： 开始位置，0(要删除的元素数量)和要插入的元素，可以在数组中删除指定的位置插入元素。第三个元素之后还可以传第四个、第五个参数，乃至任意多个要插入的元素。比如，splice(2,0,"pink","purple")会从数组位置2开始插入字符串"pink"和"purple"
* <b>替换。</b> splice()在删除元素的同时可以在指定位置插入新元素，同样哟啊传入3个参数: 开始位置、要删除元素的数量和要插入的任意多个元素。要插入的元素数量不一定要跟删除的元素数量一致。比如，splice(2, 1, "red", "green")会在位置2删除一个元素，然后从该位置开始向数组中插入"red"和"green"

```javascript
let colors = ["red", "pink", "yellow", "blue", "purple"];
// 删除
let result = colors.splice(0,1)
console.log(`colors: ${colors}     result: ${result}`);
// colors: pink,yellow,blue,purple     result: red

// 插入
result = colors.splice(1,0,"black","greed");
console.log(`colors: ${colors}     result: ${result}`);
// colors: pink,black,greed,yellow,blue,purple     result:

// 替换
result = colors.splice(4,1,"orange","white");
console.log(`colors: ${colors}     result: ${result}`);
// colors: pink,black,greed,yellow,orange,white,purple     result: blue
```
# 搜索和位置方法
ECMAScript 提供两类搜索数组的方法：按严格相等搜索和按断言函数搜索
## 按严格相等
indexOf(),lastIndexOf()和includes()。前两个方法在所有版本中都可用，而第三个方法是ECMAScript 7 新增的。这些方法都接收两个参数: 要查找的元素和一个可选的起始搜索位置。indexOf()和includes()方法从数组前头(第一项)向后搜索，而lastIndexOf()从数组末尾(最后一项)开始向前搜索。
indexOf()和lastIndexOf()都返回要查找的元素在数组中的位置，如果没有找到则返回-1。includes()返回布尔值，表示是否至少找到一个与指定元素匹配的项。在比较第一个参数跟数组每一项时，会使用全等(===)比较，也就是说两者必须严格相等。
```javascript
let num = [1, 2, 3, 4, 5, 4, 3, 2, 1];

console.log(num.indexOf(4));            // 3
console.log(num.indexOf(4,4));          // 5
console.log(num.lastIndexOf(4));        // 5

let person = {name: "Kinsiy"};
let people = [{name: "Kinsiy"}];
let otherPeople = [person];

console.log(people.indexOf(person));            // -1
console.log(otherPeople.indexOf(person));       // 0
console.log(people.includes(person));           // false
console.log(otherPeople.includes(person));      // true
```
## 断言函数
ECMAScript 也允许按照定义的断言函数搜索数组，每个索引都会调用这个函数。断言函数的返回值决定了相应索引的元素是否被认为匹配。
断言函数接收3个参数：元素、索引和数组本身。其中元素是数组中当前搜索的元素，索引是当前元素的索引，而数组就是正在搜索的数组。断言函数返回真值，表示匹配。
find()和findIndex()方法使用了断言函数。这两个方法都是从数组的最小索引开始，find()返回第一个匹配的元素，findIndex()返回第一个匹配元素的索引。这两个方法也可都可以接收第二个可选的参数用于指定断言函数内部this的值。
```javascript
const people = [{
    name: "Kinsiy",
    age: 18
},{
    name: "Restituo",
    age: 22
}];

console.log(people.find((e,i,arr) => e.age > 20));         // {name: "Restituo", age: 22} 
console.log(people.findIndex((e,i,arr) => e.age > 20));    // 1

// 找到匹配项后，这两个方法都不在继续搜索
const arr_1 = [2, 3, 4];
arr_1.find((e,i,arr)=> {
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
# 迭代方法
ECMAScript为数组定义了5个迭代方法。每个方法接收两个参数：每一项为参数运行的函数，以及可选的作为函数运行上下文的作用域对象。传个每个方法的函数接收3个参数：数组元素、元素索引和数组本身。
* <b>every():</b> 对数组每一项都运行传入的函数，如果对每一项函数都返回true，则这个方法返回true。
* <b>filter():</b> 对数组每一项都运行传入的函数，函数返回true的项会组成数组之后返回。
* <b>forEach():</b> 对数组每一项都运行传入的函数，没有返回值
* <b>map():</b> 对数组每一项都运行传入的函数，返回由每次函数调用的结果构成的数组
* <b>some():</b> 对数组每一项都运行传入的函数，如果有一项函数返回true，则这个方法返回true。

这些方法都不改变调用他们的数组。
```javascript
let num = [1, 2, 3, 4, 5, 6, 7, 8, 9];
let result = num.every((e,i,a) => e > 5);
console.log(result);        // false

result = num.some((e,i,a) => e > 5);
console.log(result);        // true

result = num.filter((e,i,a) => e > 5);
console.log(result);        // [6, 7, 8, 9]

result = num.map((e,i,a) => e ** 2);
console.log(result);        // [1, 4, 9, 16, 25, 36, 49, 64, 81]

num.forEach((e,i,a) => {
    console.log(`i: ${i}    e: ${e}`);
})
// i: 0    e: 1
// ...
// i: 8    e: 9
```
# 归并方法
ECMAScript 为数组提供两个归并方法：reduce()和reduceRight()。这两个方法都会迭代数组的所有项，并在此基础上构建一个最终返回值。reduce()从第一项归并到最后一项,reduceRight()反之。
这两个方法都接收两个参数：对每一项都会运行的归并函数，以及可选的以之为归并起点的初始值。传给reduce()和reduceRight()的函数接收四个参数：上一个归并值、当前值、当前值得索引、和数组本身。这个函数返回的任何值都会作为下一次调用同一个函数的第一个参数。如果没有给这两个函数传入可选的第二个参数(作为归并起点值)，则第一次迭代将从数组的第二项开始，因此传给归并函数的第一个参数是数组的第一项，第二个参数是数组的第二项。<
```javascript
let values = [1, 2, 3, 4, 5, 7, 8];
let sum = values.reduce((prev, cur, index, array) => prev + cur);
console.log(sum);       // 30


sum = values.reduceRight((prev, cur, index, array) => prev + cur,-3);
console.log(sum)        // 27
```
<b>注意：</b>第二个值不是索引值，并不是说从索引处开始归并，而是归并初始值。比如要求和，是在求和前先加这个初始值。