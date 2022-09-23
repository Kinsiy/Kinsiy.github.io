---
title: 二叉树 - 遍历
date: 2021-12-02 21:20:15
tags: ['前序遍历','中序遍历','后序遍历','层序遍历']
categories:
 - ['数据结构','二叉树']
description:
photos:
---

## 遍历

> 二叉树的遍历(traversing binary tree) 是指从根节点出发，按照某种次序依次访问二叉树中所有节点，使得每个节点被访问一次且仅被访问一次
>
> ——程杰. 《大话数据结构》

下列遍历算法的二叉树节点结构均为

```JavaScript
class BitNode{
  constructor(value){
    this.value = value
    this.left = null
    this.right = null
  }
}
```

<!--more-->

## 深度优先遍历

### 前序遍历

根节点 → 左子树 → 右子树

#### 递归

```javascript
function* preorderTraversal(root) {
  if (root === null) return;
  yield root.value;
  yield* preorderTraversal(root.left);
  yield* preorderTraversal(root.right);
}

for (const node of preorderTraversal(root)) {
  console.log(node);
}
```

#### 迭代

```JavaScript
var preorderTraversal = function(root) {
  const result = []
  if(root === null) return result
  const stack = []
  let node
  stack.push(root)


  while(stack.length !== 0){
    node = stack.pop()
    result.push(node.val)

    if(node.right) stack.push(node.right)
    if(node.left) stack.push(node.left)
  }
  return result
};
```

### 中序遍历

左子树 → 根节点 → 右子树

#### 递归

```javascript
function* inorderTraversal(root) {
  if (root === null) return;

  yield* inorderTraversal(root.left);
  yield root.value;
  yield* inorderTraversal(root.right);
}

for (const node of inorderTraversal(root)) {
  console.log(node);
}
```

#### 迭代

```javascript
var inorderTraversal = function (root) {
  const result = [];
  const stack = [];
  let curNode = root;

  while (curNode !== null || stack.length !== 0) {
    if (curNode !== null) {
      stack.push(curNode);
      curNode = curNode.left;
    } else {
      curNode = stack.pop();
      result.push(curNode.val);
      curNode = curNode.right;
    }
  }

  return result;
};
```

### 后序遍历

左子树 → 右子树 → 根节点

#### 递归

```JavaScript
function* postorderTraversal(root){
  if(root === null) return

  yield* postorderTraversal(root.left)
  yield* postorderTraversal(root.right)
  yield root.value
}


for (const node of postorderTraversal(root)) {
  console.log(node)
}
```

#### 迭代

```javascript
var postorderTraversal = function (root) {
  const result = [];
  if (root === null) return result;
  const stack = [];
  let node;
  stack.push(root);

  while (stack.length !== 0) {
    node = stack.pop();
    result.push(node.val);
    if (node.left) stack.push(node.left); // 注意与前序遍历的不同
    if (node.right) stack.push(node.right);
  }
  return result.reverse(); // 注意与前序遍历的不同
};
```

## 广度优先遍历

二叉树的广度优先遍历又称按层次遍历。算法借助队列实现

```JavaScript
var levelOrder = function(root) {
  const result = []
  if(root === null) return result

  const queue = [root]
  let levelLength
  let levelNumber
  while(queue.length !== 0){
    levelNumber = []

    levelLength = queue.length // 记录层节点数

    for(let i = 0; i< levelLength; i++){
      let curNode = queue.shift()

      levelNumber.push(curNode.val)        // 存储每一层的节点

      if(curNode.left){
        queue.push(curNode.left)
      }
      if(curNode.right){
        queue.push(curNode.right)
      }
    }

    result.push(levelNumber)        // 把每一层的结果放到结果数组
  }
  return result
};
```

## 参考

[1\][程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/)

[2\][Wikipedia. 二叉树.](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%8F%89%E6%A0%91)
