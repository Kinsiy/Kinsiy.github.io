---
title: Typescript - 类型控制
mathjax: false
date: 2022-04-18 23:13:30
tags:
categories:
description:
photos:
---

{% note info%}

本文参考Typescript官网[Docs - Handbook - Types Manipulation 系列](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)自翻译而来, 仅供个人参考学习使用.一切请以官网为准!

{% endnote %}

Typescript 的类型系统非常强大,它允许用一种类型来表达另一种类型.最显然的例子是泛型,通过声明泛型对象/方法,可以使用多种类型去表达此对象/方法.但不仅仅只是泛型,可以使用多种方法利用一种类型来创造/表达另一种类型

## 泛型

使用泛型能够创建一个在多种类型而不是单一类型上工作的的组件.在[Typescript - Function](https://kinsiy.github.io/Typescript-Function/#%E6%B3%9B%E5%9E%8B%E5%87%BD%E6%95%B0) 和 [Typescript - Classes](https://kinsiy.github.io/Typescript-Classes/#%E6%B3%9B%E5%9E%8B%E7%B1%BB)中我们分别说明了如何去声明名一个泛型函数及泛型类.特别是泛型函数部分,有比较详细的说明为何要使用泛型.在这里主要补充几点

### 泛型约束

泛型约束除在泛型函数中直接对参数类型属性进行约束外,还可以依据泛型{% label primary@Type %}创建出另一种类型,实现响应键值/属性的约束.

```typescript
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
 
let x = { a: 1, b: 2, c: 3, d: 4 };
 
getProperty(x, "a");
getProperty(x, "m"); // Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
```

 这里的{% label primary@Key %}类型是根据泛型{% label primary@Type %}创造出来的

### keyof 操作符

正如上面泛型约束的示例所示,{% label primary@keyof %} 操作符根据给出的{% label primary@object %}生成一个由其键值联合的字符串或数字类型.上方{% label primary@key %}类型就是{% label primary@x %}的键值的联合类型.

### typeof 操作符
