---
title: Javascript-Function
date: 2021-03-14 14:54:52
categories: [学习笔记, Javascript]
tags: [JS红宝书, Function]
keywords:
description:
photos:
---

函数实际上是对象。每个函数都是 Function 类型的实例，而 Function 也有属性和方法，跟其他引用类型一样。因为函数是对象，所以函数名就是指向函数对象的指针，而且不一定与函数本身紧密绑定。

<!-- more -->

```javascript
// 函数声明
function sum_1(val_1, val_2) {
	return val_1 + val_2;
}

// 函数表达式
let sum_2 = function (val_1, val_2) {
	return val_1 + val_2;
};

// 箭头函数
let sum_3 = (val_1, val_2) => val_1 + val_2;

// 使用Function构造函数
let sum_4 = new Function(val_1, val_2, "return val_1 + val_2");
```

# 箭头函数

很大程度上，箭头函数实例化的函数对象与正式的函数表达式创建的函数对象行为是相同的。任何可以使用函数表达式的地方，都可以使用箭头函数。

```javascript
let double = x => {retuan 2 * x}

// 多个参数与无参数需要括号

let sum = (a,b) => {return a+b}

// 函数体不用大括号则，箭头后面只能有一行代码。省略大括号会隐式的返回这行代码的值

let sum = (a,b,c) => a+b+c
```

箭头函数不能使用 arguments、super 和 new.target，也不能用作构造函数。此外箭头函数也没有 prototype 属性。

# 函数名

函数名就是指向函数的指针，它们跟其他包含对象指针的变量具有相同的行为。这意味着一个函数可以有多个变体

```javascript
function sum(num_1, num_2) {
	return num_1 + num_2;
}

let anthorSum = sum;
sum = null;
console.log(anthorSum(1, 2)); // 3

// ECMAScript 6 的所有函数都会暴露一个只读的name 属性，其中包含关于函数的信息

let king = () => {};

console.log(anthorSum.name); // sum
console.log(king.name); // king
console.log((() => {}).name); // 空字符串
console.log(new Function().name); // anonymous

// 如果函数是一个获取函数、设置函数，或者使用bind()实例化，那么标识符前面会加上一个前缀

function queen() {}
console.log(queen.bind(null).name); // bound queen

let person = {
	year_: 2021,
	get year() {
		return this.year_;
	},
	set year(newYear) {
		console.log(`oldYear: ${this.year_}`);
		this.year_ = newYear;
	},
};

let propertyDescriptor = Object.getOwnPropertyDescriptor(person, "year");
console.log(propertyDescriptor.get.name); // get year
console.log(propertyDescriptor.set.name); // set year
```

# 理解参数

ECMAScript 函数的参数跟大多数其他语言不同。ECMAScript 函数既不关系传入的参数个数，也不关心这些参数的数据类型。定义函数时要接收两个参数，并不意味着调用时就传两个参数。你可以传一个、三个，甚至一个也不传，解释器也不会报错。
ECMAScript 函数的参数在内部表现为一个数组。在使用 function 关键字定义(非箭头)函数时，可以在函数内部访问 arguments 对象，从中取得传递进来的每个参数值。arguments 对象是一个类数组对象(不是 Array 实例)

```javascript
function sum(val_1, val_2) {
	if (arguments.length == 1) {
		arguments[1] = 66;
		console.log(val_1 + 10);
		console.log(val_2);
	} else {
		arguments[1] = 88;
		console.log(val_2);
		val_2 = 100;
		console.log(arguments[1]);
	}
}

// arguments 对象的长度由传入的命名参数数量决定。
// auguments[n](n > 传入参数数量)对象与命名参数不同步，argument[n](n < 传入参数数量)与命名参数始终保持同步

sum(4); //14  undefined
sum(4, 5); // 88 100

/* 《Javascript高级程序设计》中单向同步的说法应该是错的，在chrome[ 89.0.4389.82（正式版本） （64 位）]中测试是双向同步的 */
```

如果函数是使用箭头函数定义的，那么传给函数的参数将不能使用 arguments 关键字访问，而只能通过定义的命名参数访问。

# 没有重载

ECMAScript 函数不能像传统编程那样重载。在其他语言比如 Java 中，一个函数可以有两个定义，只要签名(接收函数的类型和数量)不同就行。ECMAScript 中函数没有签名，因为参数是由包含零个或多个值得数组表示。没有函数签名，自然也就没有重载。如果在 ECMAScript 中定义了两个同名函数，则后定义的会覆盖先定义的。

# 默认函数值

在 ECMAScript5.1 及以前，实现默认参数的一种常用方式就是检测某个参数是否等于 undefined。如果是则意味着没有传这个参数，那就给它赋一个值。ES6 之后就不用这么麻烦了，因为它支持显式定义默认参数了

