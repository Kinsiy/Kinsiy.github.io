---
title: 响应系统 - 原始值
mathjax: false
date: 2022-09-27 19:46:34
tags: 响应式
categories:
 - ['Vue', 'Vue.js设计与实现']
description:
photos:
---

{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第六章 - 原始值的响应式方案 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

原始值指的是Boolean, Number, BigInt, String, Symbol, undefined 和 null 等类型的值.在 Javascript 中, 原始值是按值传递的,而非按引用传递. 形参和实参之间没有引用关系, 它们是两个完全独立的值, 对形参的修改不会影响实参. 另外, Javascript 中的Proxy 无法提供对原始值的代理, 因此想要将原始值变成响应式数据, 就必须对其做一层包裹, 也就是{% label info@ref %}

<!-- more -->

## 引入ref的概念

由于 Proxy 的代理目标必须是非原始值, 没有任何手段拦截对原始值的操作.

对于这个问题,需要使用一个非原始值去“包裹”原始值. 并且需要区分原始值的包裹对象与非原始值的响应式对象

```js
function ref(val) {
  const wrapper = {
    value: val
  }

  // 定义不可枚举且不可写的属性, 用于判断数据是否为 ref
  Object.defineProperty(wrapper, '__v_isRef', {
    value: true
  })

  return reactive(wrapper)
}

function isRef(val) {
  return val.__v_isRef === true
}
```

## 响应丢失问题

对响应式数据使用{% label danger@... %}展开时, 会导致响应丢失. 可以使用类似 {% label info@ref %}的结构,通过访问器属性读取相应的响应式数据, 以触发响应

```js
function toRefs(obj) {
  const ret = {}
  for (const key in obj) {
    ret[key] = toRef(obj, key)
  }
  return ret
}

function toRef(obj, key) {
  const wrapper = {
    get value() {
      return obj[key]
    },
    set value(val) {
      obj[key] = val
    }
  }

  Object.defineProperty(wrapper, '__v_isRef', {
    value: true
  })

  return wrapper
}
```

## 自动脱ref

上述{% label info@toRefs %}函数的确解决了响应丢失问题, 但同时也带来了新的问题. *必须使用value访问属性值*, 这会增加用户的心智负担. 这就提供需要自动脱ref 的能力. 

所谓自动脱 ref , 指的是属性的访问行为, 即如果读取的属性是一个 ref, 则直接将该 ref 对应的value值返回

```js
function proxyRefs(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const value = Reflect.get(target, key, receiver)
      return value.__v_isRef ? value.value : value	// 判断 __v_isRef
    },
    set(target, key, newValue, receiver) {
      const value = target[key]
      if (value.__v_isRef) {
        value.value = newValue
        return true
      }
      return Reflect.set(target, key, newValue, receiver)
    }
  })
}
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)
