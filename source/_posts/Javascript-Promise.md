---
title: Javascript-Promise
date: 2021-03-21 20:25:55
categories: [学习笔记, Javascript]
tags: [JS红宝书, Promise]
keywords:
description:
photos:
---

期约是对尚不存在的的结果的一个替身。ESMAScript 6 增加了对 Promise/A+规范的完善支持，即 Promise 类型。一经退出，Promise 就大受欢迎，成为了主导性的异步编程机制。所有的现代浏览器都支持 ES6 期约，很多其他浏览器 API(如 fetch()和电池 API)也以期约为基础。

# 期约基础

ESMAScript 6 新增的引用类型 Promise，可以通过 new 操作符来实例化。创建新期约时需要传入执行器(executor)函数作为参数。

<!-- more -->

## 期约状态

-   待定(pending)
-   兑现(fulfilled，有时候也称为"解决"，resolved)
-   拒绝(rejected)
    待定(pending)是期约的最初始状态。在待定状态下，期约可以落定(settled)为代表成功的兑现(fulfilled)状态，或者代表失败的拒绝(rejected)状态。无论落定为那种状态都是不可逆的。只要从待定转换为兑现或拒绝，期约的状态就不再改变。而且，也不能保证期约必然会脱离待定(pending)状态。因此，组织合理的代码无论期约解决(resolve)还是拒绝(reject)，甚至是永远待定(pending)状态，都应该具有恰当的行为。

## 解决值、拒绝理由己期约用例

期约主要有两大用途。首先是抽象地表示一个异步操作。期约的状态代表期约是否完成。"待定"表示尚未开始或者正在执行中。"兑现"表示已经成功完成，而"拒绝"则表示没有成功完成。
每个期约只要状态切换为兑现，就会有一个私有的内部值(value)。类似的，每个期约只要状态切换为拒绝，就会有一个私有的内部理由(reason)。无论是值还是理由，都是包含原始值或对象的不可修改的引用。二者都是可选的，而且默认值为 undefined。在期约到达某个落定状态时执行的异步代码始终会收到这个值或理由。

## 通过执行函数控制期约状态

由于期约的状态是私有的，所以只能在内部进行操作。内部操作在期约的执行器函数中完成。执行器函数主要有两项职责：初始化期约的异步行为和控制状态的最终转换。其中，控制期约状态的转换是通过调用它的两个函数参数实现的。这两个函数擦参数通常都命名为 resolve()和 reject()。调用 resolve()会把状态切换为兑现，调用 reject()会把状态切换为拒绝。另外，调用 reject()也会抛出错误(后面讨论)。

```javascript
let p = new Promise((resolve, reject) => {
  setTimeout(reject, 10000); // 10 秒后调用reject
});

setTimeout(console.log, 0, p); // Promise
setTimeout(console.log, 11000, p); // 11秒后再检查状态  Promise {<rejected>: undefined}
```

<b>注意：</b>执行器函数是同步执行的

## Promise.resolve()

期约并非一开始就必须处于待定状态。然后通过执行器函数才能转换为落定状态。通过调用 Promise.resolve()静态方法，可以实例化为一个解决的期约。使用这个静态方法，实际上可以把任何值都转换为一个期约

```javascript
let p1 = new Promise((resolve, reject) => resolve());
let p2 = Promise.resolve();

setTimeout(console.log, 0, Promise.resolve()); // Promise {<fulfilled>: undefined}

// 这个解决的期约的值对应着传给Promise.resolve()的第一个参数。多于的参数会忽略

setTimeout(console.log, 0, Promise.resolve(1, 2, 3)); // Promise {<fulfilled>: 1}
```

对这个静态方法而言，如果传入的参数本身就是一个期约，那它的行为就类似一个空包装。因此，Promise.resolve()可以说是一个幂等方法。这个幂等性会保留传入期约的状态。

## Promise.reject()

与 Promise.resolve()类似，Promise.reject()会实例化一个拒绝的期约并抛出一个异步错误(这个错误不能通过 try/catch 捕获，而只能通过拒绝处理程序捕获)

```javascript
let p1 = new Promise((resolve, reject) => reject());
let p2 = Promise.reject();

setTimeout(console.log, 0, Promise.reject()); // Promise {<rejected>: undefined}

// 这个解决的期约的值对应着传给Promise.reject()的第一个参数。多于的参数会忽略

setTimeout(console.log, 0, Promise.reject(1, 2, 3)); // Promise {<rejected>: 1}

setTimeout(console.log, 0, Promise.reject(Promise.resolve())); // Promise {<rejected>: Promise}
```

