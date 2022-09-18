---
title: Typescript - Function
mathjax: false
date: 2022-04-01 22:07:24
tags:
categories: [学习笔记, TypeScript]
description:
photos:
---

本文学习如何在Ts中声明函数类型

{% note info %}

学习[JavaScript - Function](https://kinsiy.github.io/Javascript-Function/)

本文参考Typescript官网[Docs - Handbook - More onFunction](https://www.typescriptlang.org/docs/handbook/2/functions.html)自翻译而来, 仅供个人参考学习使用.一切请以官网为准!

{% endnote %}

## 函数声明

### 函数类型表达式

函数类型表达式是Ts中最简单的声明函数的方法.它看起来和ES6中的箭头函数很是相像.

```typescript
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}
 
function printToConsole(s: string) {
  console.log(s);
}
 
greeter(printToConsole);

// 使用类型别名
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

<!--more-->

`(a: string) => void`的意思是, 一个拥有一个string类型的参数a,不返回任何值的函数.

### 调用签名

我们知道在Js中函数是一个对象, 可以像下面这样往函数中添加属性. (当然我们很少这么做,只是使用Function原有的属性{% label primary@name %},{% label primary@length %}等)

```javascript
function foo(){
  console.log(foo.kinsiy)  
}

foo.kinsiy = "qinglai"

foo()
```

这意味着我们需要声明的是一个可以调用的对象.这时候函数表达式的类型声明方法无法满足我们的要求,需要用到**调用签名**这一声明方法,这种方式声明也很简单,只需把函数类型表达式中的箭头 (=>) 换成冒号 (:) 即可

```typescript
type DescribableFunction = {
  description: string
  (someArg: number): boolean  // "=>" to ":"
}

function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6))
}
```

### 构造签名

在Js中,还可以使用{% label primary@new %}操作符调用函数.这时候我们可以在调用签名前添加一个{% label primary@new %},便可以声明构造函数的目的.对于调用时{% label primary@new %}操作符是可选的函数(对象),比如 {% label primary@Date %}, 需要同时声明调用签名,与构造签名

```typescript
type SomeConstructor = {
  new (s: string): SomeObject  // new
}

function fn(ctor: SomeConstructor) {
  return new ctor("hello")
}

// 同时声明构造签名与调用签名
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

## 泛型函数

通常我们会写一些,输入类型与输出类型相关或者两个参数之间存在某种关联的函数.举个栗子,返回任意数组中的第一个元素

```typescript
function firstElement(arr: any[]) {
  return arr[0]
}
```

函数很简单,我们希望函数能够根据我们输入的类型自动得出返回值的类型,当我输入`string[]`类型时,返回值就是{% label primary@string %}类型的,其他类型同理.但实际上由于是任意数组,无法事先知道参数的类型,目前为止只能使用`any[]`类型,这时候返回类型自然也就是{% label primary@any %}类型.不满足我们的要求! 这就轮到**泛型函数**登场了

```typescript
function firstElement<Type>(arr: Type[]): Type {
  return arr[0];
}

const s = firstElement(["a", "b", "c"]) // const s: string
const n = firstElement([1, 2, 3]) // const n: number
const u = firstElement([]) // const u: never
```

可以看到,我们在函数名后方声明了类型{% label primary@Type %},并在参数中使用`Type[]`对arr进行注解,函数返回类型也是{% label primary@Type %}类型.这样我们就得到了一个通用的类型函数(**泛型函数**).函数的返回类型会根据参数类型自动推断出来.

{% note  info %}

注意: 不需要主动去指定{% label primary@Type %}.Ts会自动推断出正确的类型

```typescript
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func)
}

// const parsed: number[]
// (parameter) n: string
const parsed = map(["1", "2", "3"], (n) => parseInt(n))
```

{% endnote %}

### 泛型约束

