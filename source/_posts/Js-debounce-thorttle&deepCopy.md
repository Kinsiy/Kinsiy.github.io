---
title: 防抖-节流-深拷贝
date: 2021-04-22 11:04:51
tags: [debounce, thorttle, deepClone]
categories: [学习笔记,Javascript]
keywords:
description:
photos: 
---

## 防抖函数

防抖，也叫去抖动，就是当事件快速连续不断触发时，动作只会执行一次。触发事件 delay 时间后再执行回调函数，如果在 delay 时间内又触发了事件，则重新计时

防抖函数分为，延迟防抖 和 前缘防抖，区别就是是否会立即执行。

{% note primary %}

简单实现, 生产环境可以使用 [Lodash 库函数](https://lodash.com/docs/),  [可视化查看 原生 - 防抖 - 节流 三种处理方式的不同](http://demo.nimius.net/debounce_throttle/)

{% endnote %}

<!-- more -->

```js
function debounce(fn, delay) {
	let timer = null;
	return function (...args) {
		if (timer) {
			clearTimeout(timer);
		}
		timer = setTimeout(() => {
			fn.apply(this, args);
		}, delay);
	};
}

// 前缘防抖
function debounceBefore(fn, delay) {
	let timer = null;
	return function (...args) {
		if (timer) {
			clearTimeout(timer);
		}
		if (!timer) {
			fn.apply(this, args);
		}
		timer = setTimeout(() => {
			timer = null;
		}, delay);
	};
}
```

{% note info %}

[Lodash _.debounce](https://github.com/lodash/lodash/blob/4.17.15/lodash.js#L10304)

{% endnote %}

## 节流函数

节流，就是当事件快速连续不断触发时，每隔 delay 时间间隔执行一次

```js
function thorttle(fn, delay) {
	let working = false;
	return function (...args) {
		if (working) {
			return;
		}
		working = true;
		setTimeout(() => {
			fn.apply(this, args);
			working = false;
		}, delay);
	};
}

// 前缘节流
function thorttleBefore(fn, delay) {
	let working = false;
	return function (...args) {
		if (working) {
			return;
		}
		if (!working) {
			fn.apply(this, args);
		}
		working = true;
		setTimeout(() => {
			working = false;
		}, delay);
	};
}
```

{% note info %}

[Lodash _.throttle](https://github.com/lodash/lodash/blob/4.17.15/lodash.js#L10897)

{% endnote %}

## 深拷贝

```js
function isPrimitive(value) {
	return (
		typeof value === "string" ||
		typeof value === "number" ||
		typeof value === "symbol" ||
		typeof value === "boolean"
	);
}

function isObject(value) {
	return Object.prototype.toString.call(value) === "[object Object]";
}

function isRegExp(value) {
	return Object.prototype.toString.call(value) === "[object RegExp]";
}

function deepCopy(value) {
	// debugger;
	let memo = {};

	function baseCopy(value) {
		let res;
		if (isPrimitive(value) || isRegExp(value)) {
			return value;
		} else if (Array.isArray(value)) {
			res = [...value];
		} else if (isObject) {
			res = { ...value };
		}

		Reflect.ownKeys(res).forEach((key) => {
			if (typeof res[key] === "object" && typeof res[key] != null) {
				if (memo[res[key]]) {
					res[key] = memo[res[key]];
				} else {
					memo[res[key]] = res[key];
					res[key] = arguments.callee(res[key]);
				}
			}
		});
		return res;
	}

	return baseCopy(value);
}
```

## 参考

[1] [重拾 JS——防抖与节流](https://juejin.cn/post/6844904144466083853)

[2] [Javascriipt 深拷贝](https://segmentfault.com/a/1190000015455662)