关键在于，Promise.reject()并没有照搬 Promise.resolve()的幂等逻辑。如果给它传一个期约对象，则这个对象会成为它返回的拒绝期约的理由

## 同步/异步执行的二元性

Promise 的设计很大程度上会导致一种完全不同与 Javascript 的计算模式。

```javascript
try {
  throw new Error("foo");
} catch (e) {
  console.log(e);
}
// Error: foo

try {
  Promise.reject(new Error("bar"));
} catch (e) {
  console.log(e);
}

// Uncaught (in promise) Error: bar
```

拒绝期约的错误并没有抛到执行同步代码的线程里，而是通过浏览器异步消息队列来处理的。因此，try/catch 块并不能捕获该错误。代码一旦开始以异步模式执行，则唯一与之交互的方式就是使用异步结构——更具体地说，就是期约的方法。

# 期约的实例方法

期约实例的方法是连接外部同步代码与内部异步代码之间的桥梁，这些方法可以访问异步操作返回的数据，处理期约成功和返回的输出，连续对期约求值，或者添加只有期约进入终止状态时才会执行的代码。

## 实现 Thenable 接口

在 ECMAScript 暴露的异步结构中，任何对象都有一个 then()方法。这个方法被认为实现了 Thenable()接口。ECMAScript 的 Promise 类型实现了 Thenable 接口

## Promise.prototype.then()

Promise.prototype.then()是为期约实例添加处理程序的主要方法。这个 then()方法接收最多两个参数：onResolved()处理程序和 onReject()处理程序。这两个参数都是可选的，如果提供的话，则会在期约分别进入"兑现"和拒绝"拒绝"状态时执行。传给 then()的任何非函数类型的参数都会被静默忽略。

```javascript
function onResolved(id) {
  setTimeout(console.log, 0, id, "resolved");
}

function onReject(id) {
  setTimeout(console.log, 0, id, "rejected");
}

let p1 = new Promise((resolve, reject) => setTimeout(resolve, 3000));
let p2 = new Promise((resolve, reject) => setTimeout(reject, 3000));

p1.then(
  () => onResolved("p1"),
  () => onReject("p1")
);
// 这里使用箭头函数而不直接写onResolved("p1")的原因的，then是同步执行的，
// 若直接使用函数名，函数会直接执行，而等到期约落定时不会执行任何函数

// 不传onResolve()处理程序的规范写法
p2.then(null, () => onReject("p2"));

// 3秒后
// p1 resolved
// p2 rejected
```

Promise.prototype.then()返回一个新的期约实例。这个新期约实例基于 onResovled 处理程序的返回值构建。换句话说，改处理程序的返回值会通过 Promise.resolve()包装来生成新期约。如果没有提供这个处理程序，则 Promise.resolve()就会包装上一个期约解决之后的值。

```javascript
let p1 = Promise.resolve("foo");

let p2 = p1.then();
setTimeout(console.log, 0, p2); // Promise {<fulfilled>: "foo"}
setTimeout(console.log, 0, p1 === p2); // false

// 如果有显式返回值，则Promise.resolve()会包装这个值
let p3 = p1.then(() => "bar");
setTimeout(console.log, 0, p3); // Promise {<fulfilled>: "bar"}

// 抛出异常会返回拒绝的期约
let p4 = p1.then(() => {
  throw "baz";
});
setTimeout(console.log, 0, p4); // Promise {<rejected>: "baz"}

// 而返回错误值不会触发拒绝行为
let p5 = p1.then(() => new Error("kinsiy"));
setTimeout(console.log, 0, p5); // Promise {<fulfilled>: Error: kinsiy
```

onRejected()处理程序也与之类似：onRejected()处理程序返回的值也会被 Promise.resolve()包装。

## Promise.prototype.catch()

Promise.prototype.catch()方法用于给期约添加拒绝处理程序。这个方法只接受一个参数：onRejected()处理程序。事实上，这个方法就是一个语法糖，调用它就相当于调用 Promise.prototype.then(null, onRejected)

```javascript
let p1 = Promise.reject();
let onRejected = function () {
  setTimeout(console.log, 0, "rejected");
};

// 这两种添加拒绝处理程序的行为是一样的
p1.then(null, onRejected); // rejected
p1.catch(onRejected); // rejected
```

Promise.prototype.catch()返回一个新的期约实例，其行为与 Promise.prototype.then()的 onRejected 处理程序是一样的。

## Promise.prototype.finally()

