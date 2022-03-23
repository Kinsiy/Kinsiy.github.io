---
title: 链表-双向链表
date: 2021-11-25 21:52:11
tags: ['双向链表']
categories: ['数据结构','链表']
description:
photos:
---


> 双向链表，又称为双链表，是链表的一种，它的每个数据结点都有两个指针，分别指向直接后继和直接前驱。所有，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点。一般我们都构造双向循环链表
>
> ——Wikipedia. 双向链表


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/610px-Doubly-linked-list.svg.png)

<!-- more -->

### 存储结构


```javascript
class Node {
  constructor(value) {
    this.value = value;
    this.next = null;
    this.prior = null;
  }
}
```


### 操作


{% note primary %}


代码和单向链表差不多，增加了链表尾属性。选用size进行循环


{% endnote %}


```javascript
class LinkedList {
  constructor() {
    this.head = null; // first/head/root 结点
    this.tail = null;
    this.size = 0; // 结点总数
  }
  // 在单向链表末端新增结点, 返回新的链表长度
  push(...args) {
    for (const value of args) {
      const node = new Node(value);


      if (this.head) {
        this.tail.next = node;
        node.prior = this.tail;
      } else {
        this.head = node;
      }


      this.tail = node;
      this.tail.next = this.head;
      this.head.prior = this.tail;


      this.size++;
    }
    return this.size;
  }


  // 在单向链表末端删除结点, 返回结点值
  pop() {
    const popNode = this.tail;
    if (popNode) {
      this.tail = popNode.prior;


      this.tail.next = this.head;
      this.head.prior = this.tail;


      this.size--;
      return popNode.value;
    }
  }


  // 在单向链表开端新增结点, 返回新链表长
  unshift(...args) {
    for (let index = args.length - 1; index >= 0; index--) {
      const node = new Node(args[index]);
      node.next = this.head;
      this.head.prior = node;


      this.head = node;
      this.size++;
    }


    this.head.prior = this.tail;
    this.tail.next = this.head;


    return this.size;
  }


  // 在单向链表开端删除结点, 返回结点值
  shift() {
    if (this.size === 0) return;
    const head = this.head;
    const tail = this.tail;
    if (this.size !== 1) {
      this.head = head.next;


      this.head.prior = this.tail;
      this.tail.next = this.head;
    } else {
      this.head = null;
      this.tail = null;
    }
    this.size--;
    return head.value;
  }


  // 按值搜索结点，返回其索引值,未找到则返回-1
  indexOf(value) {
    let currentNode = this.head;
    for (let i = 0; i < this.size; i++) {
      if (currentNode.value === value) {
        return i;
      } else {
        currentNode = currentNode.next;
      }
    }
    return -1;
  }


  // 按索引值删除单向链表的值, 返回结点值, 不存在返回undefined
  removeByIndex(index = 0) {
    if (index === 0) this.shift();
    let currentNode = this.head;
    for (let i = 0; i < this.size; i++) {
      if (i === index) {
        currentNode.next.prior = currentNode.prior;
        currentNode.prior.next = currentNode.next;
        return currentNode.value;
      } else {
        currentNode = currentNode.next;
      }
    }
  }


  // 获取第i个元素的值, 不存在则返回undefined
  getItemByIndex(index) {
    let currentNode = this.head;
    for (let i = 0; i < this.size; i++) {
      if (i === index) {
        return currentNode.value;
      } else {
        currentNode = currentNode.next;
      }
    }
  }


  // 在第i个位置之前插入新结点, 返回新链表长, 位置比链表长大则在链表末端新增, 位置小于0，则重置为0
  insertBefore(position = 0, ...args) {
    position < 0 && (position = 0); // 重置负值


    if (position === 0) return this.unshift(...args);


    let currentNode = this.head.next;


    for (let i = 1; i < this.size; i++, currentNode = currentNode.next) {
      if (i === position) {
        for (const value of args) {
          const insertNode = new Node(value);
          currentNode.prior.next = insertNode;
          insertNode.prior = currentNode.prior;


          insertNode.next = currentNode;
          currentNode.prior = insertNode;
          this.size++;
        }
        return this.size;
      }
    }
    return this.push(...args);
  }


  // 整表删除, 不用担心循环引用垃圾回收的问题
  clear() {
    this.head = null;
    this.tail = null;
    this.size = 0;
  }
}


const kinsiyLinkedList = new LinkedList();


console.log(kinsiyLinkedList.push(4, 5, 6, 7, 8)); // 5


console.log(kinsiyLinkedList.pop()); // 8


console.log(kinsiyLinkedList.unshift(1, 2, 3)); // 7


console.log(kinsiyLinkedList.shift()); // 1


console.log(kinsiyLinkedList.removeByIndex(7)); // undefined 越界了


console.log(kinsiyLinkedList.indexOf(0)); // -1


console.log(kinsiyLinkedList.getItemByIndex(3)); // 5


console.log(kinsiyLinkedList.insertBefore(3, "kinsiy", "qing")); // 8


kinsiyLinkedList.clear();


console.log(kinsiyLinkedList); // LinkedList {head: null, tail: null, size: 0}
```


### 参考


[1\][Wikipedia. 双向链表](https://zh.wikipedia.org/wiki/%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8)

[2\][Human Who Codes. Computer science in JavaScript: Doubly linked lists](https://humanwhocodes.com/blog/2019/02/computer-science-in-javascript-doubly-linked-lists/)

