---
title: Javascript-原始包装类型
date: 2021-1-18 21:33:05
author: Kinsiy
avatar: "https://cdn.jsdelivr.net/gh/Kinsiy/cdn/img/custom/avatar.jpg"
authorLink: "https://kinsiy.github.io"
authorAbout: I'm True
authorDesc: I'm True
description: 基本引用类型-原始包装类型：Boolean、Number、String对象的使用
categories: SKILL
tags: [Notes, Javascript]
---

# 基本引用类型(原始包装类型)

为了方便操作原始值，EMCAScript 提供了 3 种特殊的引用类型：Boolean、Number 和 String。这些类型具有与其他引用类型一样的特点，但也具有与各自原始类型对应的特殊行为。每当用到某个原始值的方法或属性时，后台都会创建一个相应包装类型的对象，从而暴露出操作原始值得各种方法。
在以读模式访问字符串值得时候，后台都会执行一下三步：

-   创建一个 String 类型的实例
-   调用实例上的特定方法
-   销毁实例

```javascript
let s1 = "some text";
let s2 = s1.substring(2);
/* 可以想象成 */
let s1_ = "some text";
let s2_ = new String(s1_).substring(2);
s_ = null;
```

这种行为可以让原始值拥有对象的行为。对布尔值和数值而言，以上三步也会在后台发生，只不过使用的是 Boolean 和 Number 包装类型而已。
引用类型与原始值包装类型的主要区别在于对象的生命周期。在通过 new 实例化引用类型后，得到的实例会在离开作用域是被销毁，而自动创建的原始值包装对象则只存在于访问它执行期间。

```javascript
let s1 = "some text";
s1.color = "red";
console.log(s1.color); //undefined

/* Object作为一个工厂方法，能够根据传入值得类型返回相应原始值包装类型的实例 */
let obj = new Object("some text");
console.log(obj instanceof String);

/* 使用new调用原始值包装类型的构造函数，与调用同名的转型函数并不一样 */
let value = "25";
let number = Number(value);
let obj = new Number(value);
console.log(typeof number); // number
console.log(typeof obj); // object
```

## Boolean

建议永远不要使用 Boolean 对象
Boolean 的实例会重写 valueof()方法，返回一个原始值 true 或 false。toString()方法在被调用时也会被覆盖，返回字符串"true"或"false"。

```javascript
let falseObject = new Boolean(false);
let result = falseObject && true;
console.log(result); // true

let falseValue = false;
result = falseValue && true;
console.log(result); // false

console.log(typeof falseObject); // object
console.log(typeof falseValue); // Boolean
console.log(falseObject instanceof Boolean); // true
console.log(falseValue instanceof Boolean); // false
```

## Number

与 Boolean 类型一样，Number 类型重写了 valueof()、toLocaleString()和 toString()方法。valueof()方法返回对象表示的原始数值，另外两个方法返回数值字符串。toString()方法可选地接收一个表示基数的参数，并返回相应技术形式的数值字符串。

```javascript
let num = 10;
console.log(num.toString()); // 10
console.log(num.toString(2)); // 1010
console.log(num.toString(16)); // a

/* toFixed()方法返回包含指定小数点位数的数值字符串 */
console.log(num.toFixed(2)); // 10.00
num = 10.008;
console.log(num.toFixed(2)); // 10.01        四舍五入

/* toExponential(),返回以科学计数法表示的数值字符串 */
num = 88888;
console.log(num.toExponential(1)); // 8.9e+4

/* toPrecision(),根据情况返回最合理的输出结果 */
num = 88;
console.log(num.toPrecision(1)); // 9e+1
console.log(num.toPrecision(2)); // 88
console.log(num.toPrecision(3)); // 88.0

// 本质上toPrecision()会根据数值和精度来决定调用toFixed()还是toExponential()。为了以正确的小数位精确表示数值，这三个方法都会向上或向下舍入。

/* isInteger(), 用于辨别一个数值是否保存为整数 */
console.log(Number.isInteger(1)); // true
console.log(Number.isInteger(1.0)); // true
console.log(Number.isInteger(1.01)); // false
```

## String

