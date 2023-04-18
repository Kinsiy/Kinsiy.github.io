---
title: 工厂模式比较
mathjax: false
date: 2023-04-08 11:57:32
tags: factory-method
categories: [设计模式,创建型模式]
description:
photos:
---

## 工厂

工厂是一个含义模糊的术语,表示可以创建一些东西的函数,方法或类.最常见的情况下,工厂创建的是对象.但是它们也可以 创建文件和数据库记录等其他东西.

例如,下面这些东西都可以非正式地被称为“工厂”

- 创建程序GUI的函数或方法
- 创建用户的类
- 以特定方式调用类构造函数的静态方法
- 一种创建型设计模式

当某人说到“工厂”这个词时,其具体含义通常可以根据上下文来确定.

<!--more-->

## 构建方法

构造方法只是构造函数调用的封装器.它可能只是一个能更好地表达以图的名称.此外,它可以让你的代码独立与构造函数的改动,甚至还可以包含一些特殊的逻辑,返回已有对象而不是创建新对象

```typescript
class FooNumber {
  private value: number
  
  constructor(value: number) {
    this.value = value
  }
  
  // 构建方法
  next() {
    return new FooNumber(this.value + 1)
  }
}
```

## 静态构建方法

静态构建方法是被声明为{% label info@static %}的构建方法.

当静态构建方法返回一个新对象时,它就成为了构造函数的代替品

{% note danger %}

构建方法在 《重构和模式》中被定义为“创建对象的方法”. 但不是工厂方法. 工厂方法是一种依赖于继承的设计模式.

同样的, 静态构建方法也不是“静态工厂方法”

{% endnote %}

## 简单工厂模式

简单工厂模式描述了一个类,它拥有一个包含大量条件语句的构建方法,可根据方法的参数来选择对何种产品进行初始化并将其返回.

人们通常会将简单工厂与普通的工厂或其它创建型设计模式混淆.在绝大多数情况下,简单工厂是引入工厂方法或抽象工厂模式时的一个中间步骤

简单工厂通常没有子类.但当从一个简单工厂中抽取子类后,它看上去就会更新经典的工厂方法模式了

另外,将一个简单工厂声明为{% label info@abstract %}类型,它并不会神奇地变成抽象工厂模式

```ts
enum USER_TYPE {
  USER,
  CUSTOMER,
  ADMIN,
}

class UserFactory {
  static create(type: USER_TYPE) {
    switch (type) {
      case USER_TYPE.USER:
        return new User();
      case USER_TYPE.CUSTOMER:
        return new Customer();
      case USER_TYPE.ADMIN:
        return new Admin();
      default:
        throw new Error('错误的用户类型')
    }
  }
}
```

## 工厂方法模式

工厂方法是一种创建型设计模式,其在父类中提供一个创建对象的方法,允许子类决定实例化对象的类型.

如果在基类及其拓展的子类中都有一个构建方法的话,那它可能就是工厂方法

```ts
// 接口或者创建者基类
interface ProductFactory {
  createProduct(name: string): Product; // 注意返回值, 为产品实现的接口
}

// 具体创建者将重写工厂方法以改变其所返回的产品类型
class BasicProductFactory implements ProductFactory {
  createProduct(name: string): Product {
    return new BasicProduct(name, 10);
  }
}

// 另一种产品工厂
class DiscountedProductFactory implements ProductFactory {
  createProduct(name: string): Product {
    return new DiscountedProduct(name, 10, 0.2);
  }
}
```

## 抽象工厂模式

抽象工厂是一种创建型设计模式,它能创建一系列相关或相互依赖的对象,而无需指定其具体类

什么是{% label info@“系列对象” %}? 

例如一组对象: {% label info@运输工具 %} + {% label info@引擎 %} + {% label info@控制器 %}. 它可能会有几个变体

-  {% label info@汽车 %} + {% label info@内燃机 %} + {% label info@方向盘 %}
-  {% label info@飞机 %} + {% label info@喷气式发动机 %} + {% label info@操纵杆 %}

如果程序中并不涉及产品系列的话,那就不需要抽象工厂

## 参考

[Refactoringguru.cn 工厂模式比较](https://refactoringguru.cn/design-patterns/factory-comparison)
