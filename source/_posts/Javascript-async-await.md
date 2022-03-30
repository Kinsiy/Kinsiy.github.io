---
title: Javascript-async/await
date: 2021-03-22 22:41:38
categories: [学习笔记, Javascript]
tags: [JS红宝书, async/await]
keywords:
description:
photos:
---

异步函数，也称为"async/await"(语法关键字)，是 ES6 期约模式在 ECMAScript 函数中的引用。async/await 是 ES8 规范新增的。

# 异步函数

ES8 的 async/await 旨在解决利用异步结构组织代码的问题。为此，ECMAScript 对函数进行就扩展，为其增加了两个新关键字: async 和 await。

<!-- more -->

## async

async 关键字用于声明异步函数。这个关键字可以用在函数声明、函数表达式、箭头函数和方法上。使用 async 关键字可以让函数具有异步特性，但总体上其代码仍然是同步求值的。而在参数或闭包方面，异步函数仍然具有普通 Javascript 函数的正常行为。

```javascript
async function foo() {
  console.log("foo");
}

foo();
console.log(2);

// foo
// 2
// foo()函数仍然会在后面的指令之前被求值
```

不过，异步函数如果使用 return 关键字返回了值(如果没有 return 则会返回 undefined)，这个值会被 Promise.resolve()包装成一个期约对象。异步函数始终会返回期约对象。在函数外部调用这个函数可以得到它返回的期约

```javascript
async function foo() {
  console.log("1");
  return 3;
}

// 给返回的期约添加一个解决处理程序
foo().then(console.log);

console.log(2);

// 1
// 2
// 3
```

异步函数的返回值期待(但实际上并不要求)一个实现 thenable 接口的对象，但常规的值也可以。如果返回的是实现 thenable 接口的对象，则这个对象可以由提供给 then()的处理程序"解包"。如果不是，则返回值就被当作已经解决的期约。

```javascript
// 返回一个实现了thenable接口的非期约对象
async function foo() {
  const thenable = {
    then(callback) {
      callback("foo");
    },
  };
  return thenable;
}

foo().then(console.log); // foo

// 返回一个期约
async function baz() {
  return Promise.resolve("baz");
}

baz().then(console.log); // baz

// 在异步函数中抛出错误会返回拒绝的期约
async function king() {
  console.log(1);
  throw 3;
}

king().catch(console.log);
console.log(2);

// 1 -> 2 -> 3

// 拒绝期约的错误不会被捕获
async function queen() {
  console.log(4);
  Promise.reject(6);
}

queen().catch(console.log);
console.log(5);

// 4 -> 5
// Uncaught (in promise) 6
```

## await

因为异步函数主要针对不会马上完成的任务，所以自然需要哦一种暂停和恢复执行的能力、使用 await 关键字可以暂停异步函数代码的执行，等待期约解决。

```javascript
async function foo() {
  let p = new Promise((resolve, reject) => setTimeout(resolve, 1000, 3));
  console.log(await p);
}
foo(); // 3
```

注意，await 关键字会暂停执行异步函数后面的代码，让出 Javascript 运行时的执行线程。这个行为与生成器函数中的 yeild 关键字是一样的。await 关键字同样是尝试"解包"对象的值，然后将这个值传给表达式，再异步恢复异步函数的行为。await 关键字的用法与 Javascript 的一元操作一样。它可以单独使用，也可以在表达式中使用。
await 关键字期待(但实际上并不要求)一个实现 thenable 接口的对象，但常规的值也可以。如果返回的是实现 thenable 接口的对象，则这个对象可以由 await 来"解包"。如果不是，则返回值就被当作已经解决的期约。

```javascript
// 等待一个原始值
async function foo() {
  console.log(await "foo");
}
foo(); // foo

// 等待一个实现了thenable接口的对象
async function baz() {
  const thenable = {
    then(callback) {
      callback("baz");
    },
  };
  console.log(await thenable);
}

baz(); //baz

// 等待一个期约
async function king() {
  console.log(await Promise.resolve("king"));
}

king(); // king

// 等待会抛出错误的同步操作，会返回拒绝的期约
async function queen() {
  console.log(1);
  await (() => {
    throw 3;
  })();
}

queen().catch(console.log);
console.log(2);

// 1 -> 2 -> 3
```

单独的 Promise.reject()不会被异步函数捕获，而会抛出未捕获错误。不过，对拒绝的期约使用 await 则会释放(unwrap)错误值(将拒绝期约返回)

```javascript
async function kinsiy() {
  console.log(1);
  await Promise.reject("3");
  console.log(4); // 这行代码不会执行
}

kinsiy().catch(console.log);
console.log(2);

// // 1 -> 2 -> 3
```

## await 的限制

await 关键字必须在异步函数中使用，不能在顶级上下文如<Script\>标签或模块中使用。不过，定义并立即调用异步函数是没问题的。此外，异步函数的特质不会扩展到嵌套函数。因此，await 关键字也只能直接出现在异步函数的定义中。在同步函数内部使用 await 会抛出 SyntaxError。

# 停止和恢复执行

async/await 中真正起作用的是 await。async 关键字，无论从哪方面来看，都不过是一个标识符。毕竟，异步函数如果不包含 await 关键字，其执行基本上跟普通函数没有什么区别。
Javascript 运行时碰到 await 关键字时，会记录在哪里暂停执行。等到 await 右边的值可用了，Javascript 运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行。因此，即使 await 后面跟着一个立即可用的值，函数的其余部分也会被异步求值。

```javascript
async function foo() {
  console.log(2);
  await null;
  console.log(4);
}

console.log(1);
foo();
console.log(3);

// 1 -> 2 -> 3 -> 4
```

# 异步函数策略

## 实现 sleep()

```javascript
async function sleep(delay) {
  return new Promise((resolve) => setTimeout(resolve, delay));
}

async function foo() {
  const t0 = Date.now();
  await sleep(3000);
  console.log(Date.now() - t0);
}

foo(); // 3014
```

## 利用平行执行

```javascript
async function randomDelay(id) {
  // 延迟0~1000ms
  const delay = Math.random() * 1000;
  return new Promise((resolve) =>
    setTimeout(() => {
      console.log(`${id} 完成`);
    }, delay)
  );
}

async function foo() {
  const t0 = Date.now();
  const promises = Array(5)
    .fill(null)
    .map((_, i) => randomDelay(i));

  for (const p of promises) {
    await p;
  }

  console.log(`共耗时${Date.now() - t0}ms`); // ???? 为什么没有打印出来
}

foo();
//4 完成
//1 完成
//2 完成
//0 完成
//3 完成
```

## 串行执行期约

```javascript
function addTwo(x) {
  return x + 2;
}
function addThree(x) {
  return x + 3;
}
function addFive(x) {
  return x + 5;
}

async function addTen(x) {
  for (const fn of [addTwo, addThree, addFive]) {
    x = await fn(x);
  }
  return x;
}

addTen(9).then(console.log); // 19
```

## 栈追踪与内存管理

略
