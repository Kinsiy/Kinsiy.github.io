---
title: 响应系统 - 实现
mathjax: false
date: 2022-09-18 11:26:48
tags: 响应式
categories:
 - ['Vue', 'Vue.js设计与实现']
description:
photos:
---

{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第四章 - 响应系统的作用与实现 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

## 一个完善的响应式系统

<!-- more -->

```js
// 存储副作用函数的桶
const bucket = new WeakMap()

// 原始数据
const data = { foo: 1 }
// 对原始数据的代理
const obj = new Proxy(data, {
  // 拦截读取操作
  get(target, key) {
    // 将副作用函数 activeEffect 添加到存储副作用函数的桶中
    track(target, key)
    // 返回属性值
    return target[key]
  },
  // 拦截设置操作
  set(target, key, newVal) {
    // 设置属性值
    target[key] = newVal
    // 把副作用函数从桶里取出并执行
    trigger(target, key)
  }
})


function track(target, key) {
  let depsMap = bucket.get(target)
  if (!depsMap) {
    bucket.set(target, (depsMap = new Map()))
  }
  let deps = depsMap.get(key)
  if (!deps) {
    depsMap.set(key, (deps = new Set()))
  }
  deps.add(activeEffect)
  activeEffect.deps.push(deps)
}

function trigger(target, key) {
  const depsMap = bucket.get(target)
  if (!depsMap) return
  const effects = depsMap.get(key)

  const effectsToRun = new Set()
  effects && effects.forEach(effectFn => {
    if (effectFn !== activeEffect) {
      // 避免无限循环
      effectsToRun.add(effectFn)
    }
  })
  effectsToRun.forEach(effectFn => effectFn())
  // effects && effects.forEach(effectFn => effectFn())
}

// 用一个全局变量存储当前激活的 effect 函数
let activeEffect
// effect 栈  用于支持effect嵌套
const effectStack = []

function effect(fn) {
  const effectFn = () => {
    cleanup(effectFn)  // 避免副作用函数遗留
    // 当调用 effect 注册副作用函数时，将副作用函数复制给 activeEffect
    activeEffect = effectFn
    // 在调用副作用函数之前将当前副作用函数压栈
    effectStack.push(effectFn)
    fn()
    // 在当前副作用函数执行完毕后，将当前副作用函数弹出栈，并还原 activeEffect 为之前的值
    effectStack.pop()
    activeEffect = effectStack[effectStack.length - 1]
  }
  // activeEffect.deps 用来存储所有与该副作用函数相关的依赖集合
  effectFn.deps = []
  // 执行副作用函数
  effectFn()
}

/**
 cleanup 函数接收副作用函数作为参数,遍历副作用函数的effectRn.deps数组,该数组的每一项都是一个依赖集合,然后将该副作用函数从依赖集合中移除,最后重置effectFn.deps数组
 */
function cleanup(effectFn) {
  for (let i = 0; i < effectFn.deps.length; i++) {
    const deps = effectFn.deps[i]
    deps.delete(effectFn)
  }
  effectFn.deps.length = 0
}
```

## 调度执行

{% note primary %}

可调度性:  当trigger动作触发副作用函数重新执行时,有能力决定副作用函数执行的时机/次数以及方式

{% endnote %}

```js
// 通过在options 上指定 scheduler 调度器实现 调度器 scheduler是一个函数
function effect(fn, options = {}) {
  const effectFn = () => {
    //...略...
  }
  // 将 options 挂在到 effectFn 上
  effectFn.options = options
  // activeEffect.deps 用来存储所有与该副作用函数相关的依赖集合
  effectFn.deps = []
  // 执行副作用函数
  effectFn()
}

function trigger(target, key) {
  // ...略...
  effectsToRun.forEach(effectFn => {
    if (effectFn.options.scheduler) {
      effectFn.options.scheduler(effectFn)    // 把控制权交给用户
    } else {
      effectFn()
    }
  })
}
```

## 计算属性computed与lazy

上面实现的 effect 函数会立即执行传递给它的副作用函数. 但在有些场景下,我们并不希望它立即执行,而是希望它在需要的时候才执行,例如计算属性.这时我们可以通过在options中添加lazy属性来达到目的.

```js
function effect(fn, options = {}) {
   const effectFn = () => {
    // ...略...
    const res = fn()  // 将真正的副作用函数fn的执行结果保存到res中
    // ...略...
    return res  // 将其作为effectFn的返回值
  }
  // ...略...
  // 执行副作用函数
  if (!options.lazy) {
    effectFn()
  }

  return effectFn
}

// 在手动执行时拿到副作用函数的返回值
const effectFn = effect(() => obj.foo + obj.bar, {lazy: true})
const value = effectFn()
```

有了上面的基数就可以实现计算属性了

```js
function computed(getter){
  const effectFn = effect(gerrer, {lazy: true})
  const obj = {
    // 当读取value时才执行effectFn
    get value(){
      return effectFn()
    }
  }
  return obj
}
```

上述实现的计算属性只做到了懒计算, 还做不到对值进行缓存, 假如我们多次访问计算属性的valve值,会导致effectFn进行多次计算

```js
function computed(getter) {
  /**
   加入缓存, 在响应式数据发生变化时, 置缓存无效. 并触发计算属性响应. 即计算属性也是一个响应式数据
   */
  let value
  let dirty = true

  const effectFn = effect(getter, {
    lazy: true,
    scheduler() {
      // getter函数中所依赖的响应式数据变化时执行
      if (!dirty) {
        dirty = true
        trigger(obj, 'value')  // 触发响应
      }
    }
  })
  
  const obj = {
    get value() {
      if (dirty) {
        value = effectFn()
        dirty = false
      }
      track(obj, 'value')  // 手动调用 track 函数进行追踪
      return value
    }
  }

  return obj
}
```

## watch的实现原理

```js
/**
 递归的进行读取操作, 这样就可以追踪到任意属性的变化, 这里仅考虑value就是一个对象,不考虑数组等其他结构
 */

function traverse(value, seen = new Set()) {
  if (typeof value !== 'object' || value === null || seen.has(value)) return
  seen.add(value)
  for (const k in value) {
    traverse(value[k], seen)
  }

  return value
}

/**
 watch函数可以接收一个响应式数据或者一个getter函数.在getter函数内部,用户可以指定该watch依赖那些响应式数据,只有这些数据变化时,才会触发回调函数执行
 */
function watch(source, cb, options = {}) {
  let getter
  if (typeof source === 'function') {
    getter = source
  } else {
    getter = () => traverse(source)
  }

  let oldValue, newValue

  const job = () => {
    newValue = effectFn()
    cb(oldValue, newValue)
    oldValue = newValue
  }

  const effectFn = effect(
    // 执行 getter
    () => getter(),
    {
      lazy: true,  // 利用 lazy 选项 获取新值与旧值
      scheduler: () => {
        if (options.flush === 'post') {  // flush 选项指定回调执行时机
          // post 代表调度函数需要将副作用函数放到一个微任务队列中,并等待DOM更新结束后再执行
          const p = Promise.resolve()
          p.then(job)
        } else {
          job()
        }
      }
    }
  )
  
  if (options.immediate) {
    job()
  } else {
    oldValue = effectFn()
  }
}
```

## 过期的副作用

思考

```js
let finalData

watch(obj, async () => {
  // 发送请求并等待结果
  const res = await fetch('/path/to/request')
  // 将请求结果赋给data
  finalData = res
})
```

假设我们第一次修改obj对象的某个字段值,这会导致回调函数执行,同时发送了第一次请求A.随着时间的推移,在请求A的结果返回之前,我们对obj对象的某个字段值进行了第二次修改,这会导致发送第二次请求B.此时请求A和请求B都在进行中,不能够确定哪一个请求会先返回结果.如果请求B先于请求A返回结果,就会导致最终finalData中存储的是A请求的结果.

但由于请求B是后发送的,因此我们认为请求B返回的数据才是最新的,而请求A则应该被视为“过期”的.finalData存储的值应该是由请求B返回的结果,而非请求A返回的结果.

```js
function watch(source, cb, options = {}) {
  // ...略...
  let oldValue, newValue

  let cleanup // 存储用户注册的过期回调
  function onInvalidate(fn) {
    cleanup = fn
  }

  const job = () => {
    newValue = effectFn()
    // 在调用回调函数cb之前,先调用过期回调
    if (cleanup) {
      cleanup()
    }
    cb(oldValue, newValue, onInvalidate)
    oldValue = newValue
  }

  // ...略...
}
```

稍微修改上面请求的代码即可得到正确的结果

```js
let finallyData

watch(() => obj.foo, async (newVal, oldVal, onInvalidate) => {
  let valid = true
  // 调用onInvalidate 函数注册一个过期回调
  onInvalidate(() => {
    valid = false
  })
  const res = await fetch()

  if (!valid) return

  finallyData = res
  console.log(finallyData)
})

obj.foo++
setTimeout(() => {
  obj.foo++
}, 200);
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)

