---
title: 图论 - 存储结构 - Ⅱ
date: 2021-12-08 20:41:46
mathjax: false
tags: ['十字链表','邻接多重表','边集数组']
categories:
  - ["数据结构", "图"]
description:
photos:
---


在图论 - Ⅱ 中学习了图的邻接矩阵及邻接表实现，本部分学习十字链表优化有向图的存储结构、邻接多重表优化无向图的存储结构，以及边集数组的使用


## 十字链表


**十字链表 (orthogonal List) 就是把邻接表和逆邻接表结合起来**，解决邻接表只关心出\入度中的一种，查找另一种时必须遍历整个图才能知道的缺陷.


![首元节点结构](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/2-1Z10H10U6236.gif)


![弧节点结构](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/2-1Z10H1092b49.gif)


![有向图及其十字链表](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/2-1Z10H11122456.gif)

<!--more-->


### 代码实现


{% note info %}


上方图示结构简单实现! 重在理解，生产环境使用考虑更稳健的第三方库。


考虑 [图论 - 存储结构 - Ⅰ](https://kinsiy.github.io/Graph-store-1/)  中邻接表实现方式，可舍弃弧节点结构，顶点新增逆邻接表维护逆邻接顶点


{% endnote %}


```javascript
//
// 十字邻接表节点
//
class Node {
  constructor(value) {
    this.value = value;
    this.headLink = null; // 第一条以 this 为弧头的弧
    this.tailLink = null; // 第一条以 this 为弧尾的弧
    // 使用headLink而非firstIn, 能更好的进行链表操作；tailLink同理
  }
}


class Arc {
  constructor(tailVex, headVex) {
    this.tailVex = tailVex;
    this.headVex = headVex;
    this.headLink = null;
    this.tailLink = null;
  }
}


//
// 图 - 十字邻接表实现
//
class Graph {
  /**
   * @description: 图初始化
   */
  constructor() {
    this.nodes = new Map();
  }


  /**
   * @description: 向图中增加顶点
   * @param {*} value 顶点值
   * @return {Node} 新顶点或已存在顶点
   */
  addVertex(value) {
    if (this.nodes.has(value)) {
      return this.nodes.get(value);
    } else {
      const vertex = new Node(value);
      this.nodes.set(value, vertex);
      return vertex;
    }
  }


  /**
   * @description: 从图中移除值为value的顶点
   * @param {*} value 顶点值
   * @return {Boolean} 移除是否成功
   */
  removeVertex(value) {
    const current = this.nodes.get(value);
    if (current) {
      while (current.tailLink) {
        this.removeEdge(value, current.tailLink.headVex);
      }


      while (current.headLink) {
        this.removeEdge(current.headLink.tailVex, value);
      }
    }
    return this.nodes.delete(value);
  }


  /**
   * @description: 在图中创建边/弧
   * @param {*} source
   * @param {*} destination
   * @return {[Node, Node]} 邻接节点对
   */
  addEdge(source, destination) {
    const sourceNode = this.addVertex(source);
    const destinationNode = this.addVertex(destination);


    const arc = new Arc(source, destination);


    let curTailEdge = sourceNode;
    while (curTailEdge.tailLink) {
      curTailEdge = curTailEdge.tailLink;
    }
    curTailEdge.tailLink = arc;


    let curHeadEdge = destinationNode;
    while (curHeadEdge.headLink) {
      curHeadEdge = curHeadEdge.headLink;
    }
    curHeadEdge.headLink = arc;


    return [sourceNode, destinationNode];
  }


  /**
   * @description: 从图中移除边/弧
   * @param {*} source
   * @param {*} destination
   * @return {[Node, Node]} 邻接节点对
   */
  removeEdge(source, destination) {
    const sourceNode = this.nodes.get(source);
    const destinationNode = this.nodes.get(destination);


    let curTailArc = sourceNode;
    while (curTailArc.tailLink) {
      if (curTailArc.tailLink.headVex === destination) {
        curTailArc.tailLink = curTailArc.tailLink.tailLink;
        break;
      }
      curTailArc = curTailArc.tailLink;
    }


    let curHeadArc = destinationNode;
    while (curHeadArc.headLink) {
      if (curHeadArc.headLink.tailVex === source) {
        curHeadArc.headLink = curHeadArc.headLink.headLink;
        break;
      }
      curHeadArc = curHeadArc.headLink;
    }


    return [sourceNode, destinationNode];
  }
}
```


## 邻接多重表


针对无向图的邻接表存储方式进行优化，其核心思想是同一条边在不同节点的邻接表上用相同的边节点表示。


{% note info %}


其实前面邻接表的实现舍弃了边节点，把邻接节点存放在链表中(Set)就已经是这种方法的实现，而且是更好的实现。毕竟两个顶点已经可以唯一确定一条边。


{% endnote %}


若使用图示结构实现，由于存储的是无向图需要考虑$(iVex,jVex)$是无序偶对，使用$(iVex,jVex)$唯一确定一条无向边，而非与十字链表一般直接将弧插入到对应链表中


![首元节点的结构示意图](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/2-1Z10R04635G8.gif)


![边节点结构](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/2-1Z10R04A6457.gif)


![无向图及其邻接多重表](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/2-1Z10R04Hb26.gif)


iVex 和 jVex 是与某条边依附的两个顶点在顶点表中下标。iLink 指向依附顶点iVex 的下一条边，jLink指向依附顶点jVex的下一条边


## 边集数组


> 边集数组是由两个一维数组构成。一个是存储顶点的信息；另一个是存储边的信息，这个边数组每个数据元素由一条边的起点下标(begin)、终点下标(end)和权(weight)组成
>
> ——程杰. 《大话数据结构》


![边集数组](https://img-blog.csdnimg.cn/20200903141819409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW1pbmdidWhhbw==,size_16,color_FFFFFF,t_70)


边集数组关注的是边的集合，在边集数组中要查找一个顶点的度需要扫描整个边数组，效率并不高。更适合对边一次经行处理的操作，不适合对顶点相关的操作


## 参考


[1\][程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/)

[2\][data.biancheng.net. 数据结构图(Graph)详解](http://data.biancheng.net/graph/)