Promise.prototype.finally()方法用于给期约添加 onFinally()处理程序，这个处理程序在期约转换为解决或拒绝状态室都会执行。onFinally()处理程序没有办法知道期约的状态是解决还是拒绝，这个方法主要用于添加清理代码

```javascript
let p1 = Promise.resolve();
let p2 = Promise.reject();
let onFinally = function () {
  setTimeout(console.log, 0, "Finally!");
};

// 这两种添加拒绝处理程序的行为是一样的
p1.finally(onFinally); // Finally!
p2.finally(onFinally); // Finally!
```

Promise.prototype.finally()方法返回一个新的期约实例。这个新期约实例不同于 then()或 catch()方式返回的实例。因为 onFinally 被设计为一个状态无关的方法，所以多数情况下它都会原样后传父期约。无论父期约是解决还是拒绝，都会原样后传。

## 非重入期约方法

当期约进入落定状态时，与改状态相关的处理程序仅仅会被排期，而非立即执行。跟在添加这个处理程序的代码之后的同步代码一定会在处理程序之前先执行。即使期约一开始就是与附加程序关联的状态，执行顺序也是这样的。这个特性由 Javascript 运行时保证，被称为"非重入"(non-reentrancy)特性.

```javascript
let synchronousResolve;

let p = new Promise((resolve) => {
  synchronousResolve = function () {
    console.log("1: invoking resolve");
    resolve();
    conosole.log("2: resolve returns");
  };
});
console.log("Kinsiy test");
p.then(() => console.log("4: then() handler executes"));

synchronousResolve();
console.log("3: synchronousResolve() returns");

// 1: invoking resolve
// 2: resolve returns
// 3: synchronousResolve() returns
// 4: then() handler executes
```

非重入适用于 onResolved/onRejected 处理程序、catch()处理程序和 finally()处理程序

## 邻近处理程序的执行顺序

如果给期约添加了多个处理程序，当期约状态变化时，相关处理程序会按照添加它们的顺序依次执行。无论是 then()、catch()还是 finally()添加的处理程序都是如此。

```javascript
let p1 = Promise.resolve();
let p2 = Promise.resolve();

p1.then(() => setTimeout(console.log, 0, 1));
p1.then(() => setTimeout(console.log, 0, 2));
// 1
// 2

p2.then(() => setTimeout(console.log, 0, 3));
p2.then(() => setTimeout(console.log, 0, 4));
// 3
// 4

p2.catch(() => setTimeout(console.log, 0, 5));
p2.catch(() => setTimeout(console.log, 0, 6));
// 5
// 6

p1.finally(() => setTimeout(console.log, 0, 7));
p1.finally(() => setTimeout(console.log, 0, 8));
// 7
// 8
```

## 传递解决值与拒绝理由

到了落定状态后，期约会提供其解决值(如果兑现)或其他拒绝理由(如果拒绝)给相关状态的处理程序。

```javascript
let p1 = new Promise((resolve) => resolve("foo"));
p1.then((value) => console.log(value)); // foo

let p2 = new Promise((resolve, reject) => reject("bar"));
p2.catch((reason) => console.log(reason)); // bar
```

## 拒绝期约与拒绝错误处理

拒绝期约类似于 throw()表达式，因为它们都代表一种程序状态，即需要中断或特殊处理。在期约的执行函数或处理程序中抛出错误会导致错误，对应的错误对象会成为拒绝的理由。
正常情况下，在通过 throw()关键字抛出错误时，Javascript 运行时的错误处理程序会停止执行抛出错误之后的任何指令。但是，在期约中抛出错误时，因为错误实际上是从消息队列中异步抛出的，所有并不会阻止运行时继续执行同步指令。

```javascript
// 正常情况下
throw Error("foo");
consle.log("bar"); // 不会执行

// 期约中
new Promise((resolve, reject) => {
  throw Error("foo");
});
console.log("bar"); // bar
// Uncaught (in promise) Error: foo
```

跟之前的 Promise.reject()示例所示，异步错误只能通过异步的 onRejected 处理程序捕获。
then()和 catch()的 onRejected 处理程序在语义上相当于 try/catch。出发点都是捕获错误后将其隔离，同时不影响正常逻辑执行。为此，onRejected 处理程序的任务应该是在捕获异步错误之后返回一个解决的期约。

```javascript
new Promise((resolve, reject) => {
  consoel.log("开始异步执行");
  reject(Error("kinsiy"));
})
  .catch((e) => {
    console.log("错误捕获: ", e);
  })
  .then(() => {
    console.log("继续异步执行");
  });

// 开始异步执行
// 错误捕获:  Error: kinsiy
// 继续异步执行
```

