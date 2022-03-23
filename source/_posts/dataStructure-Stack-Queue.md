---
title: 栈与队列
date: 2021-11-25 21:52:44
tags: ['栈','队列']
categories: 
  - ['数据结构','栈']
  - ['数据结构', '队列']
description:
photos:
---


{% note primary %}


在Javascript中实际使用时，使用Array的push(), pop()模拟栈操作；使用push(), shift()模拟队列操作即可。


{% endnote %}


> 数组是一种类列表对象，它的原型中提供了遍历和修改元素的相关操作。JavaScript 数组的长度和元素类型都是非固定的。**因为数组的长度可随时改变，并且其数据在内存中也可以不连续，所以 JavaScript 数组不一定是密集型的，这取决于它的使用方式**
>
> ——MDN. Array


### 堆栈


> **堆栈**（英语：stack）又称为**栈**或**堆叠**，是[计算机科学](https://zh.wikipedia.org/wiki/計算機科學)中的一种[抽象资料类型](https://zh.wikipedia.org/wiki/抽象資料型別)，只允许在有序的线性资料集合的一端（称为堆栈顶端，英语：top）进行加入数据（英语：push）和移除数据（英语：pop）的运算。因而按照后进先出（LIFO, Last In First Out）的原理运作。
>
> ——Wikipedia. 堆栈

<!-- more -->


#### 实现 Ⅰ


使用与单向链表类似的结构来存储栈元素


```javascript
class StackNode {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}


class Stack {
  constructor() {
    this._top = null;
    this._size = 0;
  }


  // 压栈，返回新的栈长
  push(value) {
    const stackNode = new StackNode(value);
    stackNode.next = this._top;


    this._top = stackNode;
    this._size++;


    return this._size;
  }


  // 出栈，返回出栈元素值
  pop() {
    const popNode = this._top;


    if (popNode) {
      this._top = popNode.next;
      this._size--;


      return popNode.value;
    }
  }


  // 返回栈顶元素
  peek() {
    return this._top.value;
  }


  // 返回栈长
  size() {
    return this._size;
  }
}


const stack = new Stack();
console.log(stack.pop());        // undefined


console.log(stack.push("kinsiy"));        // 1
console.log(stack.push("stack"));        // 2
console.log(stack.push("push"));        // 3


console.log(stack.size());        // 3
console.log(stack.peek());        // push
console.log(stack.pop());        // push
console.log(stack.size());        // 2
console.log(stack.peek());        // stack
```


#### 实现 Ⅱ


使用一个内部对象存储栈元素


```JavaScript
class Stack {
  constructor(capacity = Infinity) {
    this._capacity = capacity;
    this._storage = {};
    this._size = 0;
  }


  // 压栈，返回新的栈长
  push(value) {
    if (this._size < this._capacity) {
      this._size++;
      this._storage[this._size] = value;
      return this._size;
    }
    return "Max capacity already reached. Remove element before adding a new one.";
  }


  // 出栈，返回出栈元素
  pop() {
    this._size--;
    const value = this._storage[this._size + 1];
    delete this._storage[this.size];
    if (this._size < 0) {
      this._size = 0;
    }
    return value;
  }


  // 返回栈顶元素
  peek() {
    return this._storage[this._size];
  }


  // 返回栈长
  size() {
    return this._size;
  }
}


const stack = new Stack();


console.log(stack.pop()); // undefined


console.log(stack.push("kinsiy")); // 1
console.log(stack.push("stack")); // 2
console.log(stack.push("push")); // 3


console.log(stack.size()); // 3
console.log(stack.peek()); // push
console.log(stack.pop()); // push
console.log(stack.size()); // 2
console.log(stack.peek()); // stack
```


### 队列


> **队列**，又称为**伫列**（queue），[计算机科学](https://zh.wikipedia.org/wiki/計算機科學)中的一种[抽象资料型别](https://zh.wikipedia.org/wiki/抽象資料型別)，是[先进先出](https://zh.wikipedia.org/wiki/先進先出演算法)（FIFO, First-In-First-Out）的[线性表](https://zh.wikipedia.org/wiki/线性表)。在具体应用中通常用[链表](https://zh.wikipedia.org/wiki/链表)或者[数组](https://zh.wikipedia.org/wiki/数组)来实现。队列只允许在后端（称为*rear*）进行插入操作，在前端（称为*front*）进行删除操作。
>
> 队列的操作方式和[堆栈](https://zh.wikipedia.org/wiki/堆栈)类似，唯一的区别在于队列只允许新数据在后端进行添加。
>
> ——Wikipedia. 队列


#### 实现 Ⅰ


使用类似单向链表的结构存储队列元素


```JavaScript
class QueueNode {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}
class Queue {
  constructor() {
    this._head = null;
    this._tail = null;
    this._size = 0;
  }


  // 入队，并返回新的队列长
  enqueue(value) {
    const queueNode = new QueueNode(value);


    if (this._head === null) {
      this._head = queueNode;
      this._tail = queueNode;
    } else {
      this._tail.next = queueNode;
      this._tail = queueNode;
    }
    this._size++;
    return this._size;
  }


  // 出队，返回出队元素
  dequeue() {
    const dequeueNode = this._head;
    if (dequeueNode) {
      this._head = dequeueNode.next;


      if (dequeueNode === this._tail) {
        this._tail = this._head; // 队列只有一个元素时，队尾元素也要置空
      }


      this._size--;


      return dequeueNode.value;
    }
  }


  // 返回队列长
  size() {
    return this._size;
  }


  // 返回队列是否包含某一元素
  contains(value) {
    let currentNode = this._head;
    while (currentNode) {
      if (currentNode.value === value) {
        return true;
      }
      currentNode = currentNode.next;
    }
    return false;
  }
}


const queue = new Queue();


console.log(queue.dequeue()); // undefined


console.log(queue.enqueue("kinsiy")); // 1
console.log(queue.enqueue("stack")); // 2
console.log(queue.enqueue("push")); // 3


console.log(queue.size()); // 3
console.log(queue.contains("kinsiy")); // true
console.log(queue.dequeue()); // kinsiy
console.log(queue.size()); // 2
console.log(queue.contains("kinsiy")); // false
```


#### 实现 Ⅱ


使用一个内部对象存储队列元素


```JavaScript
class Queue {
  constructor(capacity = Infinity) {
    this._capacity = capacity;
    this._storage = {};
    this._head = 0;
    this._tail = 0;
    this._size = 0;
  }


  // 入队，并返回新的队列长
  enqueue(value) {
    if (this._size >= this._capacity) {
      return "Max capacity already reached. Remove element before adding a new one.";
    }
    this._storage[this._tail] = value;
    this._tail++;
    this._size++;


    return this._size;
  }


  // 出队，返回出队元素
  dequeue() {
    const value = this._storage[this._head];
    delete this._storage[this._head];
    if (this._head < this._tail) {
      this._head++;
    }
    return value;
  }


  // 返回队列长
  size() {
    return this._size;
  }


  // 返回队列是否包含某一元素
  contains(value) {
    for (let i = this._head; i < this._tail; i++) {
      if (this._storage[i] === value) return true;
    }
    return false;
  }
}


const queue = new Queue();


console.log(queue.dequeue()); // undefined


console.log(queue.enqueue("kinsiy")); // 1
console.log(queue.enqueue("stack")); // 2
console.log(queue.enqueue("push")); // 3


console.log(queue.size()); // 3
console.log(queue.contains("kinsiy")); // true
console.log(queue.dequeue()); // kinsiy
console.log(queue.size()); // 2
console.log(queue.contains("kinsiy")); // false
```


### 参考


[1\][Wikipedia. 堆栈](https://zh.wikipedia.org/wiki/%E5%A0%86%E6%A0%88)


[2\][Wikipedia. 队列](https://zh.wikipedia.org/wiki/%E9%98%9F%E5%88%97)

[3\][Jorge Moller. Medium.com. Javascript Data Structures: Stack and Queue](https://medium.com/@_jmoller/javascript-data-structures-stacks-and-queues-ea877d72a5f9)
