---
title: 快速 Diff 算法
mathjax: false
date: 2022-10-09 18:57:05
tags: 'diff'
categories:
 - ['Vue', 'Vue.js设计与实现']
description:
photos:
---

{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第十一章 - 快速 Diff 算法 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

<!-- more -->

## 代码实现

在书中有算法实现的完整思路, 由浅入深,说的非常明白. 这里仅贴出最终的示例代码, 供重温用, 更为具体的讲解参见书籍原文. 

不同于简单Diff算法和双端Diff算法, 快速Diff算法包含预处理步骤, 在对两列VNode进行Diff之前,先对他们进行{% label danger@“全等” %}比较. {% label danger@patchKeyedChildren %} 中 {% label danger@2 ~ 36 %} 行即为其预处理过程. 

另一个导致其比简单与双端diff算法都快的原因,在于快速diff算法使用了最长递增子序列辅助完成DOM的移动操作

**最长递增子序列**: 简单来说,给定一个数值序列,找到它的一个子序列,并且该子序列中的值是递增的,子序列中的元素在原序列中不一定连续

{% note primary %}

代码中{% label info@patch, patchElement, patchChildren %}函数, 在书中第八章 - 挂载与更新中有详细的讲解, 代码已贴到 [简单 Diff 算法#附录](/simple-diff/#附录)中

{% endnote %}

```js
function patchKeyedChildren(n1, n2, container) {
  const newChildren = n2.children
  const oldChildren = n1.children
  // 更新相同的前缀节点
  // 索引 j 指向新旧两组子节点的开头
  let j = 0
  let oldVNode = oldChildren[j]
  let newVNode = newChildren[j]
  // while 循环向后遍历，直到遇到拥有不同 key 值的节点为止
  while (oldVNode.key === newVNode.key) {
    // 调用 patch 函数更新
    patch(oldVNode, newVNode, container)
    j++
    oldVNode = oldChildren[j]
    newVNode = newChildren[j]
  }

  // 更新相同的后缀节点
  // 索引 oldEnd 指向旧的一组子节点的最后一个节点
  let oldEnd = oldChildren.length - 1
  // 索引 newEnd 指向新的一组子节点的最后一个节点
  let newEnd = newChildren.length - 1

  oldVNode = oldChildren[oldEnd]
  newVNode = newChildren[newEnd]

  // while 循环向前遍历，直到遇到拥有不同 key 值的节点为止
  while (oldVNode.key === newVNode.key) {
    // 调用 patch 函数更新
    patch(oldVNode, newVNode, container)
    oldEnd--
    newEnd--
    oldVNode = oldChildren[oldEnd]
    newVNode = newChildren[newEnd]
  }

  // 满足条件，则说明从 j -> newEnd 之间的节点应作为新节点插入
  if (j > oldEnd && j <= newEnd) {
    // 锚点的索引
    const anchorIndex = newEnd + 1
    // 锚点元素
    const anchor = anchorIndex < newChildren.length ? newChildren[anchorIndex].el : null
    // 采用 while 循环，调用 patch 函数逐个挂载新增的节点
    while (j <= newEnd) {
      patch(null, newChildren[j++], container, anchor)
    }
  } else if (j > newEnd && j <= oldEnd) {
    // j -> oldEnd 之间的节点应该被卸载
    while (j <= oldEnd) {
      unmount(oldChildren[j++])
    }
  } else {
    // 构造 source 数组
    // source数组将用来存储新的一组子节点中的节点在旧的一组子节点中的位置索引(用于计算最长递增子序列,辅助DOM移动)
    const count = newEnd - j + 1  // 新的一组子节点中剩余未处理节点的数量
    const source = new Array(count)
    source.fill(-1)

    const oldStart = j
    const newStart = j
    let moved = false
    let pos = 0
    const keyIndex = {}	// 索引表, 避免嵌套循环.
    for(let i = newStart; i <= newEnd; i++) {
      keyIndex[newChildren[i].key] = i
    }
    let patched = 0	// 已更新节点数
    for(let i = oldStart; i <= oldEnd; i++) {
      oldVNode = oldChildren[i]
      if (patched < count) {
        const k = keyIndex[oldVNode.key]
        if (typeof k !== 'undefined') {
          newVNode = newChildren[k]
          patch(oldVNode, newVNode, container)
          patched++
          source[k - newStart] = i
          // 判断是否需要移动
          if (k < pos) {
            moved = true
          } else {
            pos = k
          }
        } else {
          // 没找到
          unmount(oldVNode)
        }
      } else {
        unmount(oldVNode)
      }
    }

    if (moved) {
      const seq = lis(source)	// 计算最长递增子序列
      // s 指向最长递增子序列的最后一个值
      let s = seq.length - 1
      let i = count - 1
      for (i; i >= 0; i--) {
        if (source[i] === -1) {
          // 说明索引为 i 的节点是全新的节点，应该将其挂载
          // 该节点在新 children 中的真实位置索引
          const pos = i + newStart
          const newVNode = newChildren[pos]
          // 该节点下一个节点的位置索引
          const nextPos = pos + 1
          // 锚点
          const anchor = nextPos < newChildren.length
            ? newChildren[nextPos].el
            : null
          // 挂载
          patch(null, newVNode, container, anchor)
        } else if (i !== seq[j]) {
          // 说明该节点需要移动
          // 该节点在新的一组子节点中的真实位置索引
          const pos = i + newStart
          const newVNode = newChildren[pos]
          // 该节点下一个节点的位置索引
          const nextPos = pos + 1
          // 锚点
          const anchor = nextPos < newChildren.length
            ? newChildren[nextPos].el
            : null
          // 移动
          insert(newVNode.el, container, anchor)
        } else {
          // 当 i === seq[j] 时，说明该位置的节点不需要移动
          // 并让 s 指向下一个位置
          s--
        }
      }
    }
  }
}
```

## 附录

### 计算递增最长子序列

```js
function lis(arr) {
  const p = arr.slice()
  const result = [0]
  let i, j, u, v, c
  const len = arr.length
  for (i = 0; i < len; i++) {
    const arrI = arr[i]
    if (arrI !== 0) {
      j = result[result.length - 1]
      if (arr[j] < arrI) {
        p[i] = j
        result.push(i)
        continue
      }
      u = 0
      v = result.length - 1
      while (u < v) {
        c = ((u + v) / 2) | 0
        if (arr[result[c]] < arrI) {
          u = c + 1
        } else {
          v = c
        }
      }
      if (arrI < arr[result[u]]) {
        if (u > 0) {
          p[i] = result[u - 1]
        }
        result[u] = i
      }
    }
  }
  u = result.length
  v = result[u - 1]
  while (u-- > 0) {
    result[u] = v
    v = p[v]
  }
  return result
}
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)