# 期约连锁与期约合成

多个期约组合在一起可以构成强大的代码逻辑。这种组合可以通过两种方式实现：期约连锁与期约合成。前者是一个期约接一个期约地拼接，后者则是多个期约组合为一个期约。

## 期约连锁

把期约逐个地串联起来是一种非常有用的编程模式。之所以可以这样做，是因为每个期约实例的方法(then(),catch(),finally())都会返回一个新的期约对象，而这个新期约又有自己的实例方法。这样连缀方法就可以构成所谓的"期约连锁"

```javascript
let p = new Promise((resolve, reject) => {
  console.log("初始化拒绝期约");
  reject();
});

p.catch(() => {
  console.log("拒绝处理程序");
})
  .then(() => {
    console.log("解决处理程序");
  })
  .finally(() => {
    console.log("最终处理程序");
  });

// 初始化拒绝期约
// 拒绝处理程序
// 解决处理程序
// 最终处理程序
```

## 期约图

因为一个期约可以有任意多个处理程序，所以期约连锁可以构建有向非循环图的结构。这样每个期约都是图中的一个节点，而使用实例方法添加的处理处理程序则是有向顶点。因为图中的每个节点都会等待前一个节点落定，所以图的方向就是期约的解决或拒绝顺序

## Promise.all()和 Promise.race()

Promise 类提供两个将多个期约实例组合成一个期约的静态方法：Promise.all()和 Promise.race()。而合成后期约的行为取决于内部期约的行为。

### Promise.all

Promise.all()静态方法创建的期约会在一组期约全部解决之后再解决。这个静态方法接收一个可迭代对象，返回一个新期约

```javascript
// 合成的期约只会在每个包含的期约都解决之后才解决
let p = Promise.all([Promise.resolve(), new Promise((resolve) => setTimeout(resolve, 1000))]);

p.then(() => setTimeout(console.log, 0, p)); // Promise {<fulfilled>: Array(2)}
```

如果所有期约都成功解决，则合成期约的解决值就是包含期约解决值得数组。
如果有期约拒绝，则第一个拒绝的期约会将自己的理由作为合成期约的拒绝理由。之后在拒绝的期约不会影响最终期约的拒绝理由。不过，这并不影响所有包含期约正常的拒绝操作。合成的期约会静默处理所有包含期约的拒绝操作。

```javascript
// 虽然只有第一个期约的拒绝理由会进入拒绝处理程序
// 第二个期约的拒绝也会被静默处理，不会有错误跑掉
let p = Promise.all([Promise.reject(8), new Promise((resolve, reject) => setTimeout(reject, 1000))]);

p.catch((reason) => setTimeout(console.log, 0, reason));

// 没有未处理的错误
```

### Promise.race()

Promise.race()静态方法返回一个包装期约，是一组集合中最先解决或拒绝的期约的镜像。这个方法接收一个可迭代对象，返回一个新期约。Promise.race()不会对解决或拒绝的期约区别对待。无论是解决还是拒绝，只要是第一个落定的，Promise.race()就会包装其解决值或拒绝理由并返回新期约。

```javascript
// 解决先发生，超时的拒绝被忽略
let p = Promise.race([Promise.resolve(6), new Promise((resolve, reject) => setTimeout(reject, 1000))]);

p.then(() => setTimeout(console.log, 0, p)); // Promise {<fulfilled>: 6}
```

如果有一个期约拒绝，只要它是第一个落定的，就会成为拒绝合成期约的理由。之后在拒绝的期约不会影响最终期约的拒绝理由。不过，这并不影响所有包含期约正常的拒绝操作。与 Promise.all()类似，合成的期约会静默处理所有包含期约的拒绝操作。

```javascript
// 虽然只有第一个期约的拒绝理由会进入拒绝处理程序
// 第二个期约的拒绝也会被静默处理，不会有错误跑掉
let p = Promise.race([Promise.reject(4), new Promise((resolve, reject) => setTimeout(reject, 1000))]);

p.catch((reason) => setTimeout(console.log, 0, reason)); // 4

// 没有未处理的错误
```

## 串行期约合成

期约的另一个主要特性：异步产生值并将其传给处理程序。基于后续期约使用之前期约的返回值来串联期约是期约的基本功能。这很像函数合成，即将多个函数合成为一个函数

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

function compose(...fns) {
  return (x) => fns.reduce((promise, fn) => promise.then(fn), Promise.resolve(x));
}

let addTen = compose(addTwo, addThree, addFive);

addTen(9).then(console.log); // 19
```
