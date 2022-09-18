---
title: Typescript - 类型
mathjax: true
date: 2022-03-22 23:46:36
tags:
categories: [学习笔记, TypeScript]
description:
photos:
---


## 基本类型


### boolean、string、number


这三种数据类型与JavaScript中的同名类型一致，唯一不同的也是 TypeScript  与 JavaScript 的不同，Typescript 是静态类型语言，无法将某种数据类型的数据赋值给另一种数据类型的变量。如下，将 number 类型 17 赋值给 string 类型的kinsiyName，无法通过编译

<!--more-->

```typescript
let isDone: boolean = false
let kinsiyName: string = "Kinsiy"
let kinsiyAge: number = 24

kinsiyName = 17
// Type 'number' is not assignable to type 'string'.
```


### null、undefined


在JavaScript中一般用null表示空对象指针，undefined 表示值未进行初始化，它两均只有一个值。在TypeScript中一样，特殊点在于null与undefined能够赋给任意类型的变量。*可以配置 strictNullChecks 强制检查*


```typescript
let isDone: boolean = false
let age: number = 24

isDone = undefined
age = null
// ok
```


### any、void


any 表示任意类型，用于为在编程阶段还不清楚类型的变量指定一个类型。void 则相反，它表示没有任意类型。当一个函数没有返回值时，你通常会见到其返回值类型是 void


```typescript
let notSure: any = "kinsiy"
notSure = 24        
// ok

function foo(): void{
  console.log('return void')
}
```


### unknown、never


unknown表示一种**匿名**类型，某个值存在某种类型，只是我不知道它是什么类型，所有类型都可以赋值给unknown类型，unknown类型只能赋值给 any 或 unknown 类型本身，对unknown类型变量进行任何操作都无法通过编译。never则表示从未观察到的值，比如走不到终点的函数返回值，联合类型中无剩余类型的条件分支


```typescript
let notKnown: unknown = 5

let a: number = notKnown + 1
// Operator '+' cannot be applied to types 'unknown' and '1'.

let notSure: any = 5


notSure = notKnown

function fail(msg: string): never {        // 走不到终点的函数
  throw new Error(msg);
}

function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```


### 数组类型


指定某一类型的数组有两种方式，注意不要和元组搞混了


```typescript
let array1: number[] = [1, 2, 3]
let array2: Array<string> = ["1", "2", "3"]
```


### 元组类型


元组类型是另一种数组类型，它确切地知道它包含多少元素，以及在特定位置包含哪些类型。


```typescript
let tuple: [number, string] = [24, 'kinsiy']
tuple[1] = 17
// 类型不匹配
// Type 'number' is not assignable to type 'string'.

tuple[2] = 'lai'
// 越界类型为undefin
// Type '"lai"' is not assignable to type 'undefined'.
// 越界
// Tuple type '[number, string]' of length '2' has no element at index '2'.
```


### 枚举类型(Enum)


枚举类型是少部分不是JavaScript类型级拓展的特性之一。枚举允许我们定义一组命名常量 [enums](https://www.typescriptlang.org/docs/handbook/enums.html)


```typescript
// 数字枚举
enum Direction {
  Up = 1,        // 后续枚举值递增
  Down,
  Left,
  Right,
}

// 字符串枚举
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}

// 异构枚举 避免使用
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = "YES",
}
```


## 类型断言


有时候我们会比Typescript更清楚一个值的类型。可以使用类型断言去更改其类型。比如{% label primary@let kinsiyEle = document.getElementById("kinsiy") %} Typescript只知道 {% label primary@kinsiyEle %}  是一个{% label primary@HTMLElement %}类型的变量，但我们可能清楚的知道他是一个{% label primary@HTMLCanvasElement %}类型的元素。在使用{% label primary@HTMLCanvasElement %}的属性或方法时我们就需要用到类型断言。


```typescript
let kinsiyEle = document.getElementById("kinsiy")        
kinsiyEle.getContext('2d')
//Property 'getContext' does not exist on type 'HTMLElement'.

// 第一种方式使用 as
let kinsiyEle = document.getElementById("kinsiy") as HTMLCanvasElement
kinsiyEle.getContext('2d')
// 使用时再声明也是可以的
(kinsiyEle as HTMLCanvasElement).getContext('2d')


// 第二种方式使用尖括号
let kinsiyEle = <HTMLCanvasElement>document.getElementById("kinsiy");
kinsiyEle.getContext('2d')
(<HTMLCanvasElement>kinsiyEle).getContext('2d')


// 在使用 querySelector 等带泛型的选择器使用泛型也是可以的
let kinsiyEle = document.querySelector<HTMLCanvasElement>("#kinsiy");
kinsiyEle.getContext('2d')
```


类型断言只允许断言一个类型更具体或更不具体，可以理解为缩放两者(A, B)中必须存在$A\subset B$ 或者 $B \subset A$(硬要改也是可以的)。


```typescript
const kinsiy = 'kinsiy' as number
/**
Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
*/

class A{
  a: string
}
class B extends A{
  b: string
}
class C{
  c: string
}

let b = new B() as A        // ok 缩小
let a = new A() as B  // ok 放大

let _c = new C() as B        // error miss b

let c = (new C as any) as B        // 强制更改。any 与 unknown 是任何类型的超集
```


## 交叉类型


交叉类型可以将多个类型合并为一个类型


```typescript
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;

function draw(circle: ColorfulCircle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}
 
// okay
draw({ color: "blue", radius: 42 });
 
// oops
draw({ color: "red", raidus: 42 });

/**
Argument of type '{ color: string; raidus: number; }' is not assignable to parameter of type 'ColorfulCircle'.
  Object literal may only specify known properties, but 'raidus' does not exist in type 'ColorfulCircle'. Did you mean to write 'radius'?
*/
```


## 联合类型


联合类型表示一个值可以是几种类型之一


```typescript
let numAndStr: number | string

numAndStr = "kinsiy"        // ok
numAndStr = 24        // ok
numAndStr = true
//Type 'boolean' is not assignable to type 'string | number'.


interface Bird {
  fly: () => void
  layEggs: () => void
}

interface Fish {
  swim: () => void
  layEggs: () => void
}

function getSmallPet(): Fish | Bird {
  let result: Fish | Bird
  if(Math.random() > 0.5){
    result =  {
      fly: () => console.log("Bird fly"),
      layEggs: () => console.log("Bird layEggs")
    }
  }else{
    result =  {
      swim: () => console.log("Fish fly"),
      layEggs: () => console.log("Fish layEggs")
    }
  }
  return result
}

let pet = getSmallPet();
pet.layEggs(); // okay        
pet.swim();


/**
Property 'swim' does not exist on type 'Bird | Fish'.
  Property 'swim' does not exist on type 'Bird'.
*/
```


## 类型别名


可以使用{% label primary@type %}关键字对类型设置别名


```typescript
type Point = {
  x: number;
  y: number;
};
 
// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });


type ID = number | string;
```


{% note info no-icon%}


### type与interface的区别


一般来说用interface描述**数据结构**，用type描述**类型关系**


- type 可以声明基本类型别名，联合类型，元组等类型
- type 语句中还可以使用 typeof 获取实例的 类型进行赋值
- interface 能够声明合并


{% endnote %}


## 参考


[1\][TypeScriptLang.org Docs](https://www.typescriptlang.org/docs/)


[2\][Tslang.cn Docs](https://www.tslang.cn/docs/home.html)


[3\][掘金. Typescript 中的 interface 和 type 到底有什么区别](https://juejin.cn/post/6844903749501059085)
