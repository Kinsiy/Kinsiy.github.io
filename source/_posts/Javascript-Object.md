---
title: 集合引用类型(Object)
author: Kinsiy
avatar: 'https://cdn.jsdelivr.net/gh/Kinsiy/cdn/img/custom/avatar.jpg'
authorLink: 'https://kinsiy.github.io'
authorAbout: I'm True
authorDesc: I'm True
categories: SKILL
comments: true
date: 2021-01-24 01:44:54
tags: [Notes,Javascript]
keywords:
description: Object 对象类型的使用
photos:
---
Object是ECMAScript中最常用的类型之一。虽然Object的实例没有多少功能，但很适合存储和在应用程序间交换数据。
```javascript
/* new Object() */
let person = new Object();
person.name = "Kinsiy";
person.age = 23;

/* 对象字面量 */
let people = {
    name: "Meng",
    age: 24,
    5: true
}

/* 存取 */
console.log(people.name);       // Meng
console.log(people["age"]);     // 24

let propertyName = "name";
console.log(people[propertyName]);  // Meng

people["first name"] = "ZHAO";
console.log(people["first name"]);  // ZHAO
```
更多内容待补充！