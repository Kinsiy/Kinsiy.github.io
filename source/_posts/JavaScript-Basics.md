---
title: JavaScript-语言基础
date: 2020-12-20 12:01:49
description:
categories: [学习笔记, Javascript]
tags: 
---

本文依据红宝书内容概括了 Javascript 的语言基础，分语法、关键字与保留字、变量、数据类型、操作符、语句、函数六部分。为个人读书笔记，可能会有错误的地方。

<!-- more -->

## 语法

### 区分大小写

ECMAScript 中一切都区分大小写。无论是变量、函数名、还是操作符，都区分大小写。

### 标识符

-   第一个字符必须是字母、下划线(\_)或美元符号($)
-   剩下的其他字符可以是字母、下划线、美元符号或数字

字母可以是扩展 ASCII 中的字母，也可以是 Unicode 的字母字符。
按照惯例，ECMAScript 标识符使用驼峰大小写形式，即第一个单词的首字母小写，后面每个单词的首字母大写。

### 注释

```javascript
// 驼峰大小写(单行注释)
/*· 多行注释
    firstSecond
    myCar
    doSomethingImportant
*/
```

### 严格模式

严格模式(ECMAScript 5 增加)是一种不同的 Javascript 解析和执行模式，ECMAScript 3 的一些不规范写法在这种模式下会被处理，对于不安全的活动会抛出错误。

```javascript
"use strict"; //对整个脚本启用严格模式

function doSomething() {
  "use strict"; // 在指定函数中启用严格模式
}
```

### 语句

ECMAScript 中的语句以分号结尾。多条语句可以合并到一个 C 语言风格的代码块中。代码块由一个左花括号( { )标识开始，一个右花括号( } )标识结束。

```javascript
let sum = a + b; //没有分号也有效，但不推荐
let sum = a - b; //加分号有效，推荐
/* 代码块 */
if (test) {
  test = false;
  console.log(test);
}
/* if 之类的控制语句只在执行多条语句时要求必须有代码块。不过，最佳实践是始终在控制语句中使用代码块，即使要执行的只有一条语句 */
```

### 关键字与保留字

ECMA-262 第六版规定的所有关键字如下：

|    -     |    -     |   -   |    -    |    -     |    -    |     -      |   -    |
| :------: | :------: | :---: | :-----: | :------: | :-----: | :--------: | :----: |
|  break   |    do    |  in   | typeof  |   case   |  else   | instanceof |  var   |
|  catch   |  export  |  new  |  void   |  class   | extends |   return   | while  |
|  const   | finally  | super |  with   | continue |   for   |   switch   | yield  |
| debugger | function | this  | default |    if    |  throw  |   delete   | import |
|   try    |    -     |   -   |    -    |    -     |    -    |     -      |   -    |

#### 未来的保留字

ECMA-262 第 6 版为将来保留的所有词汇：<br>
始终保留：<b>enum</b>

严格模式下保留：

| implements | package | public | interface | protected | static | let | private |
| :--------: | :-----: | :----: | :-------: | :-------: | :----: | :-: | :-----: |

代码块中保留：<b>await</b>

### 变量

有三个关键字可以声明变量：var、const 和 let。const 和 let 只能在 ECMAScript 6 及更晚的版本中使用。

#### var

```javascript
var message = "hello worid!";
message = "hi"; // 合法，但不推荐

/* 使用var操作符定义的变量会成为包含它的函数的局部变量
    在使用var声明变量时，变量会自动添加到最接近的上下文中(参见《变量、作用域与内存》，不好意思还没写！) */

function doSomething() {
  var firstName = "Kinsiy"; //局部变量
  secondName = "Kinsiy"; //全局变量。不推荐，难维护

  console.log(sex); //不会报错
  var sex = "boy"; //使用var这个关键字声明的变量会自动提升到函数作用域的顶部
  var sex = "girl"; //反复多次使用var声明一个变量也没有问题
}
console.log(firstName); //ReferenceError
console.log(secondName); //Kinsiy
```

#### let