当函数需要处理不同类型的对象时,在函数中我们可能只需要读/写特定的属性.只要某对象存在这些属性,就应该可以成为函数的合法参数.举例来说, 比较两个参数的{% label primary@length: number %}属性, 只要参数存在{% label primary@length: number %}属性即可比较,无其他约束.

```typescript
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a
  } else {
    return b
  }
}
 
const longerArray = longest([1, 2], [1, 2, 3]) // const longerArray: number[]
const longerString = longest("alice", "bob") // const longerString: "alice" | "bob"
const notOK = longest(10, 100) // Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
```

另外对于返回泛型{% label primary@Type %}的函数, 返回值满足约束条件并不一定能满足类型{% label primary@Type %},{% label primary@Type %}是由入参决定的,{% label primary@Type %}是“约束”的超集.

```typescript
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum }; // Error "约束"不能满足其超集{% label primary@Type %}
  }
}

/**
Type '{ length: number; }' is not assignable to type 'Type'.
  '{ length: number; }' is assignable to the constraint of type 'Type', but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.
*/
```

### 指定类型

对于泛型函数,若指定多个入参均为泛型{% label primary@Type %},这些入参的类型应完全一样.对于不一样的类型,可以在调用函数时指定通用泛型

```typescript
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2)
}

const arr = combine([1, 2, 3], ["hello"]) // Error: Type 'string' is not assignable to type 'number'.

const _arr = combine<string | number>([1, 2, 3], ["hello"]) // const arr: (string | number)[]
```

### 最佳实践

1. 尽可能的使用类型参数本身,而不是“约束”拓展

   >  When possible, use the type parameter itself rather than constraining it

   ```typescript
   function firstElement1<Type>(arr: Type[]) {
     return arr[0]
   }
   
   // don't do this
   function firstElement2<Type extends any[]>(arr: Type) {
     return arr[0]
   }
   
   const a = firstElement1([1, 2, 3]) // const a: number
   const b = firstElement2([1, 2, 3]) // const b: any
   ```

   Ts必须在函数调用之前推断出它的返回类型,而不是在函数调用的时候动态决定.

2. 尽可能的少用泛型

   > Always use as few type parameters as possible

   ```typescript
   function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
     return arr.filter(func);
   }
    
   function filter2<Type, Func extends (arg: Type) => boolean>(
     arr: Type[],
     func: Func
   ): Type[] {
     return arr.filter(func);
   }
   ```

   对比{% label primary@filter1 %}与{% label primary@filter2 %},不难发现两种写法,在不指定类型其实是没差别的,但当指定类型时,{% label primary@filter2 %}却需要多指定一个泛型.多此一举

3. 泛型{% label primary@Type %}至少应该出现两次

   >  If a type parameter only appears in one location, strongly reconsider if you actually need it

   ```typescript
   function greet<Str extends string>(s: Str) {
     console.log("Hello, " + s);
   }
    
   greet("world");
   
   // 直接使用"约束类型"即可
   function greet(s: string) {
     console.log("Hello, " + s);
   }
   ```

   对于只出现一次的类型,完全没有必要使用泛型.记住,泛型用于关联多个值的类型

## 可选参数

在参数前使用{% label primary@? %}修饰,可以使参数变为可选参数.对于未指定默认值的可选参数,其类型永远都是声明类型本身与{% label primary@undefined %}的联合类型.因为在Js中未赋值的参数的值就是{% label primary@undefined %}.

```typescript
function f(x?: number){
  console.log(x) // (parameter) x: number | undefined
}

f()  // underfined
f(10) // 10

function g(x = 10){
  console.log(x) // (parameter) x: number
}
```

与ES6一样,可以对参数设定默认值,设定默认值后,改参数变为可选参数,但不传该值或传入的值为undefined时,参数获得改默认值,并且值的类型为其本身类型,不会存在{% label primary@undefined %}

### 回调函数

注意,尽量不要在回调函数中使用可选参数.除非你打算在不传递参数的情况下进行回调. 具有较少参数(相同类型)的函数总是可以代替具有更多参数的函数.所以在回调中定义多个参数即可,传入**回调函数**定义的参数可以比**定义的函数**定义的参数的参数少