```javascript
// ES 5
function makeKing(name) {
	name = typeof name !== "undefined" ? name : "Kinsiy";
	return `King ${name} VII`;
}

// ES 6
function makeQueen(name = "QingLai") {
	return `Queen ${name} VII`;
}

// 在使用默认参数时，arguments 对象的值不反应参数的默认值，只反映传给函数的参数
// 跟ES5 严格模式一样，修改命名参数也不会影响arguments对象，它始终以调用函数时传入的值为准
```

## 默认函数作用域与暂时性死区

给多个参数定义默认值实际上跟使用 let 关键字顺序声明变量一样。参数初始化顺序遵循"暂时性死区"规则，即前面定义的参数不能引用后面定义的。

```javascript
// 报错 ReferenceError
function marry(king = queen, queen = "QingLai") {
	return `king ${king} queen ${queen}`;
}

// 参数也存在于自己的作用域中，它们不能引用函数体的作用域

// 报错 ReferenceError
function makeMarry(name = "Kinsiy", age = num) {
	let num = 18;
	console.log(`age ${age}`);
}
```

# 参数扩展与收集

ECMAScript 6 新增了扩展操作符，使用它可以非常简洁地操作和组合集合数据。扩展操作符最有用的场景就是函数定义中的参数列表。

## 扩展参数

```javascript
let values = [1, 2, 3, 4];

function countArguments() {
	console.log(arguments.length);
}

countArguments(-1, ...values); // 5
```

## 收集参数

在构思函数定义时，可以使用扩展操作符把不同长度的独立参数组合为一个数组。这有点像 arguments 对象的构造机制，只不过收集参数的结果会得到一个 Array 实例。因为收集参数的结果可变，所以只能把它作为最后一个参数：

```javascript
function getSum(...values) {
	return values.reduce((x, y) => x + y, 0);
}

console.log(getSum(1, 2, 3, 4, 5, 6)); // 21
```

# 函数声明与函数表达式

函数声明会在任何代码执行之前先被读取并添加到执行上下文。这个过程叫作函数声明提升。而函数表达式必须等到代码执行到它那一行，才会在执行上下文中生成函数定义。

```javascript
console.log(sum(10, 5)); // 15

function sum(num_1, num_2) {
	return num_1 + num_2;
}

console.log(sum_1(20, 5)); // ReferenceError: Cannot access 'sum_1' before initialization
let sum_1 = function (num_1, num_2) {
	// 使用var 也是一样的
	return num_1 + num_2;
};
```

除了函数什么时候真正有定义这个区别之外，这两种语法是等价的。

# 函数作为值

如果是访问函数而不是调用函数，那就必须不带括号

```javascript
function createComparisonFunction(propertyName) {
	return function (obj_1, obj_2) {
		let value_1 = obj_1[propertyName];
		let value_2 = obj_2[propertyName];

		if (value_1 < value_2) {
			return -1;
		} else if (value_1 > value_2) {
			return 1;
		} else {
			return 0;
		}
	};
}

data = [
	{ name: "Kinsiy", age: 22 },
	{ name: "QingLai", age: 21 },
];

data.sort(createComparisonFunction("age"));

console.log(data[0].name); // QingLai
```

# 函数内部

ES 5 中，函数内部存在两个特殊的对象：arguments 和 this。ES 6 新增了 new.target 属性

## arguments

arguments 前面已经说过很多次了，这里主要讲一下 arguments 对象的 callee 属性，是一个指向 arguments 对象所在函数的指针。

```javascript
function factorial(num) {
	if (num <= 1) {
		return 1;
	} else {
		return num * arguments.callee(num - 1); // 递归函数逻辑与函数名解耦
	}
}

console.log(factorial(5)); // 120
```

## this

this 在标准函数和箭头函数中有不同的行为。

-   在标准函数中 this 引用的是把函数当成方法调用的上下文对象，这时候通常称其为 this 值
-   在剪头函数中，this 引用的是定义箭头函数的上下文

```javascript
window.color = "purple";

let o = {
	color: "orange",
};

function sayColor() {
	console.log(this.color);
}

let sayColor_1 = () => console.log(this.color);

sayColor(); // purple
sayColor_1(); // purple

o.sayColor = sayColor;
o.sayColor(); // orange

o.sayColor_1 = sayColor_1;
o.sayColor_1(); // purple
```

## caller

这个属性引用的是调用当前函数的函数，或者如果是在全局作用域中调用的则为 null

```javascript
function outer() {
	inner();
}

function inner() {
	console.log(arguments.callee.caller);
}

outer(); // ƒ outer()
```

## new.target

