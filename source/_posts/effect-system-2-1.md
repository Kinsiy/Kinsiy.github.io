---
title: 响应系统 - 非原始值 - 常规对象
mathjax: false
date: 2022-09-24 19:25:09
tags: 响应式
categories:
 - ['Vue', 'Vue.js设计与实现']
description:
photos:
---
{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第五章 - 非原始值的响应式方案 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

## Proxy & Reflect

{% note info %}

**Javascript - Proxy**

[代理基础](/Javascript-Proxy-1/) /[捕获器与反射](/Javascript-Proxy-2/)/ [代理模式](/Javascript-Proxy-3/)

{% endnote %}

代理指的是对一个对象{%label primary@基本语义%}的代理。它允许我们{% label primary@拦截 %}并{% label primary@重新定义 %}对一个对象的基本操作

<!-- more -->

**基本操作:**  读取、设置属性值、调用函数

**复合操作:** 非基本操作

```js
const obj = {
  foo: 1,
  get bar() {
    return this.foo
  }
}

const p = new Proxy(obj, {
  get(target, key, receiver) {
    track(target, key)
    // 使用Reflect.get 返回读取到属性值
    return Reflect.get(target, key, receiver)
  },
  //  ...
})

effect(() => console.log(p.bar))

p.foo++
```

注意上面使用{% label primary@Reflect %} 以达到收集 {% label danger@obj.foo %}依赖. 直接使用{% label info@target[key] %}无法收集



## Javascript对象及Proxy的工作原理

**{% label primary@常规对象 %}**: 

- 对于 *{% label info@ECMA-262. 6.1.7.3 Invariants of the Essential Internal Methods %}* 列出的除 [[Call]]、[[Construct]]外的内部方法,必须使用ECMA规范 10.1.x 节给出的定义实现
- 对于内部方法[[Call]], 必须使用ECMA规范 10.2.1 节给出的定义实现
- 对于内部方法[[Construct]], 必须使用ECMA规范 10.2.2 节给出的定义实现

**{% label primary@异质对象 %}**: 非常规对象



{% note info %}

**ECMA - 262**

[12th edition, June 2021](https://www.ecma-international.org/wp-content/uploads/ECMA-262_12th_edition_june_2021.pdf)

{% endnote %}



创建代理对象时指定的拦截函数,实际上是用来自定义代理对象本身的内部方法和行为的,而不是用来指定被代理对象的内部方法和行为的

Proxy对象部署的所有内部方法以及用来自定义内部方法和行为的拦截函数名字 *{% label info@ECMA-262. 10.5 Proxy Object Internal Methods and Internal Slots %}* 

| Internal Method       | Handler Method               |
| --------------------- | ---------------------------- |
| [[GetPrototypeOf]]    | **getPrototypeOf**           |
| [[SetPrototypeOf]]    | **setPrototypeOf**           |
| [[IsExtensible]]      | **isExtensible**             |
| [[PreventExtensions]] | **preventExtensions**        |
| [[GetOwnProperty]]    | **getOwnPropertyDescriptor** |
| [[DefineOwnProperty]] | **defineProperty**           |
| [[HasProperty]]       | **has**                      |
| [[Get]]               | **get**                      |
| [[Set]]               | **set**                      |
| [[Delete]]            | **deleteProperty**           |
| [[OwnPropertyKeys]]   | **ownKeys**                  |
| [[Call]]              | **apply**                    |
| [[Construct]]         | **construct**                |

## 如何代理Object

这里直接贴出代理代码, 至于原因请阅读原文或者ECMA-262

### in

{% label primary@[[HasProperty]] %}

```js
const obj = { foo: 1 }
const p = new Proxy(obj, {
  has(target, key){
    tarck(target, key)
    return Reflect.has(target, key)
  }
})
```

### for ... in

{% label primary@ownKeys %}

```js
const obj = { foo: 1 }
const ITERATE_KEY = Symbol

const p = new Proxy(obj, {
  ownKeys(target){
    // 将副作用函数与 ITERATE_KEY 关联
    tarck(target, ITERATE_KEY)
    return Reflect.ownKeys(target)
  }
})
```

当为对象添加新属性时,会对{% label primary@for...in %}循环产生影响,所以需要触发与{% label primary@ITERATE_KEY %}相关联的副作用函数重新执行.

由于添加新属性与修改已有的属性值的基于语义都是{% label info@[[Set]] %}, 为避免修改已有属性值时触发副作用.需要区分这两种操作

```js
const ITERATE_KEY = Symbol()

const p = new Proxy(obj, {
  // ...
  // 拦截设置操作
  set(target, key, newVal, receiver) {
    const type = Object.prototype.hasOwnProperty.call(target, key) ? 'SET' : 'ADD' // 如果属性不存在，则说明是在添加新的属性，否则是设置已存在的属性
    const res = Reflect.set(target, key, newVal, receiver) // 设置属性值
    trigger(target, key, type) // 将 type 作为第三个参数传递给 trigger 函数
    return res
  }
  // ...
})

function trigger(target, key, type) {
  const depsMap = bucket.get(target)
  if (!depsMap) return
  const effects = depsMap.get(key)
  
  const iterateEffects = depsMap.get(ITERATE_KEY)
  
  // ...
  
  if (type === 'ADD') {
    iterateEffects && iterateEffects.forEach(effectFn => {
      if (effectFn !== activeEffect) {
        effectsToRun.add(effectFn)
      }
    })
  }
	
  // ...
}
```

### delete

{% label primary@deleteProperty %}

```js

const p = new Proxy(obj, {
  // ...
  deleteProperty(target, key) {
    const hadKey = Object.prototype.hasOwnProperty.call(target, key)
    const res = Reflect.deleteProperty(target, key)

    if (res && hadKey) {
      trigger(target, key, 'DELETE')
    }

    return res
  }
})
```

{% label primary@delete %}同样影响{% label info@for...in %}循环

```js
function trigger(target, key, type) {

  // ...
  
  if (type === 'ADD' || type === 'DELETE') {
    iterateEffects && iterateEffects.forEach(effectFn => {
      if (effectFn !== activeEffect) {
        effectsToRun.add(effectFn)
      }
    })
  }
	
  // ...
}
```

## 合理地触发响应

考虑

- 多次设置相同的值
- 原型链上响应式对象

```js
  const obj = { foo: 1 }
  const p = new Proxy(obj, { /** ... */ })
  
  effect(() => console.log(p.foo))
  p.foo = 1
  
  /**
   * 创建响应式数据
   */
  function reactive(obj){
    return new Proxy(obj, { /** ... */ })
  }
  
  const _obj = {}
  const proto = { bar: 1 }
  const child = reactive(_obj)
  const parent = reactive(proto)
  
  Object.setPrototypeOf(child, parent)
  
  effect(() => console.log(child.bar))
  
  child.bar = 2 // 会导致副作用函数执行两次
```

  对于设置相同值,只需要判断新值与旧值即可.  原型链上的响应式值则需要判断 {% label info@receiver %}是不是{% label info@target %}的代理对象

```js
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key, receiver) {
      if (key === 'raw') {	
        return target
      }
      track(target, key)
      return Reflect.get(target, key, receiver)
    },
    set(target, key, newVal, receiver) {
      const oldVal = target[key]	// 对比新旧值
      const type = Object.prototype.hasOwnProperty.call(target, key) ? 'SET' : 'ADD'
      // 设置属性值
      const res = Reflect.set(target, key, newVal, receiver)
      if (target === receiver.raw) {	// 处理原型链上的响应式数据
        if (oldVal !== newVal && (oldVal === oldVal || newVal === newVal)) { // 处理NaN
          trigger(target, key, type)
        }
      }
      return res
    },
    
    // ...
  })
}
```

## 浅响应与深响应

```js
function reactive(obj) {
  return createReactive(obj)
}
function shallowReactive(obj) {
  return createReactive(obj, true)
}


function createReactive(obj, isShallow = false) {
  return new Proxy(obj, {
    // 拦截读取操作
    get(target, key, receiver) {
      if (key === 'raw') {
        return target
      }

      // 将副作用函数 activeEffect 添加到存储副作用函数的桶中
      track(target, key)

      const res = Reflect.get(target, key, receiver)

      if (isShallow) { 	// 浅响应
        return res
      }

      if (typeof res === 'object' && res !== null) {
        return reactive(res)	// 深响应
      }

      return res
    }
    // ...
  })
}
```

## 只读与浅只读

如果一个数据是只读的,那就意味着任何方式都无法修改它.因此,没有必要为只读数据建立响应联系

```js
function readonly(obj) {
  return createReactive(obj, false, true)
}

function shallowReadonly(obj) {
  return createReactive(obj, true, true)
}


function createReactive(obj, isShallow = false, isReadonly = false) {
  return new Proxy(obj, {
    // 拦截读取操作
    get(target, key, receiver) {
      if (key === 'raw') {
        return target
      }
      // 非只读的时候才需要建立响应联系
      if (!isReadonly) {
        track(target, key)
      }

      const res = Reflect.get(target, key, receiver)

      if (isShallow) {
        return res
      }

      if (typeof res === 'object' && res !== null) {
        // 深只读/响应
        return isReadonly ? readonly(res) : reactive(res)
      }

      return res
    },
    // 拦截设置操作
    set(target, key, newVal, receiver) {
      if (isReadonly) {
        console.warn(`属性 ${key} 是只读的`)
        return true
      }
      // ...
    },
    // ...
    deleteProperty(target, key) {
      if (isReadonly) {
        console.warn(`属性 ${key} 是只读的`)
        return true
      }
      // ...
    }
  })
}
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)