String 对象的方法可以在所有字符串原始值上调用。上个继承的方法 valueof()、toLocaleString()和 toString()都返回对象的原始字符串值。
每个 String 对象都有一个 length 属性，表示字符串中字符的数量。

```javascript
let stringValue = "hello world";
console.log(stringValue.length);        // 11

// 即使字符串中包含双字节字符(而不是单字节的ASCII字符)，也仍然会按单字符来计数
let str_1 = "这串文字的长度是⑨"；
console.log(str_1.length);              // 9
```

### Javascript 字符

Javascript 字符串有 16 位码元(code unit)组成。对多数字符来说，每 16 位码元对应一个字符。换句话说，字符串的 length 属性表示字符串包含多少 16 位码元。

```javascript
/* charAt()用于返回给定索引位置的字符 */
let message = "abcde";
console.log(message.charAt(2)); // c

/* charCodeAt()方法可以查看指定码元的字符编码 */
console.log(message.charCodeAt(2)); //  99

/* fromCharCode(), 用于根据给定的UTF-16码元创建字符串中的字符。 */
console.log(String.fromCharCode(0x61, 0x62, 0x63, 0x64, 0x65)); // abcde
console.log(String.fromCharCode(97, 98, 99, 100, 101)); // abcde

/* 当字符编码扩展到Unicode增补平面时 */
/* codePointAt(), 接收16位码元的索引并返回改索引位置上的码点 */
message = "ab🤣de";
console.log(message.length); // 6
console.log(message.charAt(1)); // b
console.log(message.charAt(2)); // <?\>
console.log(message.charAt(3)); // <?\>
console.log(message.charAt(4)); // d

console.log(message.charCodeAt(1)); // 98
console.log(message.charCodeAt(2)); // 55358
console.log(message.charCodeAt(3)); // 56611
console.log(message.charCodeAt(4)); // 100

console.log(message.codePointAt(1)); // 98
console.log(message.codePointAt(2)); // 129315
console.log(message.codePointAt(3)); // 56611
console.log(message.codePointAt(4)); // 100

/* fromCodePoint(), 接受任意个码点，返回对应字符拼接起来的字符串 */
console.log(String.fromCharCode(97, 98, 55358, 56611, 100, 101)); // ab🤣de
console.log(String.fromCodePoint(97, 98, 129315, 100, 101)); // ab🤣de
```

### normalize()

某些 Unicode 字符可以有多种编码方式。有的字符即可以通过一个 BMP 字符表示，也可以通过一个代理对表示。

```javascript
let a = String.fromCharCode(0x00c5),
	b = String.fromCharCode(0x212b),
	c = String.fromCharCode(0x0041, 0x030a);
console.log(`a: ${a} b: ${b} c: ${c}`); // a: Å b: Å c: Å

console.log(a === b); // false
console.log(a === c); // false
console.log(b === c); // false

/* 使用normalize() 规范化，使用时需要传入表示那种形式的字符串："NFD","NFC","NFKD","NFKC" */
console.log(a.normalize("NFC") === b.normalize("NFC")); // true
```

### 字符串操作方法

#### 拼接-concat()

concat(), 用于将一个或多个字符串拼接成一个新字符串。常用(+)替代

```javascript
let strValue = "Hello";
let result = strValue.concat(" ", "World ", "!");
console.log(result); // Hello World !
```

#### 切片-slice(), substr(), substring()

这三个方法都返回调用他们的字符串的一个字字符串，而且都接收一或两个参数。对 slice()和 substring()而言，第二个参数是提取结束的位置。对 substr()而言，第二个参数表示返回的字符串数量。任何情况下，省略第二个参数都意味着提取到字符串末尾。

```javascript
let strValue = "Hello World!";
console.log(strValue.slice(3)); // lo World!
console.log(strValue.slice(-3)); // ld!

console.log(strValue.slice(-3, -1)); // ld
console.log(strValue.substring(3, -4)); // Hel      substring(), 把所有负参数转换为0
console.log(strValue.substr(-3, 2)); // ld
console.log(strValue.substr(-3, -2)); // ""(空字符串) substr()，第二个参数若为负数，转换为0
```

#### 定位-indexOf(), lastIndexOf()