let 和 var 的作用差不多,但有着非常重要的区别。最明显的区别是，let 声明的范围是块作用域，而 var 声明的范围是函数作用域。

```javascript
//全局声明
/* 使用let在全局作用域中声明的变量不会成为window对象的属性(var声明则会) */
var car = "civic";
let house = "Oh! No";
console.log(window.car); //civic
console.log(window.house); //undefined

if (true) {
  console.log(myName); //undefined     myName变量提升
  console.log(anotherName); //ReferenceError    anotherName不会被提升   暂时性死区

  var myName = "Kinsiy";
  let anotherName = "Restituo";
  let anotherName = "QING"; //SyntaxError; 标识符anotherName已经声明过了

  console.log(myName); //Kinsiy
  console.log(anotherName); //Restituo
}

console.log(myName); //Kinsiy
console.log(anotherName); //ReferenceError     作用域仅作用与块内部

// 条件声明
/* let 不能依赖条件声明模式，即使使用try/catch或typeof也不能解决 */
if (typeof Testname === "undefined") {
  let Testname;
}
/* Testname 被限制在 if { } 块的作用域中
    这个声明形同下面这个全局声明 */
Testname;

try {
  console.log(sex);
} catch (error) {
  let sex = "boy";
}
/* sex 被限制在 catch { } 块的作用域中
    这个声明形同下面这个全局声明 */
sex;
/* 总之一句话不能使用let进行条件式声明 */
```

#### const

const 的行为与 let 基本相同，唯一一个重要的区别是用它声明变量时必须同时初始化变量尝试修改 const 声明的变量会导致运行时错误。

```javascript
const age = 22;
age = 17; //TypeError
//const 也不允许重复声明
const name = "Restituo";
const name = "Kinsiy"; //SyntaxError
//const 的作用域也是块
const sex = "girl";
if (true) {
  const sex = "boy";
}
console.log(sex); //girl
/* const 声明的限制只适用与它指向的变量的引用 */
const person = {};
person.name = "Kinsiy"; //没有问题

// 特别得
/* 对于for、for in 、 for of 循环 
    for in 和for of它们两个都是一种严格的迭代语句，对于对象中的每一个属性值，有一个指定的语句块被执行。
    也就是每一次循环，都会产生一个块级作用域来完成每个变量的行为
    然而for循环并不会遍历对象的属性，每一次循环都是在同个块级作用域中进行，使用const声明就会报错。
    所以在for in或者for of当中，推荐使用const来确保访问到的属性值不会被后续语句所改变。
*/
```

### 数据类型

#### Undefined

Undefined 类型只有一个值，就是特殊值 undefined(假值)。当使用 var 或 let 声明了变量但没有初始化值时，就相当于给变量赋予了 undefined 值：

```javascript
let message;
console.log(message == undefinded); //true

console.log(message); //"undefined"
console.log(age); //ReferenceError，声明了变量但未赋值与未声明变量是不同的

//typeof 时变量声明与否均返回undefined
console.log(typeof message); //"undefined"
console.log(typeof age); //"undefined"
```

#### Null

Null 类型同样只有一个值，即特殊值 null(假值)。逻辑上讲，null 值表示一个空对象指针。

```javascript
let car = null;
console.log(typeof car); //"object"

// undefined 值是有null值派生而来的，因此ECMA-262将他们定义为表面上相等
console.log(null == undefined); //true

/* 任何时候，只要变量要保存对象，而当时又没有那个对象可保存,就要用null来填充该变量。这样可以保持null是空对象指针的语义，并进一步与undefined区分开来 */
```

#### Boolean 类型

Boolean 有两个字面值：true 和 false。这两个布尔值不同于数值，因此 true 不等于 1，false 不等于 0。<br>
Boolean()转型函数可以在任意类型的数据上调用，而且始终返回一个布尔值。转换规则如下：

| 数据类型  |   转换为 true 的值   | 转化为 false 的值 |
| :-------: | :------------------: | :---------------: |
|  Boolean  |         true         |       false       |
|  String   |      非空字符串      |   ""(空字符串)    |
|  Number   | 非零数值(包括无穷值) |      0、NaN       |
|  Object   |       任意对象       |       null        |
| Undefined |     N/A(不存在)      |     undefined     |

