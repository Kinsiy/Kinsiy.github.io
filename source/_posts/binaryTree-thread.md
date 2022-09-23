---
title: 二叉树 - 线索化
date: 2021-12-03 20:57:17
tags: ["线索二叉树"]
categories:
    - ["数据结构", "二叉树"]
description:
photos:
---

## 定义

> 线索二叉树（threaded binary tree，保留遍历时结点在任一序列的前驱和后继的信息）：若结点有左子树，则其 lchild 域指示其左孩子，否则令 lchild 域指示其前驱；若结点有右子树，则其 rchild 域指示其右孩子，否则令 rchild 指示其后继。
>
> 还需在结点结构中增加两个标志域 LTag 和 RTag。LTag=0 时，lchild 域指示结点的左孩子，LTag=1 时，lchild 域指示结点的前驱；RTag=0 时，rchild 域指示结点的右孩子，RTag=1 时，rchild 域指示结点的后继。
>
> <!-- more -->
>
> 以这种结点结构构成的二叉链表作为二叉树的存储结构，叫做线索链表，其中指向结点前驱和后继的指针叫做线索，加上线索的二叉树称为线索二叉树。
>
> 对二叉树以某种次序遍历使其变为线索二叉树的过程叫做线索化。
>
> 若对二叉树进行中序遍历，则所得的线索二叉树称为中序线索二叉树，线索链表称为为中序线索链表。
>
> 线索二叉树是一种物理结构。
>
> ——Wikipedia. 二叉树

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Tbt1.jpg)

## 节点结构

```JavaScript
class BithrNode{
  constructor(value){
    this.val = value
    this.lchild = null
    this.rchild = null
    this.LTag = 0
    this.RTag = 0
  }
}
```

## 线索化(中序)

```JavaScript
class BithrNode{
  constructor(value){
    this.val = value
    this.lchild = null
    this.rchild = null
    this.LTag = 0
    this.RTag = 0
  }
}

class BithrTree{
  constructor(tree){        // tree 是以二叉链表存储的二叉树，tree指向根节点
    this.head = new BithrNode

    this.head.RTag = 1
    this.head.lchild = tree

    this.inThreading(tree)
  }

  inThreading(root){
    let preNode = this.head

    fn(root)

    this.head.rchild = preNode
    preNode.RTag = 1
    preNode.rchild = this.head

    function fn(root) {
      if(root === null) return

      fn(root.lchild)

      if(root.lchild === null){
        root.LTag = 1
        root.lchild = preNode
      }
      if(preNode.rchild === null){
        preNode.RTag = 1
        preNode.rchild = root
      }

      preNode = root

      fn(root.rchild)
    }
  }
}
```

前序与后序的线索化类似

## 遍历

对线索二叉树进行遍历，其实就等于是操作一个双向链表结构

```JavaScript
function inorderTraverseThr(head) {
  const result = []

  let root = head.lchild
  while(root !== head){

    while(root.LTag === 0){   // 中序序列第一个节点
      root = root.lchild
    }

    result.push(root.val)

    while(root.RTag === 1 && root.rchild !== head){
      root = root.rchild
      result.push(root.val)
    }

    root = root.rchild
  }

  return result
}
```

如果所用的二叉树需经常遍历或查找节点时需要某种遍历序列中的前驱和后继，那么采用线索二叉链表的存储结构就是非常不错的选择

## 参考

[1\][程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/)

[2\][Wikipedia. 二叉树.](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%8F%89%E6%A0%91)