这两个方法从字符串中搜索传入的字符串，并返回位置(如果没找到，则返回-1)。两者的区别在于，indexof()方法从字符串开头开始查找子字符串，而 lastIndexOf 从字符串末尾开始查找子字符串。这两个方法都可以接收第二个参数，表示开始搜索的位置。

```javascript
let strValue = "slipped the surly bonds of earth to touch the face of God";
console.log(strValue.indexOf("f")); // 25
console.log(strValue.lastindexOf("f")); // 52

let position = [];
let pos = strValue.indexOf("f");

while (pos > -1) {
	position.push(pos);
	pos = strValue.indexOf("f", ++pos);
}
console.log(position); // [25,46,52]
```

#### 包含-startWith(), endsWith(), includes()

用于判断字符串中是否包含另一个字符串，这些方法都会从字符串中搜索传入的字符串，并返回表示是否包含的布尔值。区别在于，startsWith()检查目标字符串是否以传入字符串开头，endsWith()检查目标字符串是否以传入字符串结尾，includes()检查目标字符串是否包含传入字符串。

```javascript
let message = "foobarbaz";

console.log(message.startsWith("foo")); // true
console.log(message.endsWith("bak")); // false
console.log(message.endsWith("bar", -3)); // false        可接受第二个参数结束索引
console.log(message.includes("oba")); // true
```

#### 修剪-trim(), trimLeft(), trimRight()

创建字符串的一个副本，删除前、后所有空格，再返回结果。原字符串不受影响

```javascript
let strValue = "   Hello  ";
console.log(strValue.trim()); // Hello
console.log(strValue.trimLeft()); // Hello__(两空格)
console.log(strValue.trimRight()); // ___Hello
```

#### 重复-repeat()

ECMAScript 在所有字符串上都提供 repeat()方法。这个方法接收一个整数参数，表示要将字符串复制多少次，然后返回拼接所有副本的结果。

```javascript
let strValue = "Kinsiy ";
console.log(strValue.repeat(5) + "🤣"); // Kinsiy Kinsiy Kinsiy Kinsiy Kinsiy 🤣
```

#### 填充-padStart(), padEnd()

padStatr()和 padEnd()方法会复制字符串，如果小于指定长度，则在相应的一边填充字符直至满足长度条件。这个方法的第一个参数是长度，第二个参数是可选的填充字符串，默认为空格(U+0020)。

```javascript
let strValue = "Kinsiy";
console.log(strValue.padStart(10, "🤣")); // 🤣🤣Kinsiy 别忘了前面说的Unicode代理对
console.log(strValue.padEnd(10, "🤣")); // Kinsiy🤣🤣

/* 可选的第二个参数不限于一个字符。如果提供了多个字符的字符串，则会将其拼接并截断以匹配指定长度。
    此外，如果长度小于或等于字符串长度，则会返回原始字符串
 */
strValue = "foo";
console.log(strValue.padStart(10, "bar")); // barbarbfoo
```

#### 迭代与解构-@@iterator()

字符串的原型上暴露了一个@@iterator()方法，表示可以迭代字符串的每个字符。可以向下面这样手动使用迭代器：

```javascript
let strValue = "ab";
let stringIterator = strValue[Symbol.iterator]();

console.log(stringIterator.next()); //{value: "a", done: false}
console.log(stringIterator.next()); //{value: "b", done: false}
console.log(stringIterator.next()); //{value: undefined, done: true}

/* 在for-of 循环中可以通过这个迭代器按序访问每个字符 */
for (const c of "abcde") {
	console.log(c);
}

/* 使用解构操作符解构为数组 */
let name = "Kinsiy";
console.log([...name]); // ["K", "i", "n", "s", "i", "y"]
```

#### 大小写-toLowerCase(), toUpperCase()

还有 toLocaleLowerCase(),toLocaleUpperCase(), 旨在基于特定地区实现。

```javascript
let strValue = "Kinsiy ";
console.log(strValue.toLowerCase()); // kinsiy
console.log(strValue.toUpperCase()); // KINSIY
```

#### 正则

##### match()

本质上跟 RegExp 对象的 exec()方法相同。match()方法接收一个参数，可以是一个正则表达式字面量，也可以是一个 RegExp 对象。

