---
title: 链表-单向链表
date: 2021-11-15 20:27:24
tags: ['单向链表']
categories: ['数据结构','链表']
description:
photos:
---

## 前置知识

### 链表

> 在[计算机科学](https://zh.wikipedia.org/wiki/電腦科學)中，**链表**（Linked list）是一种常见的基础数据结构，是一种[线性表](https://zh.wikipedia.org/wiki/线性表)，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的[指针](https://zh.wikipedia.org/wiki/指標_(電腦科學))(Pointer)。由于不必须按顺序存储，链表在插入的时候可以达到O(1)的[复杂度](https://zh.wikipedia.org/wiki/複雜度)，比另一种线性表[顺序表](https://zh.wikipedia.org/wiki/顺序表)快得多，但是查找一个节点或者访问特定编号的节点则需要O(n)的时间，而顺序表相应的时间复杂度分别是O(logn)和O(1)。
>
> 使用链表结构可以克服数组链表需要预先知道数据大小的缺点，链表结构可以充分利用计算机内存空间，实现灵活的内存动态管理。但是链表失去了数组随机读取的优点，同时链表由于增加了结点的指针域，空间开销比较大。
>
> --Wikipedia. 链表

<!-- more -->

### 单向链表

> **单向链表**（又名单链表、线性链表）是[链表](https://zh.wikipedia.org/wiki/链表)的一种，其特点是链表的链接方向是单向的，对链表的访问要通过从头部开始，依序往下读取。
>
> 一个单向链表的节点被分成两个部分。第一个部分保存或者显示关于节点的信息，第二个部分存储下一个节点的地址。单向链表只可向一个方向遍历。

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Link_zh.png)

### 头指针与头结点的异同

#### 头指针

- 头指针是指链表指向第一个结点的指针，若链表有头结点，则是指向头结点的指针
- 头指针具有标识作用，所以常用头指针冠以链表的名字
- 无论链表是否为空，头指针均不为空。头指针是链表的必要元素

#### 头结点

- 头结点是为了操作的统一和方便而设立的，放在第一元素的结点之前，其数据域一般无意义(也可存放链表的长度)
- 有了头结点，对在第一元素结点前插入结点和删除第一结点，其操作与其它结点的操作就统一了
- 头结点不一定是链表必须要素

## 代码实现

{% note warning %}

使用参数收集符(**...args**)，感觉使用了顺序链表(数组)，有点循环套娃的感觉

{% endnote %}

```javascript
// Singly Linked Lists
class Node {
    constructor(value) {
        this.value = value;
        this.next = null;
    }
}

class LinkedList {
    constructor() {
        this.root = null; // first/head/root 结点
        this.size = 0; // 结点总数
    }
    // 在单向链表末端新增结点, 返回新的链表长度
    push(...args) {
        for (const value of args) {
            const node = new Node(value);
            if (this.root) {
                let currentNode = this.root;
                while (currentNode && currentNode.next) {
                    currentNode = currentNode.next;
                }
                currentNode.next = node;
            } else {
                this.root = node;
            }
            this.size++;
        }
        return this.size;
    }

    // 在单向链表末端删除结点, 返回结点值
    pop() {
        let prevNode = null;
        let currentNode = this.root;

        if (currentNode && currentNode.next) {
            // 判断是不是只有一个结点
            while (currentNode && currentNode.next) {
                // 到达最后一个结点
                prevNode = currentNode;
                currentNode = currentNode.next;
            }
            prevNode.next = null;
        } else {
            this.root = null;
        }

        if (currentNode) {
            this.size--;
            return currentNode.value;
        }
    }

    // 在单向链表开端新增结点, 返回新链表长
    unshift(...args) {
        for (let index = args.length - 1; index >= 0; index--) {
            const node = new Node(args[index]);
            node.next = this.root;
            this.size++;
            this.root = node;
        }
        return this.size;
    }

    // 在单向链表开端删除结点, 返回结点值
    shift() {
        const first = this.root;
        if (first) {
            this.root = first.next;
            this.size--;
            return first.value;
        }
    }

    // 按值搜索结点，返回其索引值,未找到则返回-1
    indexOf(value) {
        for (let currentNode = this.root, i = 0; currentNode; i++, currentNode = currentNode.next) {
            if (currentNode.value === value) {
                return i;
            }
        }
        return -1;
    }

    // 按索引值删除单向链表的值, 返回结点值, 不存在返回undefined
    removeByIndex(index = 0) {
        if (index === 0) return this.shift();
        let prevNode = this.root;
        for (let currentNode = this.root.next, i = 1; currentNode; i++, currentNode = currentNode.next) {
            if (i === index) {
                prevNode.next = currentNode.next;
                this.size--;
                return currentNode.value;
            }
            prevNode = currentNode;
        }
    }

    // 获取第i个元素的值, 不存在则返回undefined
    getItemByIndex(index) {
        for (let currentNode = this.root, i = 0; currentNode; i++, currentNode = currentNode.next) {
            if (i === index) {
                return currentNode.value;
            }
        }
        return undefined;
    }

    // 在第i个位置之前插入新结点, 返回新链表长, 位置比链表长大则在链表末端新增, 位置小于0，则重置为0
    insertBefore(position = 0, ...args) {
        position < 0 && (position = 0); // 重置负值

        if (position === 0) return this.unshift(...args);

        let prevNode = this.root;
        let currentNode = this.root.next;
        let i = 1;

        for (; currentNode; i++, currentNode = currentNode.next) {
            if (i === position) {
                for (const value of args) {
                    const insertNode = new Node(value);
                    prevNode.next = insertNode;
                    insertNode.next = currentNode;
                    prevNode = insertNode;
                    this.size++;
                }
                return this.size;
            }
            prevNode = currentNode;
        }
        return this.push(...args);
    }

    // 整表删除, 不用担心循环引用垃圾回收的问题
    clear() {
        this.root = null;
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

console.log(kinsiyLinkedList.insertBefore(-5, "kinsiy", "qing")); // 8

kinsiyLinkedList.clear();

console.log(kinsiyLinkedList); // LinkedList {root: null, size: 0}
```

## 参考

[1\] [AdrianMejia. Data Structures in JavaScript: Arrays, HashMaps, and Lists. ](https://adrianmejia.com/data-structures-time-complexity-for-beginners-arrays-hashmaps-linked-lists-stacks-queues-tutorial/#Linked-Lists)

[2\] [程杰. 大话数据结构.](https://book.douban.com/subject/6424904/)

