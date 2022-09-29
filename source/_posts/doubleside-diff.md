---
title: 双端 Diff 算法
mathjax: false
date: 2022-09-29 22:16:21
tags: diff
categories:
 - ['Vue', 'Vue.js设计与实现']
description:
photos:
---

{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第十章 - 双端 Diff 算法 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

<!-- more -->

双端 Diff 的算法指的是, 在新旧两组子节点的四个端点之间分别进行比较, 并视图找到可复用的节点.相比简单 Diff 算法, 双端 Diff 算法的优势在于, 对于同样的更新场景, 执行的 DOM 移动操作次数更少

## 代码实现

在书中有算法实现的完整思路, 由浅入深,说的非常明白. 这里仅贴出最终的示例代码, 供重温用, 更为具体的讲解参见书籍原文. 

{% note primary %}

代码中{% label info@patch, patchElement, patchChildren %}函数, 在书中第八章 - 挂载与更新中有详细的讲解, 代码已贴到 [简单 Diff 算法#附录](/simple-diff/#附录)中

{% endnote %}

```js
// 双端 Diff 算法
function patchKeyedChildren(n1, n2, container) {
  const oldChildren = n1.children
  const newChildren = n2.children
  
  // 四个索引值
  let oldStartIdx = 0
  let oldEndIdx = oldChildren.length - 1
  let newStartIdx = 0
  let newEndIdx = newChildren.length - 1

  // 四个索引指向的 vnode 节点
  let oldStartVNode = oldChildren[oldStartIdx]
  let oldEndVNode = oldChildren[oldEndIdx]
  let newStartVNode = newChildren[newStartIdx]
  let newEndVNode = newChildren[newEndIdx]

  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    if (!oldStartVNode) {
      // DOM 已被移动
      oldStartVNode = oldChildren[++oldStartIdx]
    } else if (!oldEndVNode) {
      // DOM 已被移动
      oldEndVNode = newChildren[--oldEndIdx]
    } else if (oldStartVNode.key === newStartVNode.key) {
      // 步骤一： oldStartVNode 和 newStartVNode 比较
      // 无需移动
      patch(oldStartVNode, newStartVNode, container)
      oldStartVNode = oldChildren[++oldStartIdx]
      newStartVNode = newChildren[++newStartIdx]
    } else if (oldEndVNode.key === newEndVNode.key) {
      // 步骤二： oldEndVNode 和 newEndVNode 比较
      // 无需移动
      patch(oldEndVNode, newEndVNode, container)
      oldEndVNode = oldChildren[--oldEndIdx]
      newEndVNode = newChildren[--newEndIdx]
    } else if (oldStartVNode.key === newEndVNode.key) {
      // 步骤三： oldStartVNode 和 newEndVNode 比较
      patch(oldStartVNode, newEndVNode, container)
      // oldStartVNode.el 移动到 oldEndVNode.el.nextSibling 前面
      insert(oldStartVNode.el, container, oldEndVNode.el.nextSibling)

      oldStartVNode = oldChildren[++oldStartIdx]
      newEndVNode = newChildren[--newEndIdx]
    } else if (oldEndVNode.key === newStartVNode.key) {
      // 步骤四：oldEndVNode 和 newStartVNode 比对
      patch(oldEndVNode, newStartVNode, container)
      // oldEndVNode.el 移动到 oldStartVNode.el 前面
      insert(oldEndVNode.el, container, oldStartVNode.el)

      oldEndVNode = oldChildren[--oldEndIdx]
      newStartVNode = newChildren[++newStartIdx]
    } else {
      // 遍历旧 children，试图寻找与 newStartVNode 拥有相同 key 值的元素
      const idxInOld = oldChildren.findIndex(
        node => node.key === newStartVNode.key
      )
      if (idxInOld > 0) {
        // 找到可复用的节点
        const vnodeToMove = oldChildren[idxInOld]
        patch(vnodeToMove, newStartVNode, container)
        // vnodeToMove.el 移动到 oldStartVNode.el 前
        insert(vnodeToMove.el, container, oldStartVNode.el)
        oldChildren[idxInOld] = undefined	// 标记该处对应的真实DOM已经移动
      } else {
        // 没找到可复用的节点 挂载新节点到 oldStartVNode.el 前
        patch(null, newStartVNode, container, oldStartVNode.el)
      }

      newStartVNode = newChildren[++newStartIdx]
    }
  }

  if (oldEndIdx < oldStartIdx && newStartIdx <= newEndIdx) {
    // 添加新节点
    for (let i = newStartIdx; i <= newEndIdx; i++) {
      patch(null, newChildren[i], container, oldStartVNode.el)
    }
  } else if (newEndIdx < newStartIdx && oldStartIdx <= oldEndIdx) {
    // 移除操作
    for (let i = oldStartIdx; i <= oldEndIdx; i++) {
      unmount(oldChildren[i])
    }
  }
}
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)