```javascript
let text = "cat, bat, sat, fat";
let pattern = /.at/;

let matches = text.match(pattern);
console.log(matches.index); // 0
console.log(matches[0]); // cat
console.log(pattern.lastIndex); // 0
```

##### search()

这个方法唯一的参数和 match()一样：正则表达式字面量或 RegExp 对象。这个方法返回模式第一个匹配位置的位置索引，如果没找到则返回-1

```javascript
let text = "cat, bat, sat, fat";
let pattern = /at/;

console.log(text.search(pattern)); // 1
```

##### replace()

这个方法接收两个参数，第一个参数可以是一个 RegExp 对象或一个字符串(这个字符串不会转换为正则表达式)，第二个参数可以是一个字符串或一个函数。如果第一个参数是字符串，那么只会替换第一个字字符串。想要替换所有子字符串，第一个参数必须为正则表达式并且带全局标记。

```javascript
let text = "cat, bat, sat, fat";
let result = text.replace("at", "ond");
console.log(result); // cond, bat, sat, fat

result = text.replace(/at/g, "ond"); // cond, bond, sond, fond
console.log(result);
```

在第二个参数是字符串的情况下，有几个特殊的字符序列，可以用来插入正则表达式操作的值。

| 字符序列 | 替换文本                                                |
| :------: | :------------------------------------------------------ |
|    $$    | $                                                       |
|    $&    | 匹配整个模式的子字符串。与 RegExp.lastMatch 相同        |
|    &'    | 匹配的子字符串之前的字符串。与 RegExp.rightContext 相同 |
|    &`    | 匹配的子字符串之后的字符串。与 RegExp.leftContext 相同  |
|    &n    | 匹配第 n 个捕获组的字符串，其中 n 是 0~9                |
|   &nn    | 匹配第 nn 个捕获组的字符串，其中 nn 是 01~99            |

```javascript
let text = "cat, bat, sat, fat";
let result = text.replace(/(.at)/g, "word ($1)");
console.log(result); // word (cat), word (bat), word (sat), word (fat)

/* replace() 第二个参数可以是一个函数
    在只有一个匹配项时，这个函数会收到3个参数：与整个模式匹配的字符串，匹配项在字符串中的开始位置，以及整个字符串。在有多个捕获组的情况下，每个匹配捕获组的字符串也会作为参数传给这个函数。但最后两个参数还是与整个模式匹配的开始位置和原始字符串。
 */

function htmlEscape(text) {
	return text.replace(/[<>"&]/g, function (match, pos, originalText) {
		switch (match) {
			case "<":
				return "&lt;";
			case ">":
				return "&gt;";
			case "&":
				return "&amp;";
			case '"':
				return "&quot;";
		}
	});
}

console.log(htmlEscape('<p class="greeting">Hello World!</p>'));
// &lt;p class=&quot;greeting&quot;&gt;Hello World!&lt;/p&gt;
```

##### split()

这个方法会根据传入的分隔符将字符串拆分成数组。作为分隔符的参数可以是字符串，也可以是 RegExp 对象。还可以传入第二个参数，即数组大小，确保返回的数组不会超过指定大小。

```javascript
let colorText = "ref,bule,green,yellow";
let colors1 = colorText.split(",");
console.log(colors1); //["ref", "bule", "green", "yellow"]

let colors2 = colorText.split(",", 2);
console.log(colors2); //["ref", "bule"]

let colors3 = colorText.split(/[^,]+/);
console.log(colors3); //["", ",", ",", ",", ""]
```

#### 比较—localeCompare()

这个方法比价两个字符串，返回如下 3 个值中的一个

-   如果按照字母表顺序，字符串应该排在字符串参数前头，则返回负值
-   如果字符串与字符串参数相等，则返回 0
-   如果按照字母表顺序，字符串应该排在字符串参数后头，则返回正值

```javascript
let stringValue = "yellow";
console.log(stringValue.localeCompare("brick")); // 1
console.log(stringValue.localeCompare("yellow")); // 0
console.log(stringValue.localeCompare("zoo")); // -1
```

#### html 方法

略
