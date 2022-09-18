---
title: TypeScript - 类型推断
mathjax: false
date: 2022-03-25 20:09:08
tags:
categories: [学习笔记, TypeScript]
description:
photos:
---

{% note info%}

本文参考Typescript官网[Docs - Handbook - Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)自翻译而来, 仅供个人参考学习使用.一切请以官网为准!

{% endnote %}

考虑下面这段代码，能否通过Ts编译？


```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding.slice(0) + input;
}
```


<!--more-->


答案是可以。为什么？我们明明指定 padding 的类型是 number | string ，而 number 类型是没有 slice 方法的，却能通过编译？关键在于{% label primary@typeof padding === "number" %}这一句，这一特殊的检查被TypeScript称作**类型保护(type guards)**。


在{% label primary@typeof padding === "number" %} 分支中，padding 的类型缩小至 number ，并发现最后一句{% label primary@return padding.slice(0) + input; %}是不可到达的，则在分支外去掉 padding 的 number 类型。在调用 slice 方法时， padding 的类型已经被缩小至 string ，自然也就能够通过编译。


## typeof 操作符


正如上面 padLeft 所示，可以通过 typeof 类型保护缩小类型范围。TypeScript可以理解 typeof 返回的值，对类型范围进行缩小。 typeof 会返回下列结果之一


- "string"
- "number"
- "bigint"
- "boolean"
- "symbol"
- "undefined"
- "object"
- "function"


## 真值推断


typeof  在检查基本类型时很好用，但当检查包含 null 得类型时，由于在 Javascript 中 typeof null 会返回"object"。如下，单检查  typeof strs === "object" 并不能排除 null , 这时候就需要利用真值来进行类型缩小


```typescript
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
      // Object is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```


利用 null 、undefined等会转化为假值的特性([Boolean 类型](https://kinsiy.github.io/JavaScript-Basics/#Boolean-%E7%B1%BB%E5%9E%8B))，很容易就可以排除 null 等类型


```typescript
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {        // ok
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```


## 相等性推断


TypeScript 还可以使用相等性来进行类型缩小，如 \=\=\=、!\=\=、\=\=、!\= 操作符


```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase();
    y.toLowerCase();
  } else {
    console.log(x); //(parameter) x: string | number
    console.log(y); //(parameter) y: string | boolean
  }
}
```


对于上述代码，由于只有 x y 都是 string 类型时才存在严格相等的可能，TypeScript 能够依据这个信息推断出在if 这个分支上 x y的类型均是 string 类型


另外由于在 JavaScript中 undefined 和 null 是**相等**(不是严格相等哦)的，我们可以利用这个特性一次去除两个类型，相应的只需要两者之一时需要使用严格相等


```typescript
interface Container {
  value: number | null | undefined;
}
 
function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value); // (property) Container.value: number
 
    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```


## in 运算符


我们知道在JavaScript中，如果指定的属性在指定的对象或其原型链中，则 **{% label primary@in %} 运算符** 返回 true。在Ts中也可以利用这个特性来缩小类型范围。举例来说


```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
 
  return animal.fly();
}
```


形如 {% label primary@"value" in x %}， value是一段字符串，x是某种联合类型，对于真值，x 缩小为包含"value"属性的**联合类型**(不一定缩小至唯一，因为 x 中不一定只有一种类型包含 "value" 属性)，反之，假值这缩小为不含"value"属性的联合属性。若value 是某类型的可选属性，这两分支中均包含 该类型


```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };
 
function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal; // (parameter) animal: Fish | Human
  } else {
    animal; // (parameter) animal: Bird | Human
  }
}
```


## instanceof 运算符


typeof、in 都可以用来推断类型，instanceof自然也不会缺席!


```typescript
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString()); // (parameter) x: Date
  } else {
    console.log(x.toUpperCase()); // (parameter) x: string
  }
}
```


## 赋值推断


当赋值给变量时，Ts会根据值的类型对变量的类型进行推断


```typescript
let x = Math.random() < 0.5 ? 10 : "hello world!"; // let x: string | number


x = 1;
console.log(x); // let x: number


x = "goodbye!"; 
console.log(x); // let x: string


// 赋值判断的是最初的类型，这就是为什么当第一次赋值推断出number类型后，仍可以赋值string类型的"goodbye"
x = true 
// Type 'boolean' is not assignable to type 'string | number'.
```


## 控制流分析


正如开篇的例子一般，Ts判断{% label primary@if %}分支外能否到达来推断变量的类型。Ts可以通过分析{% label primary@if %}、{% label primary@while %}等控制流进行类型推断。变量可以通过一遍又一遍的控制流分析在程序的不同的位置展现出不同的类型


```typescript
function example() {
  let x: string | number | boolean;
 
  x = Math.random() < 0.5;
 
  console.log(x); // let x: boolean
 
  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x); // let x: string
  } else {
    x = 100;
    console.log(x); //let x: number
  }
  return x; // let x: string | number
}
```


## 类型谓语


到目前为止，我们已经使用现有的 JavaScript 结构来处理类型推断，但是有时我们希望可以更直接地控制类型在整个代码中的变化方式。这时候就需要文明自定义类型保护


要自定义类型保护，我们只需要定义一个返回类型为**类型谓词** 的函数：


```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}


const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];
 
// The predicate may need repeating for more complex examples
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```


## 可辨识联合


目前为止我们主要都是对简单的基本类型进行推断。考虑下面这样一个结构，我们有一个 Shape 接口，对于 circle,  必有radius，对于 square 则必有 sideLength。 


```typescript
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```


写一个方法求 Shape 的面积


```typescript
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
                // Object is possibly 'undefined'.
  }
}
```


尽管我们用 kind 进行了判断，分支内 Shape 必是circle，我们知道radius必然存在。但Ts无法知道这个信息，strictNullChecks下会给出我们 radius 可能不存在的错误。对于这种我们比Ts更清楚的情况，可以使用**非空断言(添加!后缀)**告诉Ts某个值必然存在。


```typescript
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2; //ok
  }
}
```


这样虽然不报错了，但在"circle"分支内还是可以选择sideLength属性并且后续每次使用radius属性仍然要继续使用非空断言。还是会有出错的可能！这就轮到到可辨识类型登场了


> 你可以合并单例类型，联合类型，类型保护和类型别名来创建一个叫做 *可辨识联合*的高级模式，它也称做 *标签联合*或 *代数数据类型*。 可辨识联合在函数式编程很有用处。 一些语言会自动地为你辨识联合；而TypeScript则基于已有的JavaScript模式。 它具有3个要素：
>
> 1. 具有普通的单例类型属性— *可辨识的特征*。
> 2. 一个类型别名包含了那些类型的联合— *联合*。
> 3. 此属性上的类型保护


```typescript
// 重新定义Shape
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}
 
type Shape = Circle | Square;


// 不使用任何非空断言重新实现getArea()
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // (parameter) shape: Circle
    case "square":
      return shape.sideLength ** 2; // (parameter) shape: Square
    default:
      return shape; //(parameter) shape: never
  }
}
```


可以看到，在每个case分支中，联合类型shape的类型都能推断正确，属性提示也没问题。对于 default 分支，由于前两个分支已经将 Shape 穷举完毕。default 分支内将得到一个 never 类型

## 参考

[1\][Typescriptlang.org. Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)

[2\][Tslang.cn. 高级类型](https://www.tslang.cn/docs/handbook/advanced-types.html)
