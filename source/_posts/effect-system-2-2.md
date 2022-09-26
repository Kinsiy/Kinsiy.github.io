---
title: 响应系统 - 非原始值 - 数组
mathjax: false
date: 2022-09-27 00:30:30
tags:
categories:
description:
photos:
---

{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第五章 - 非原始值的响应式方案 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

## 代理数组

数组属于异质对象, 其{% label info@[[DefineOwnProperty]] %}内部方法与常规对象不同. 大部分用来代理常规对象的代码对于数组也是生效的. 但数组的一些操作需要特别对待

<!-- more -->

### 数组的索引与length

- 通过索引设置大于当前数组长度的值时,会影响{% label primary@length %} , 与属性 {% label primary@length %}相关的副作用函数需要重新执行
- 反过来, 当修改{% label primary@length %}的值时, 索引值大于或等于新的{% label primary@length %}属性值的元素需要触发响应

```js
function createReactive(obj, isShallow = false, isReadonly = false) {
  return new Proxy(obj, {

    // ...
    set(target, key, newVal, receiver) {
      if (isReadonly) {
        console.warn(`属性 ${key} 是只读的`)
        return true
      }
      const oldVal = target[key]
      const type = Array.isArray(target)
        ? Number(key) < target.length ? 'SET' : 'ADD'	// 区分设置大于当前数组长度的值的操作
        : Object.prototype.hasOwnProperty.call(target, key) ? 'SET' : 'ADD'
      
      const res = Reflect.set(target, key, newVal, receiver)
      if (target === receiver.raw) {
        if (oldVal !== newVal && (oldVal === oldVal || newVal === newVal)) {
          // 增加第四个参数, 即触发响应的新值
          trigger(target, key, type, newVal)
        }
      }

      return res
    }
    // ...
  })
}


function trigger(target, key, type, newVal) {
	// ...
  // 当操作类型为 ADD 并且目标类型是数组时,应该取出并执行那些与 length 属性相关联的副作用函数
  if (type === 'ADD' && Array.isArray(target)) {
    const lengthEffects = depsMap.get('length')
    lengthEffects && lengthEffects.forEach(effectFn => {
      if (effectFn !== activeEffect) {
        effectsToRun.add(effectFn)
      }
    })
  }

  // 如果操作目标是数组, 并且修改了数组的 length 属性
  if (Array.isArray(target) && key === 'length') {
    depsMap.forEach((effects, key) => {
      if (key >= newVal) {
        effects.forEach(effectFn => {
          if (effectFn !== activeEffect) {
            effectsToRun.add(effectFn)
          }
        })
      }
    })
  }
  // ...
}
```

### 遍历数组

#### for...in

尽管{% label info@for...in %}一般不用来(也不建议)遍历数组, 但数组也是对象, 在语法上是行得通的.

当数组的 {% label info@length %}变化时, 我们需要去触发响应

```js
const ITERATE_KEY = Symbol()

function createReactive(obj, isShallow = false, isReadonly = false) {
  return new Proxy(obj, {
    // ...
    ownKeys(target) {
      track(target, Array.isArray(target) ? 'length' : ITERATE_KEY)
      return Reflect.ownKeys(target)
    }
    // ... 
  })
}
```

#### for...of

当使用 {% label info@for...of %}遍历时, 会去读取{% label info@length %}属性, 我们不需要增加任何代码就可以使其正确地工作.但无论是使用{% label info@for...of %}循环,还是调用{% label info@values %}方法,它们都会读取数组的 {% label primary@Symbol.iterator %} 属性.该属性是一个{% label info@symbol %}值,为了避免发生意外的错误,以及性能上的考虑, 不应该在副作用函数与 {% label primary@Symbol.iterator %} 这类{% label info@symbol %}值之间建立联系

```js
function createReactive(obj, isShallow = false, isReadonly = false) {
  return new Proxy(obj, {
    // ...
    get(target, key, receiver) {
      if (key === 'raw') {
        return target
      }

      // 添加判断, 如果key的类型是 symbol, 则不进行追踪
      if (!isReadonly && typeof key !== 'symbol') {
        track(target, key)
      }

			// ...
    }
    // ...
  })
}
```

### 数组的查找方法

由于在 [浅响应与深响应](/effect-system-2-1#浅响应与深响应)中我们做了深响应, 即通过代理对象来访问元素值时,如果值仍然是可以被代理的,那么得到的值就是新的代理对象而非原始对象.

```js
if(typeof res === 'object' && res !== null){
  return isReadonly ? readonly(res) : reactive(res)
}
```

由于查找方法{% label info@includes/indexOf/lastIndexOf %}在ECMA规范中的实现需要读取索引值并与查找值对比, 返回代理对象无法查找到正确值,解决如下问题

```js
const obj = {}
const arr = reactive([obj])

console.log(arr.includes(arr[0])) // false

console.log(arr.includes(obj)) // false
```

第一个问题,需要缓存代理对象, 优先通过原始对象寻找之前创建的代理对象.

第二个问题,需要重写查找方法{% label info@includes/indexOf/lastIndexOf %}解决

```js
// 定义一个Map实例, 存储原始对象到代理对象的映射
const reactiveMap = new Map()

function reactive(obj) {
  // 优先通过原始对象obj 寻找之前创建的代理对象, 如果找到了, 直接返回已有的代理对象
  const proxy = createReactive(obj)

  const existionProxy = reactiveMap.get(obj)
  if (existionProxy) return existionProxy
	
  // 否则, 创建新的代理对象
  const proxy = creatReactive(obj)
  // 存储到 Map 中, 从而避免重复创建
  reactiveMap.set(obj, proxy)

  return proxy
}

const arrayInstrumentations = {}

;['includes', 'indexOf', 'lastIndexOf'].forEach(method => {
  const originMethod = Array.prototype[method]
  arrayInstrumentations[method] = function(...args) {
    // this 是代理对象，先在代理对象中查找，将结果存储到 res 中
    let res = originMethod.apply(this, args)

    if (res === false) {
      // res 为 false 说明没找到，在通过 this.raw 拿到原始数组，再去原始数组中查找，并更新 res 值
      res = originMethod.apply(this.raw, args)
    }
    // 返回最终的结果
    return res
  }
})

function createReactive(obj, isShallow = false, isReadonly = false) {
  return new Proxy(obj, {
    // 拦截读取操作
    get(target, key, receiver) {
      if (key === 'raw') {
        return target
      }

      // 注意这里是在设置追踪之前返回的, 也就是说实际查找方法 includes/indexOf/lastIndexOf 的响应追踪值并不是自身, 而是由于查找方法会读取每一个索引值, 会把副作用函数与没一个索引值绑定
      if (Array.isArray(target) && arrayInstrumentations.hasOwnProperty(key)) {
        return Reflect.get(arrayInstrumentations, key, receiver)
      }
      // 
      if (!isReadonly && typeof key !== 'symbol') {
        track(target, key)
      }
      // ...
    }
    // ...
  })
}
```

{% note warning %}

**注意**

上面{% label danger@createReactive %}函数是在设置追踪之前返回的, 也就是说实际查找方法 includes/indexOf/lastIndexOf 的响应追踪值并不是自身, 而是由于查找方法会读取每一个索引值, 会把副作用函数与每一个索引值绑定

{% endnote %}

### 隐式修改数组长度的原型方法

对于 **push, pop, shift, unshift, splice** 这些隐式修改数组长度的原型方法,也需要进行重写. 解决形如如下问题

```js
const arr = reactive([])

effect(() => arr.push(1))
effect(() => arr.push(1))	// 栈溢出 Maximum call stack size exceeded
```

第二个副作用函数执行时,会隐式改变数组的{% label info@length %}属性, 触发第一个副作用函数执行, 第一个副作用函数执行又会触发第二个函数,如此反复

```js
let shouldTrack = true
;['push'].forEach(method => {
  const originMethod = Array.prototype[method]
  arrayInstrumentations[method] = function(...args) {
    shouldTrack = false
    let res = originMethod.apply(this, args)
    shouldTrack = true
    return res
  }
})

function track(target, key) {
  // 当禁止追踪时, 直接返回
  if (!activeEffect || !shouldTrack) return
  // ... 
}
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)