#### Number 类型

##### 浮点值

要定义浮点值，数值中必须包含小数点，且小数点后面必须至少有一个数字。

```javascript
// 科学记数法
// 格式要求是一个数值(整数或浮点数)后跟一个大写或小写的e，再加上一个要乘的10的多少次幂
let floatNum = 3.14e7; //等于31400000

// 值的范围
/*如果某个计算得到的数值结果超过了Javascript可以表示的范围，那么这个数值会被自动转换为一个特殊的Infinity(无穷值)。任何无法表示的负数以-Infinity表示，任何无法表示的正数以Infinity表示。*/

let result = Number.MAX_VALUE + Number.MAX_VALUE;
console.log(isFinite(result)); //false, isFinite()确定一个值是不是有限大

//NaN
/* 用于表示本来要反回数值的操作失败了(不是错误) ，任何涉及NaN的操作始终返回NaN */

console.log(0 / 0); //NaN
console.log(-0 / +0); //NaN
console.log(NaN / 10); //NaN
console.log(5 / 0); //Infinity
console.log(5 / -0); //-Infinity

// 数值转换
/* 有3个函数可以将非数值转换为数值：Number()、parseInt()、parseFloat(); Number()为转型函数,可用于任何数据类型。后两个主要用于将字符串转换为数值 */
let num1 = Number("Hello World!"); //NaN
let num2 = Number(""); //0
let num3 = Number("000011"); //11
let num4 = Number("true"); //1

let num5 = parseInt("Hello World!"); //NaN
let num6 = parseInt("1234hi"); //1234
let num7 = parseInt("0xA"); //10
let num8 = parseInt("AF", 16); //175

// parseFloat() 只解析十进制值，不能指定底数
let num10 = parseFloat("1234hi"); //1234       按整数解析
let num11 = parseFloat("22.4.5"); //22.4        指解析到第一个小数点
let num12 = parseFloat("0xA"); //0
```

Number()函数基于如下规则执行转换：

-   布尔值，true 转换为 1，false 转换为 0。
-   数值，直接返回
-   null，返回 0
-   undefined，返回 NaN
-   字符串应用一下规则
    -   如果字符串包含数值字符，包括数值字符前面带加、减号的情况，则转换为一个十进制数值。
    -   如果字符串包含有效的浮点值格式如"1.1",则会转换为相应的浮点值
    -   如果字符串包含有效的十六进制格式如"0xf",则会转换为与该十六进制对应的十进制数值
    -   如果是空字符串，则返回 0
    -   如果字符串包含除上述情况之外的其他字符，则返回 NaN
-   对象，调用 valueof()方法，并按照上述规则转换返回的值。如果转换结果是 NaN，则调用 toString()方法,在按照字符串的规则转换。

#### String 类型

