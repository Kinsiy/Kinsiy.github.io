---
title: 面向对象建模-类图关系说明
mathjax: false
date: 2023-05-30 21:47:52
tags:
categories:
description:
photos:
---

## 泛化

{% label info@空心三角和实线，空心三角指向父类 %}

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/6d3d721c.png)

{% note info%}

**泛化是指继承吗**

继承是面向对象编程中的一个概念，它允许一个类继承另一个类的属性和方法。通过继承，子类可以从父类那里继承一些通用的特性，从而避免重复编写代码。子类可以通过继承获得父类的属性和方法，并且可以添加自己的特定功能或修改继承的功能。

泛化是面向对象建模中的一个概念，它表示从具体的事物中提取出共同的特征和概念，形成一个更为抽象的概念或类别。通过泛化，可以将一组类或对象抽象为一个更一般化的类或对象，以便更好地理解和组织它们之间的关系。

因此，泛化和继承是两个独立但相关的概念。继承是实现泛化的一种方式，通过继承可以实现将共同的属性和方法从一个类传递给另一个类。泛化更加宽泛，可以指代更一般化的抽象过程，而不仅仅局限于继承。

{% endnote %}



<!--more-->

## 实现

{% label info@空心三角和虚线，空心三角指向接口 %}

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/dd75800e.png)

## 关联

{% label info@普通箭头和实心线，指向被拥有者 %}

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/80e7f62c.png)

## 聚合

{% label info@空心菱形、普通箭头（或没有箭头）和实心线，菱形挨着整体，箭头指向部分 %}

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/28d17034.png)

## 组合

{% label info@实心菱形、普通箭头（或没有箭头）和实心线，菱形挨着整体，箭头指向部分 %}

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/d57a5c32.png)

## 依赖

{% label info@普通箭头和虚线 %}

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/0267b672.png)

## 一张图

![image-20230530224326526](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20230530224326526.png)

## 参考

[OpenAI ChatGPT](https://chat.openai.com/)