```typescript
function myForEach(arr: any[], callback: (arg: any, index: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}

myForEach([1, 2, 3], (a) => console.log(a))  // ok
myForEach([1, 2, 3], (a, i) => console.log(a, i)) // ok
```

## 函数重载

针对Js中同名函数可以传入不同数量/类型的用法,除了可选/默认参数外.Ts中还可以使用函数重载实现

```typescript
function makeDate(timestamp: number): Date
function makeDate(m: number, d: number, y: number): Date
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d)
  } else {
    return new Date(mOrTimestamp)
  }
}
const d1 = makeDate(12345678)
const d2 = makeDate(5, 5, 5)
const d3 = makeDate(1, 3) // Error

//No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```

{% note  warning %}

### 注意

需要区分**重载函数**和**实现函数**, 重载函数指没有函数体的函数声明,实现函数则指拥有函数体的函数.实现函数要求写在所有重载函数后,并且实现函数是兼容所有重载函数的.但是实现函数对**外部**不可见,这里外部指不能直接调用实现函数,只能通过重载函数签名对函数进行调用.举例来说

```typescript
function fn(x: string): void
function fn() {
  // ...
}

fn() // Error: Expected to be able to call with zero arguments
```

这里直接对实现函数进行调用,无法通过编译

{% endnote %}

### 最佳实践

尽可能使用联合类型而不是函数重载.

> Always prefer parameters with union types instead of overloads when possible

举例来说

```typescript
function len(s: string): number
function len(arr: any[]): number
function len(x: any) {
  return x.length
}

len("") // OK
len([0]) // OK
len(Math.random() > 0.5 ? "hello" : [0]) // Error

/**
No overload matches this call.
  Overload 1 of 2, '(s: string): number', gave the following error.
    Argument of type 'number[] | "hello"' is not assignable to parameter of type 'string'.
      Type 'number[]' is not assignable to type 'string'.
  Overload 2 of 2, '(arr: any[]): number', gave the following error.
    Argument of type 'number[] | "hello"' is not assignable to parameter of type 'any[]'.
      Type 'string' is not assignable to type 'any[]'.
*/

function len(x: any[] | string) {
  return x.length;
}

len("") // OK
len([0]) // OK
len(Math.random() > 0.5 ? "hello" : [0]) // ok
```

{% note info %}

对于重载,我个人理解,在Js中与可选参数差别不大,重载能做到的可选参数一样可以做到.对于不同类型,Ts中则是建议使用联合类型.

那么在实践中就是尽量不要使用重载.

- 对相同数量参数的函数,尽量使用联合类型

- 对不同数量参数的函数,用可选参数代替

- 对参数数量/类型均不同的函数,结合联合类型与可选参数实现.

{% endnote %}

## Void

对于**拥有类型为返回void的函数类型**的函数,Ts不会禁止该函数返回值,但返回值赋值给变量时,变量获取到的类型始终是void.举例来说

```typescript
type voidFunc = () => void
 
const f1: voidFunc = () => true // ok

const v1 = f1() // const v1: void
```

这对于定义回调很有用,使得我们可以在形如`Array.prototype.forEach{% label primary@$1 %}Array.prototype.push`这样返回函数的值

```typescript
const src = [1, 2, 3];
const dst = [0];
 
src.forEach((el) => dst.push(el));
```

但当我们声明一个函数拥有void返回类型时,则禁止返回任何值

```typescript
const f1 = (): void => true // Type 'boolean' is not assignable to type 'void'.

function f2(): void {
  return true // Type 'boolean' is not assignable to type 'void'.
}
 
const f3 = function (): void {
  return true // Type 'boolean' is not assignable to type 'void'.
}
```

## 参考

[1\][Typescriptlang.org. More onFunction](https://www.typescriptlang.org/docs/handbook/2/functions.html)