String (字符串)数据类型表示零或多个 16 位 Unicode 字符序列。字符串可以使用双引号(")、单引号(')或反引号(`)标识。

| 字面量 |                                              含义                                              |
| :----: | :--------------------------------------------------------------------------------------------: |
|   \n   |                                              换行                                              |
|   \t   |                                              制表                                              |
|   \b   |                                              退格                                              |
|   \r   |                                              回车                                              |
|   \f   |                                              换页                                              |
|  \\\\  |                                           反斜杠(\\)                                           |
|  \\'   |                 单引号，在字符串以单引号标示时使用，例如'He said, \\'hey.\\''                  |
|  \\"   |                 双引号，在字符串以双引号标示时使用，例如"He said, \\"hey.\\""                  |
|  \\\`  |               反引号，在字符串以反引号标示时使用，例如\`He said, \\\`hey.\\\`\`                |
|  \xnn  |           以十六进制编码 nn 标识的字符(其中 n 是十六进制数字 0~F)，例如\x41 等于"A"            |
| \unnn  | 以十六进制编码 nnnn 标识的 Unicode 字符(其中 n 是十六进制数字 0~F)，例如\u03a3 等于希腊字符"Σ" |

ECMAScript 中的字符串是不可变的，意思是一旦创建，它们的值就不能变了。要修改某个变量中的字符串值，必须先销毁原始字符串，然后将包含新值得另一个字符串保存到该变量。

```javascript
// toString()
/* toString()方法可见于数值、布尔值、对象、和字符串值。null和undefined值没有toString()方法 */
let num = 10;
console.log(num.toString()); //"10"
console.log(num.toString(2)); //"1010"
console.log(num.toString(8)); //"12"
console.log(num.toString(10)); //"10"
console.log(num.toString(16)); //"a"
/* 不确定一个值是不是null或undefined，可以用String()转型函数 */

// 模板字面量
/* 模板字面量可以保留换行字符，可以跨行定义字符串 */
let pageHtml = `
<div>
    <a herf="#">
        <span>Kinsiy</span>
    </a>
</div>`;
console.log(pageHtml); //输出保留制表符(空格)

let value = 5;
let exponent = "五倍";
let resultStr = `${value}的${exponent}是${value * 5}`;
console.log(resultStr); //5的5倍是25

//原始字符串
/* 使用模板字面量也可以直接获取原始的模板字面量内容(如换行符或Unicode字符)，而不是被转义后的字符表示。 */
console.log(`\u00A9`); //  ©
console.log(String.raw`\u00A9`); // \u00A9
```

String()函数遵循如下规则：

-   如果有 toString()方法，则调用该方法(不传参数)并返回结果
-   如果值是 null，返回"null"
-   如果值是 undefined，返回"undefined"

#### Symbol 类型

Symbol(符号)是 EMCAScript 6 新增的数据类型。符号是原始值，且符号是唯一、不可变的。符号的用途是确保对象属性使用唯一标识符，不会发生属性冲突的危险。
<br>看不明白，暂时略过。

#### Object 类型

ECMAScript 中的对象其实就是一组数据和功能的集合。
<br>每个 Object 实例都有如下属性和方法：

-   constructor: 用于创建当前对象的函数
-   hasOwnProperty(propertyName): 用于判断当前对象实例(不是原型)上是否存在给定的属性。要检查的属性名必须是字符串。
-   propertyIsEnumerable(propertyName): 用于判断给定的属性是否可以使用 for-in 语句枚举
-   toLocalString(): 返回对象的字符串表示, 该字符串反应对象所在的本地唤环境
-   toString(): 返回对象的字符串表示
-   valueof(): 返会对象对应的字符串、数值、布尔值表示。通常与 toString()的返回值相同。

### 操作符

#### 一元操作符

| "+" | "++" | "-" | "--" |
| :-: | :--: | :-: | :--: |

#### 位操作符

|    名称    |  符号  |                  规则(效果)                   |
| :--------: | :----: | :-------------------------------------------: |
|   按位非   |   ~    | 返回数值的一补数(最终效果：对数值取反并减 1)  |
|   按位与   |   &    |              两位均为 1，返回 1               |
|   按位或   | &#124; |            在至少一位是 1 是返回 1            |
|  按位异或  |   ^    | 只在一位是 1 是返回 1，两位都是 1 或 0 返回 0 |
|    左移    |   <<   |     按照指定的位数将数值的所有位向左移动      |
| 有符号右移 |   >>   |   将数值的所有 32 位都向右移，同时保留符号    |
| 无符号右移 |  >>>   |          将数值的所有 32 位都向右移           |

#### 布尔操作符

##### 逻辑非(!)

逻辑非会遵循如下规则：

-   如果操作数是对象，则返回 false
-   如果操作数是空字符串，返回 true
-   如果操作数是非空字符串，返回 false
-   如果操作数是数值 0，返回 true
-   如果操作数是非 0 数值(包括 Infinity)，返回 false
-   如果操作数是 null，返回 true
-   如果操作数是 NaN，返回 true
-   如果操作数数 undefined，返回 true

##### 逻辑与(&&)

逻辑与操作符可用于任何类型的操作数，不限与布尔值。如果有操作数不是布尔值，则逻辑与并不一定会返回布尔值，而是遵循如下规则。

-   如果第一个操作数是对象，则返回第二个操作数
-   如果第二个操作数是对象，则只有第一个操作数求值为 true 才会返回该对象
-   如果两个操作数均是对象，则返回第二个操作数
-   如果有一个操作数是 null，则返回 null
-   如果有一个操作数是 NaN，则返回 NaN
-   如果有一个操作数是 undefined，则返回 undefined

##### 逻辑或(||)

与逻辑与类似，如果有一个操作数不是布尔值，那么逻辑或操作符也不一定返回布尔值

-   如果第一个操作数是对象，则返回第一个操作数
-   如果第一个操作数求值为 false，则返回第二个操作数
-   如果两个操作数均是对象，则返回第一个操作数
-   如果两个操作数都是 null，则返回 null
-   如果两个操作数都是 NaN，则返回 NaN
-   如果两个操作数都是 undefined，则返回 undefined

#### 乘性操作符

##### 乘法(\*)

-   有任一操作数是 NaN，返回 NaN
-   Infinity \* 0 ，返回 NaN
-   Infinity \* x，依据 x 的正负返回 Infinity 或-Infinity
-   Infinity \* Infinity，返回 Infinity
-   如果不是数值的操作数，则现在后台用 Number()将其转换为数值，再应用上述规则

##### 除法(/)

-   有任一操作数是 NaN，返回 NaN
-   Infinity / Infinity ，返回 NaN
-   x / 0 (x≠0)，依据 x 的正负返回 Infinity 或-Infinity
-   0 / 0，返回 NaN
-   Infinity / x，依据 x 的正负返回 Infinity 或-Infinity
-   如果不是数值的操作数，则现在后台用 Number()将其转换为数值，再应用上述规则

##### 取模(%)

与其他乘性操作符一样，取模操作符对特殊值也有一些特殊的行为

-   如果操作数是数值，则执行常规除法运算，返回余数
-   如果被除数是无限值，除数是有限值，返回 NaN
-   如果被除数是有限值，除数是 0，则返回 NaN
-   Infinity % Infinity，返回 NaN
-   被除数是有限值，除数是无限值，返回被除数
-   如果不是数值的操作数，则现在后台用 Number()将其转换为数值，再应用上述规则

#### 指数操作符

```javascript
console.log(3 ** 2); //9
console.log(16 ** 0.5); //4
```

#### 加性操作符

##### 加法操作符

-   如果有任一操作数是 NaN，返回 NaN
-   Infinity + Infinity,返回 Infinity
-   -Infinity + -Infinity,返回-Infinity
-   -Infinity + Infinity,返回 NaN
-   +0 + +0，返回+0
-   -0 + +0，返回+0
-   -0 + -0，返回-0
-   若有一个操作数是字符串
    -   两个操作数均为字符串，将第二个字符串拼接到第一个字符串后面
    -   如果只有一个操作数是字符串，则将另一个操作数转换为字符串，再将两个字符串拼接到一起

##### 减法操作符

-   如果有任一操作数是 NaN，返回 NaN
-   Infinity - Infinity,返回 NaN
-   Infinity - -Infinity,返回 Infinity
-   -Infinity - Infinity,返回-Infinity
-   +0 - +0，返回+0
-   +0 - -0，返回-0
-   -0 - -0，返回+0
-   若有一个操作数是字符串、布尔值、null 或 undefined，则先在后台使用 Number()将其转换为数值，然后在根据前面的规则执行数学运算。
-   如果有任一操作数是对象，则调用其 valueof()方法取得表示他的数值。如果对象没有 valueof()方法，则调用其 toString()方法，然后再将得到的字符串转换为数值。

#### 关系操作符

-   如果操作符都是数值，则执行数值比较
-   如果操作数是字符串，则逐个比较字符串中对应字符的编码
-   如果有任一操作数是数值，则将另一个操作数转换为数值，执行数值比较
-   如果有任一操作数是对象，则调用其 valueof()方法取得表示他的数值。如果对象没有 valueof()方法，则调用其 toString()方法，然后再将得到的字符串转换为数值。
-   如果有任一操作数是布尔值，则将其转换为数值在执行比较
-   只要任一操作数为 NaN，均返会 false

#### 相等操作符

##### 等于(==)与不等于(!=)

-   null 与 undefined 不能转换为其他类型的值再进行比较

| 表达式            | 结果  |
| :---------------- | :---- |
| null == undefined | true  |
| "NaN" == NaN      | false |
| 5 == NaN          | false |
| NaN == NaN        | false |
| NaN != NaN        | true  |
| false == 0        | true  |
| true == 1         | true  |
| true == 2         | false |
| undefined == 0    | false |
| null == 0         | false |
| "5" == 5          | true  |

##### 全等(===)和不全等(!==)

```javascript
let result_1 = "55" == 55; //true, 转换后相等
let result_2 = "55" === 55; //false, 因为数据类型不同
```

#### 条件操作符

```javascript
// variable = boolean_expression ? true_value : false_value;
let max = num1 > num2 ? num1 : num2;
```

#### 赋值操作符

<b>"="</b>

#### 逗号操作符

```javascript
let num1 = 1,
  num2 = 2,
  num3 = 3;
let num = (5, 1, 4, 8, 0); //num = 0
```

### 语句

#### if

> if (condition) statement1 else statement2

这里的条件(condition)可以是任何表达式，并且求值结果不一定是布尔值。ECMAScript 会自动调用 Boolean()函数将这个表达式转换为布尔值。

#### do-while

> do { statement } while (expression)

```javascript
let i = 0;
do{
    i += 2;
}while(i<10>);
```

#### while

> while (expression) statement

```javascript
let i = 0;
while(i<10>){
    i += 2;
};
```

#### for

> for (initialization; expression; post-loop-expression) statement

```javascript
let count = 10;
for (let i = 0; i < count; i++>){
    console.log(i)
}
```

#### for-in

for-in 语句是一种严格的迭代语句，<b>用于枚举对象中的非符号键属性</b>

> for (property in expression) statement

```javascript
for (const proName in window) {
  console.log(proName);
}
```

ECMAScript 中对象的属性是无序的，因此 for-in 语句不能保证返回对象属性的顺序

#### for-of

for-of 语句是一种严格的迭代语句，<b>用于遍历可迭代对象的元素</b>

> for (property of expression) statement

```javascript
for (const el of [1, 5, 4, 6]) {
  console.log(el);
}
```

for-of 会按照可迭代对象的 next()方法产生值得顺序迭代元素。

#### 标签 & break & continue

```javascript
let num = 0;

outermost: for (let i = 0; i < 10; i++) {
  for (let j = 0; j < 10; j++) {
    if (i == 5 && j == 5) {
      continue outermost;
    }
    num++;
  }
}
console.log(num); //95
```

#### with

不推荐使用<br>
使用场景是针对一个对象反复操作

```javascript
let qs = location.search.substring(1);
let hostName = location.hostname;
let url = location.herf;
// 上方代码等同于
with (location) {
  let qs = search.substring(1);
  let hostName = hostname;
  let url = href;
}
```

#### switch

```javascript
/*
    switch(expression){
        case value1:
            statement
            break;
        case value2:
            statement
            break;
        default:
            statement
    }
*/
switch (i) {
  case 25:
  /* 跳过 */
  case 35:
    console.log("25 或 35");
    break;
  case 45:
    console.log("45");
  default:
    console.log("其他");
}
/* switch语句可以用于所有数据类型，可以使用字符串甚至对象、条件的值不需要是常量，也可以是变量或表达式。 */
let num = 25;
switch (true) {
  case num < 0:
    console.log("(-∞，0)");
    break;
  case num >= 0 && num <= 10:
    console.log("[0,10]");
    break;
  case num > 10 && num <= 20:
    console.log("(10,20]");
    break;
  default:
    console.log("(20,+∞)");
}
```

### 函数

见 [Javascript-Function](../../../../2021/03/14/Javascript-Function/)

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
