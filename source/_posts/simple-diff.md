---
title: 简单Diff算法
mathjax: false
date: 2022-09-28 23:14:25
tags: 'diff'
categories:
 - ['Vue', 'Vue.js设计与实现']
description:
photos:
---

{% note primary %}

文章依据[《Vue.js设计与实现》](https://book.douban.com/subject/35768338/)第九章 - 简单Diff算法 摘录而来, 更为详细的内容参见书籍原文

{% endnote %}

<!-- more -->

## 代码实现

在书中有算法实现的完整思路, 由浅入深,说的非常明白. 这里仅贴出最终的示例代码, 更为具体的讲解参见书籍原文. 

{% note primary %}

代码中的{% label info@patch %}函数, 在书中第八章 - 挂载与更新中有详细的讲解, 代码已贴到附录中

{% endnote %}

```js
  function patchChildren(n1, n2, container) {
    if (typeof n2.children === 'string') {
      if (Array.isArray(n1.children)) {
        n1.children.forEach((c) => unmount(c))
      }
      setElementText(container, n2.children)
    } else if (Array.isArray(n2.children)) {
      const oldChildren = n1.children
      const newChildren = n2.children

      let lastIndex = 0
      // 遍历新的 children
      for (let i = 0; i < newChildren.length; i++) {
        const newVNode = newChildren[i]
        let j = 0
        let find = false
        // 遍历旧的 children
        for (j; j < oldChildren.length; j++) {
          const oldVNode = oldChildren[j]
          // 如果找到了具有相同 key 值的两个节点，则调用 `patch` 函数更新之
          if (newVNode.key === oldVNode.key) {
            find = true
            patch(oldVNode, newVNode, container)
            if (j < lastIndex) {
              // 需要移动
              const prevVNode = newChildren[i - 1]
              // 如果 prevVNode  不存在, 则说明当前 newVNode 是第一个节点, 它不需要移动
              if (prevVNode) {
                const anchor = prevVNode.el.nextSibling
                // 移动到 prevVNode 对应的真实DOM的后面
                insert(newVNode.el, container, anchor)	// insertBefore
              }
            } else {
              // 更新 lastIndex
              lastIndex = j
            }
            break // 这里需要 break
          }
        }
        // 当前 newVNode 没有在旧的一组节点中找到可复用的节点
        // 也就是说, 当前newVNode 是新增节点, 需要挂载
        if (!find) {
          // 为了将节点挂载到正确位置,我们需要先获取锚点元素
          // 首先获取当前 newVNode 的前一个 vnode 节点
          const prevVNode = newChildren[i - 1]
          let anchor = null
          if (prevVNode) {
            // 如果有前一个 vnode 节点, 则使用它的下一个兄弟节点作为锚点元素
            anchor = prevVNode.el.nextSibling
          } else {
            // 如果没有前一个 vnode 节点, 说明即将挂载的新节点是第一个节点
            // 使用容器元素的 firstChild 作为锚点
            anchor = container.firstChild
          }
          patch(null, newVNode, container, anchor)
        }
      }

      // 遍历旧的节点
      for (let i = 0; i < oldChildren.length; i++) {
        const oldVNode = oldChildren[i]
        // 拿着旧 VNode 去新 children 中寻找相同的节点
        const has = newChildren.find(
          vnode => vnode.key === oldVNode.key
        )
        if (!has) {
          // 如果没有找到相同的节点，则移除
          unmount(oldVNode)
        }
      }
      
    } else {
      if (Array.isArray(n1.children)) {
        n1.children.forEach(c => unmount(c))
      } else if (typeof n1.children === 'string') {
        setElementText(container, '')
      }
    }
  }
```

## 附录

### patch

{% note warning %}

代码中有很多方法具体实现并未贴出, 根据函数名很容易看出函数的作用, 这里主要做理解用, 具体实现参考书籍原文或[Github](https://github.com/HcySunYang/code-for-vue-3-book/blob/master/course5-%E6%B8%B2%E6%9F%93%E5%99%A8/code-9.6.html)

{% endnote %}

```js
function patch(n1, n2, container, anchor) {
  if (n1 && n1.type !== n2.type) {
    unmount(n1)
    n1 = null
  }

  const { type } = n2

  if (typeof type === 'string') {
    if (!n1) {
      mountElement(n2, container, anchor)
    } else {
      patchElement(n1, n2)
    }
  } else if (type === Text) {
    if (!n1) {
      const el = n2.el = createText(n2.children)
      insert(el, container)
    } else {
      const el = n2.el = n1.el
      if (n2.children !== n1.children) {
        setText(el, n2.children)
      }
    }
  } else if (type === Fragment) {
    if (!n1) {
      n2.children.forEach(c => patch(null, c, container))
    } else {
      patchChildren(n1, n2, container)
    }
  }
}
```

## 参考

[1\][霍春阳. Vue.js设计与实现](https://book.douban.com/subject/35768338/)
