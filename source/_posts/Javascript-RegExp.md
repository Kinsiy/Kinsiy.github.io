---
title: Javascript-RegExp
date: 2021-1-17 22:14:48
description:
categories: [学习笔记, Javascript]
tags: [JS红宝书, RegExp]
---

## RegExp

ECMAscript 通过 RegExp 类型支持正则表达式。正则表达式使用类似 Perl 的简洁语法来创建:

```javascript
let expression = /pattern/afgls;
```

这个正则表达式的 pattern(模式)可以是任何简单或复杂的正则表达式，包括字符类、限定符、分组、先前查找和反向查找和反向引用、每个正则表达式可以带零个或多个 flags(标记)、用于控制正则表达式的行为。下面给出了表示匹配模式的标记。

<!-- more -->

-   g：全局模式，表示查找字符串的全部内容，而不是找到第一个匹配的内容就结束
-   i：不区分大小写，表示在查找匹配时忽略 pattern 和字符串的大小写
-   m：多行模式，表示查到一行文本末尾时会继续查找
-   y：粘贴模式，表示只查找从 lastIndex 开始及之后的字符串
-   u：Unicode 模式，启用 Unicode 匹配
-   s：dotAll 模式，表示元字符。匹配任何字符(包括\n 或\r)

```javascript
/* 匹配 ( [ { \ ^ | } ] ) ? * = . 需要使用(\)转义 */

/* 使用字面量形式定义 */

let pattern1 = /at/g; // 匹配字符串中的所有"at"
let pattern2 = /[bc]at/i; // 匹配第一个"bat"或"cat",忽略大小写
let pattern3 = /\.at/gi; // 匹配所有以".at"结尾的三字符组合，忽略大小写

/* 使用RegExp构造函数定义 
    RegExp构造函数接受两个参数：模式字符串和标记字符串(可选)
    因为RegExp的模式参数是字符串，所以在某些情况下需要二次转义
*/
let pattern4 = new RegExp("\\.at", "gi"); // 与pattern3等价

let pattern5 = new RegExp(pattern2, "gi"); // 基于已有的正则表达式实例，可以选择性的修改它们的标记
console.log(pattern5); // /[bc]at/gi

/* 无论正则表达式是怎么创建的，继承的方法toLocalString() 和toString() 都返回正则表达式的字面量表示 */
console.log(patter4.toLocalString()); // /\.at/gi
console.log(patter4.toString()); // /\.at/gi
```

### RegExp 实例属性

-   global: 布尔值，表示是否设置了 g 标记
-   ignoreCase: 布尔值，表示是否设置了 i 标记
-   unicode: 布尔值，表示是否设置了 u 标记
-   sticky: 布尔值，表示是否设置了 y 标记
-   lastIndex: 整数，表示在源自符串中下一次搜索的开始位置，始终从 0 开始
-   multiline: 布尔值，表示是否设置了 m 标记
-   dotAll: 布尔值，表示是否设置了 s 标记
-   source：正则表达式的字面量字符串(不是传给构造函数的模式字符串)，没有开头和结尾的斜杠
-   flags: 正则表达式的标记字符串。始终以字面量而非传入构造函数的字符串模式形式返回(没有前后斜杠)

```javascript
let pattern = new RegExp("\\[bc\\]at", "i");
console.log(pattern.global); // false
console.log(pattern.ignoreCase); // true
console.log(pattern.unicode); // false
console.log(pattern.sticky); // false
console.log(pattern.lastIndex); // 0
console.log(pattern.multiline); // false
console.log(pattern.dotAll); // false
console.log(pattern.source); // \[bc\]at
console.log(pattern.flags); // i
```

### RegExp 实例方法

#### exec()

RegExp 实例的主要方法是 exec(),主要用于配合捕获组使用。这个方法只接受一个参数，即要应用的模式字符串。如果找到了匹配项。则返回包含第一个匹配信息的数组；如果没找到匹配项，则返回 null。返回的数组虽然是 Array 的实例，但包含两个额外的属性：index 和 input。index 是字符串中匹配模式的起始位置，input 是要查找的字符串。这个数组的第一个元素是匹配整个模式的字符串，其他元素是与表达式中的捕获组匹配的字符串。如果模式中没有捕获组，则数组只包含一个元素。

```javascript
let text = "mom and dad and bady";
let pattern = /mom( and dad( and bady)?)?/gi;

let matches = pattern.exec(text);
console.log(matches.index); // 0
console.log(matches.input); // mom and dad and baby
console.log(matches[0]); // mom and dad and baby
console.log(matches[1]); // and dad and baby
console.log(matches[2]); // and bady

/* 如果模式设置了全局模式，则每次调用exec()方法会返回一个匹配的信息。直至搜索到字符串的末尾，此时返回null
    反之，则无论对同一字符串调用了多少次exec(), 也只返回第一个匹配的信息
*/

let string = "cat, bat, sat";
let pattern_2 = /.at/g;
let matches_2 = pattern_2.exec(string);
console.log(matches_2.index); // 0
console.log(matches_2[0]); // cat
console.log(matches_2.lastIndex); // 3

matches_2 = pattern_2.exec(string);
console.log(matches_2.index); // 5
console.log(matches_2[0]); // bat
console.log(matches_2.lastIndex); // 8

matches_2 = pattern_2.exec(string);
console.log(matches_2.index); // 10
console.log(matches_2[0]); // sat
console.log(matches_2.lastIndex); // 13

matches_2 = pattern_2.exec(string);
console.log(matches_2); //null
```

#### test()

正则表达式的另一个方法是 test(),接收一个字符串参数。如果输入的文本与模式匹配，则参数返回 true，反之返回 false。这个方法适用于只想测试模式是否匹配，而不需要实际匹配内容的情况。

```javascript
let text = "000-00-0000";
let pattern = /\d{3}-\d{2}-\d{4}/;

if (pattern.test(text)) {
  console.log("匹配成功！"); // 匹配成功！
}
```

### RegExp 构造函数属性

RegExp 构造函数本身也有几个属性。这些属性适用于作用域中的所有正则表达式，而且会根据最后执行的正则表达式操作而变化。

|     全名     | 简写 |                   说明                    |
| :----------: | :--: | :---------------------------------------: |
|    input     | $\_  |             最后搜索的字符串              |
|  lastMatch   |  $&  |             最后匹配的字符串              |
|  lastParen   |  $+  |               最后的捕获组                |
| leftContext  |  $`  | input 字符串中出现在 lastMatch 前面的文本 |
| rightContext |  $'  | input 字符串中出现在 lastMach 后面的文本  |

```javascript
let text = "this has been a short winter";
let pattern_1 = /(..)or(.)/g;
let pattern_2 = /(..)win/g;

let matches_1 = pattern_2.exec(text);
console.log(RegExp.input); // this has been a short winter
console.log(RegExp.lastMatch); // t win
console.log(RegExp["$+"]); // t (t后边有一个空格)
console.log(RegExp["$`"]); // this has been a shor
console.log(RegExp["$'"]); // ter

pattern_1.test(text);
console.log(RegExp["$&"]); // short
console.log(RegExp.$1); // sh
console.log(RegExp["$2"]); // t
```

### 模式局限

下列特性目前还没有得到 EMCAScript 的支持

-   \\A 和\\Z 锚(分别匹配字符串的开始和结尾)
-   联合及交叉类
-   原子组
-   x(忽略空格)匹配模式
-   条件式匹配
-   正则表达式注释

## 参考

[1\][JavaScript高级程序设计(第4版).](https://book.douban.com/subject/35175321/)