检测函数是否使用 new 关键字调用。如果函数是正常调用的，new.target 的值是 undefined；如果是使用 new 关键字调用的，则 new.target 将引用被调用的构造函数。

```javascript
function King() {
	if (!new.target) {
		throw new Error("King 必须new");
	}
	console.log(`new King`);
}

new King(); // new King
King(); // Error: King 必须new
```

# 函数属性与方法

每个函数都有两个属性：length 和 prototype。其中 length 属性保存函数定义的命名参数的个数。prototype 是保存引用类型所有实例方法的地方，这意味着 toString()、valueOf()等方法实际上都保存在 prototype 上，进而由所有实例共享。

```javascript
function sayName(name) {
	console.log(name);
}

function sum(num_1, num_2) {
	return num_1 + num_2;
}

function sayHi() {
	console.log("Hi");
}

console.log(sayName.length); // 1
console.log(sum.length); // 2
console.log(sayHi.length); // 0
```

函数还有两个方法: apply()和 call()。这两个方法都会以指定 this 值来调用函数，即会设置调用函数时函数体内 this 对象的值.apply()方法接受两个参数：函数内 this 的值和一个参数数组。第二个参数可以是 Array 的实例，但也可以是 arguments 对象。call()方法与 apply()的作用一样，只是传参的形式不同，第一个参数跟 apply()一样，也是 this 值，而剩下的要传给被调用函数的参数则是逐个传递的。换句话说，通过 call()向函数传参时，必须将参数一个个地列出来。

```javascript
function sum(num_1, num_2) {
	console.log(this);
	return num_1 + num_2;
}

function applysum(num_1, num_2) {
	return sum.apply(this, arguments);
}

function callsum(num_1, num_2) {
	return sum.call(this, num_1, num_2);
}

sum(1, 2); // window

let object = {};
applysum.call(object, 3, 4); // {}

// ES 5 出于同样的目的定义了一个新方法：bind()。bind()方法会创建一个新的函数实例，其this值会被绑定到传给this的对象。

window.color = "orange";
let o = {
	color: "purple",
};

function sayColor() {
	console.log(this.color);
}

let ObjectSayColor = sayColor.bind(o);

ObjectSayColor(); // purple
```

# 函数表达式

记住。函数声明会被提升，而函数表达式不会被提升。任何时候，只要函数被当作值来使用，它就是一个函数表达式。

# 递归

在编写递归函数时，arguments.callee 是引用当前函数的首选。在严格模式下可以使用命名函数表达式达到目的。

```javascript
const factorial = function f(num) {
	if (num <= 1) {
		return 1;
	} else {
		return num * f(num - 1);
	}
};

console.log(factorial(5)); // 120
```

# 尾调用优化

ES 6 规范新增了一项内存管理优化机制，让 javascript 引擎在满足条件时重用栈帧。具体来说，这项优化非常适合"尾调用"，即外部函数的返回值是一个内部函数的返回值。
示例及适用条件略(:

# 闭包

闭包指的是那些引用了另一个函数作用域中变量的函数，通常是在嵌套函数中实现的。

```javascript
function createComparisonFunction(propertyName) {
	return function (obj_1, obj_2) {
		let value_1 = obj_1[propertyName];
		let value_2 = obj_2[propertyName];

		if (value_1 < value_2) {
			return -1;
		} else if (value_1 > value_2) {
			return 1;
		} else {
			return 0;
		}
	};
}

let compare = createComparisonFunction("name");
let result = compare({ name: "Kinsiy" }, { name: "Restituo" });
```

compare 函数就是一个闭包，他的作用域链包含本身函数的的作用域与 createComparisonFunction()的作用域以及全局作用域。createComparisonFunction()活动对象并不能在它执行完毕后销毁，因为匿名函数的作用域链中仍然有对它的引用。

## this 对象

```javascript
window.identity = "the window";
let object = {
	identity: "my object",
	getIdentity() {
		return function () {
			console.log(this.identity); // 内部函数永远不可能直接访问外部函数的this与arguments
		};
	},
};

console.log(object.getIdentity()()); // the window
```

## 内存泄露

因为 IE 在 IE9 之前对 JScript 和 COM 对象使用了不同的垃圾回收机制，所以 IE 在旧版本 IE 中可能回导致内存泄露。

# 立即调用表达式

立即调用的匿名函数又被称作立即调用的函数表达式。它类似与函数声明，但由于被包含在括号中，所以会被解释为函数表达式。紧跟在第一组括号后面的第二组括号会立即调用前面的函数表达式。

```javascript
(function () {
	// 块级作用域
})()(
	// ES5 中常用来隔离块级作用域
	function () {
		for (var i = 0; i < 3; i++) {
			console.log(i);
		}
	}
)(); // 0,1,2
```

# 私有变量

略
