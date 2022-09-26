---
title: 响应系统 - 非原始值 - Set&Map
mathjax: false
date: 2022-09-27 00:30:43
tags:
categories:
description:
photos:
---

{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第五章 - 非原始值的响应式方案 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

## 代理 Set 和 Map

Set 和 Map 的代理思路与代理数组是一样的, 需要注意的是其{% label info@size %}是一个访问器属性, 访问器内部检查了{% label danger@[[SetData]] %}内部槽, {% label info@Reflect.get %}中{% label info@this %}的值要指向原始对象而非代理对象,其余方法也是一样

<!-- more -->

### 建立响应联系

```js
const mutableInstrumentations = {
  add(key) {
    const target = this.raw
    const hadKey = target.has(key)
    const res = target.add(key)
    if (!hadKey) {
      trigger(target, key, 'ADD')
    }
    return res
  },
  delete(key) {
    const target = this.raw
    const hadKey = target.has(key)
    const res = target.delete(key)
    if (hadKey) {
      trigger(target, key, 'DELETE')
    }
    return res
  },
  get(key) {
    const target = this.raw
    const had = target.has(key)

    track(target, key)

    if (had) {
      const res = target.get(key)
      return typeof res === 'object' ? reactive(res) : res
    }
  },
  set(key, value) {
    const target = this.raw
    const had = target.has(key)

    const oldValue = target.get(key)
    const rawValue = value.raw || value
    target.set(key, rawValue)

    if (!had) {
      trigger(target, key, 'ADD')
    } else if (oldValue !== value || (oldValue === oldValue && value === value)) {
      trigger(target, key, 'SET')
    }
  }
}

function createReactive(obj, isShallow = false, isReadonly = false) {
  return new Proxy(obj, {
    get(target, key, receiver) {
      if (key === 'raw') return target
      if (key === 'size') {
        track(target, ITERATE_KEY)
        return Reflect.get(target, key, target)
      }

      return mutableInstrumentations[key]
    }
  })
}
```

### 避免污染原始数据

**响应式数据设置到原始数据上的行为称为数据污染**, 形如

```js
const m = new Map
const p1 = reactive(m)
const p2 = reactive(new Map)

p1.set('p2', p2)

effect(() => console.log(m.get('p2').size))

m.get('p2').set('foo', 1) // 2
```

注意 [建立响应联系](#建立响应联系)中{% label danger@mutableInstrumentations.set %}, 在  target.set 函数设置值之前对值进行检查: 只要发现即将要设置的值时响应式数据,那么就通过raw属性获取原始数据,再把原始数据设置到 target 上

{% note primary %}

除了set方法需要避免污染原始数据外, Set 类型的 add 方法, 普通对象的写值操作, 还有为数组添加元素的方法等, 都需要做类似的处理

{% endnote %}

### 处理forEach

由于上述 访问 {% label danger@size %}属性时, 是通过原始数据进行操作的, 会导致在forEach 回调函数中参数不是响应式的问题, 需要手动处理, 并且对比 {% label info@for...in %}操作, Map类型的forEach 需要在值更新时触发响应

```js
const mutableInstrumentations = {
  forEach(callback, thisArg) {
    const wrap = (val) => typeof val === 'object' ? reactive(val) : val
    const target = this.raw
    track(target, ITERATE_KEY)
    target.forEach((v, k) => {
      // 手动调用callback, 实现深响应
      callback.call(thisArg, wrap(v), wrap(k), this)
    })
  }
}

function trigger(target, key, type, newVal) {
  // ...

  const effectsToRun = new Set()
  effects && effects.forEach(effectFn => {
    if (effectFn !== activeEffect) {
      effectsToRun.add(effectFn)
    }
  })
  
  if (
    type === 'ADD' ||
    type === 'DELETE' ||
    (
      type === 'SET' &&
      Object.prototype.toString.call(target) === '[object Map]'
    )
  ) {
    const iterateEffects = depsMap.get(ITERATE_KEY)
    iterateEffects && iterateEffects.forEach(effectFn => {
      if (effectFn !== activeEffect) {
        effectsToRun.add(effectFn)
      }
    })
  }

  // ...
}
```

### 迭代器方法

Map 类型的迭代器方法, 包括{% label info@[Symbol.iterator]/entries/keys/values %}, 与上述{% label info@forEach %}一样, 迭代器方法需要将迭代产生的值包装成响应式数据.

{% note info%}

可迭代协议指的是一个对象实现了{% label info@Symbol.iterator %}

迭代器协议指的是一个对象实现了{% label info@next %}方法

{% endnote %}

```js
const mutableInstrumentations = {
  [Symbol.iterator]: iterationMethod,
  entries: iterationMethod,
  keys: keysIterationMethod,
  values: valuesIterationMethod,
}

function iterationMethod() {
  // 获取原始数据对象 target
  const target = this.raw
  // 获取到原始迭代器方法
  const itr = target[Symbol.iterator]()

  const wrap = (val) => typeof val === 'object' ? reactive(val) : val

  // 调用 track 函数建立响应联系
  track(target, ITERATE_KEY)

  // 将其返回
  return {
    next() {
      const { value, done } = itr.next()
      return {
        value: value ? [wrap(value[0]), wrap(value[1])] : value,
        done
      }
    },
    [Symbol.iterator]() {
      return this
    }
  }
}
```

### values 与 keys 方法

values 方法的实现与entries 方法类似, 不同的是, 当使用 for...of 迭代values是, 得到的仅仅是Map数据的值, 而非键值对

```js
function valuesIterationMethod() {
  // 获取原始数据对象 target
  const target = this.raw
  // 获取到原始迭代器方法
  const itr = target.values()

  const wrap = (val) => typeof val === 'object' ? reactive(val) : val

  track(target, ITERATE_KEY)

  // 将其返回
  return {
    next() {
      const { value, done } = itr.next()
      return {
        value: wrap(value),	// 只有值
        done
      }
    },
    [Symbol.iterator]() {
      return this
    }
  }
}
```

keys 方法与values 方法非常类似, 不同点在于, 前者处理的是键而非值.

另外对于 values 或 entries 来说 Map 类型的数据 即使操作类型为 SET, 也会触发那些与 ITERATE_KEY 相关联的副作用函数, 但这对keys 来说是没有必要的, keys 方法 只关心Map 类型数据的键的变化

```js
const MAP_KEY_ITERATE_KEY = Symbol()

function keysIterationMethod() {
  // 获取原始数据对象 target
  const target = this.raw
  // 获取到原始迭代器方法
  const itr = target.keys()	// 获取键值

  const wrap = (val) => typeof val === 'object' ? reactive(val) : val

  track(target, MAP_KEY_ITERATE_KEY)

  // 将其返回
  return {
    next() {
      const { value, done } = itr.next()
      return {
        value: wrap(value),
        done
      }
    },
    [Symbol.iterator]() {
      return this
    }
  }
}

function trigger(target, key, type, newVal) {
  // ...
  if ((
    type === 'ADD' || type === 'DELETE') &&
    Object.prototype.toString.call(target) === '[object Map]'
  ) {
    // 执行 Map 类型 keys 相关联的副作用函数
    const iterateEffects = depsMap.get(MAP_KEY_ITERATE_KEY)
    iterateEffects && iterateEffects.forEach(effectFn => {
      if (effectFn !== activeEffect) {
        effectsToRun.add(effectFn)
      }
    })
  }
  // ...
}
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)
