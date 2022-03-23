---
title: 图论 - 存储结构 - Ⅰ
date: 2021-12-07 23:01:25
tags: ["邻接矩阵", "邻接表"]
categories:
    - ["数据结构", "图"]
mathjax: true
description:
photos:
---


## 邻接矩阵


> 图的邻接矩阵 (Adjacency Matrix) 存储方式是用两个数组来表示图。一个一维数组存储图中顶点信息，一个二维数组 (称为邻接矩阵) 存储图中的边或弧的信息
>
> ——程杰. 《大话数据结构》.

设图$G$有 n 个顶点，则邻接矩阵是一个 n * n 的方阵，定义为


$$
arc[i][j] =
\begin{cases} 
1 &(v_i,v_j) \in E(G) \text{ 或} <v_i,v_j> \in E(G)  \\
0 
\end{cases}
$$
也可以用大于0的值表示边的权值
$$
arc[i][j] =
\begin{cases} 
W_\text{ij} &(v_i,v_j) \in E(G) \text{ 或} <v_i,v_j> \in E(G)  \\
\infty
\end{cases}
$$
<!-- more -->

无向图的邻接矩阵是[对称矩阵](https://zh.wikipedia.org/wiki/%E5%B0%8D%E7%A8%B1%E7%9F%A9%E9%99%A3)


{% note primary %}


所谓对称矩阵就是 n 阶矩阵元满足 $a_\text{ij} = a_\text{ji}, (0 \le i, j \le n)$。即从矩阵的左上角到右下角的主对角线为轴，由上角的元与左下角对应的元全都是相等的


{% endnote %}


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/1_lMI_7wNnNgr3rqRqLULMZQ.png)




对于有向图，定义有向图的邻接矩阵中某个元素$A_\text{ij}$ 代表：从 i 指向 j 的边的数目，则：


有向图的某个节点的入度可以通过对应的列(column) 求和得到，出度可以通过对应的行(row)求和而得。


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/1_O6lPwoKQKFPpBEH7QhxIIw.png)


### 代码实现


{% note info %}


简单实现! 重在理解，生产环境使用考虑更稳健的第三方库


{% endnote %}


```javascript
//
// 图 - 邻接矩阵实现
//
class Node {
  constructor(value) {
    this.value = value;
  }
}

class Graph {
  /**
   * @description: 图初始化
   * @param {*} edgeDirection Graph.DIRECTED 或 Graph.UNDIRECTED
   */
  constructor(edgeDirection = Graph.DIRECTED) {
    this.nodes = [];        // 顶点表
    this.matrix = [];        // 邻接矩阵
    this.edgeDirection = edgeDirection;
  }

  /**
   * @description: 向图中新增顶点
   * @param {*} value 顶点值
   * @return {Node} 新顶点或已存在顶点
   */
  addVertex(value) {
    let current = this.nodes.find((e) => e.value === value);
    if (current) {
      return current;
    }
    current = new Node(value);
    this.nodes.push(current);
    this.matrix.forEach((e) => e.push(Graph.UNBOUND));
    this.matrix.push(Array(this.matrix.length + 1).fill(Graph.UNBOUND));
    return current;
  }

  /**
   * @description: 从图中移除值为value的顶点
   * @param {*} value 顶点值
   * @return {Boolean} 移除是否成功
   */
  removeVertex(value) {
    let currentIndex = this.nodes.findIndex((e) => e.value === value);
    if (currentIndex === -1) return false;

    this.nodes.splice(currentIndex, 1);
    this.matrix.forEach((e) => e.splice(currentIndex, 1));
    this.matrix.splice(currentIndex, 1);

    return true;
  }

  /**
   * @description: 在图中创建边/弧
   * @param {*} source
   * @param {*} destination
   * @return {[Node, Node]} 邻接节点对
   */
  addEdge(source, destination, weight = 1) {
    const sourceNode = this.addVertex(source);
    const destinationNode = this.addVertex(destination);

    const sourceIndex = this.nodes.indexOf(sourceNode);
    const destinationIndex = this.nodes.indexOf(destinationNode);

    this.matrix[sourceIndex][destinationIndex] = weight;

    if (this.edgeDirection === Graph.UNDIRECTED) {
      this.matrix[destinationIndex][sourceIndex] = weight;
    }

    return [sourceNode, destinationNode];
  }

  /**
   * @description: 从图中移除边/弧
   * @param {*} source
   * @param {*} destination
   * @return {[Node, Node]} 邻接节点对
   */
  removeEdge(source, destination) {
    const sourceIndex = this.nodes.findIndex((e) => e.value === source);
    const destinationIndex = this.nodes.findIndex((e) => e.value === destination);

    if (~sourceIndex && ~destinationIndex) {
      this.matrix[sourceIndex][destinationIndex] = Graph.UNBOUND;
      if (this.edgeDirection === Graph.UNDIRECTED) {
        this.matrix[destinationIndex][sourceIndex] = Graph.UNBOUND;
      }
    }
    return [this.nodes[sourceIndex], this.nodes[destinationIndex]];
  }
}

Graph.UNBOUND = Symbol("unbound");
Graph.UNDIRECTED = Symbol("undirected graph"); // 无向图
Graph.DIRECTED = Symbol("directed graph"); // 有向图
```




## 邻接表


> 在[图论](https://zh.wikipedia.org/wiki/图论)和[计算机科学](https://zh.wikipedia.org/wiki/计算机科学)中，**邻接表(adjacency list)** 是表示了[图](https://zh.wikipedia.org/wiki/图_(数学))中与每一个[顶点](https://zh.wikipedia.org/wiki/顶点_(图论))相邻的边集的[集合](https://zh.wikipedia.org/wiki/集合_(数学))，这里的集合指的是无序集。
>
> ——Wikipedia. 邻接表


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/1_iGp2hOLcnMaKD3zE262ZAg.png)


### 代码实现


```javascript
class Node {
  constructor(value) {
    this.value = value;
    this.adjacents = new Set();        // 邻接链表
  }

  /**
   * @description: 新增邻接节点
   * @param {Node} node
   */
  addAdjacent(node) {
    this.adjacents.add(node);
  }

  /**
   * @description: 移除邻接节点
   * @param {Node} node
   */
  removeAdjacent(node) {
    this.adjacents.delete(node);
  }

  /**
   * @description: 节点是否与node邻接
   * @param {Node} node
   */
  isAdjacent(node) {
    this.adjacents.has(node);
  }

  /**
   * @description: 获取所有邻接节点
   * @return {Array} 邻接节点数组
   */
  getAdjacents() {
    return [...this.adjacents];
  }
}

//
// 图 - 邻接表实现
//
class Graph {
  /**
   * @description: 图初始化
   * @param {Symbol} edgeDirection Graph.DIRECTED 或 Graph.UNDIRECTED
   */
  constructor(edgeDirection = Graph.DIRECTED) {
    this.nodes = new Map();
    this.edgeDirection = edgeDirection;
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
      for (const node of this.nodes.values()) {
        node.removeAdjacent(current);
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

    sourceNode.addAdjacent(destinationNode);

    if (this.edgeDirection === Graph.UNDIRECTED) {
      destinationNode.addAdjacent(sourceNode);
    }

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

    if (sourceNode && destinationNode) {
      sourceNode.removeAdjacent(destinationNode);

      if (this.edgeDirection === Graph.UNDIRECTED) {
        destinationNode.removeAdjacent(sourceNode);
      }
    }

    return [sourceNode, destinationNode];
  }

  /**
   * @description: 判断两节点是否邻接
   * @param {*} source
   * @param {*} destination
   * @return {Boolean} 邻接与否
   */
  isAdjacent(source, destination) {
    const sourceNode = this.nodes.get(source);
    const destinationNode = this.nodes.get(destination);

    if (sourceNode && destinationNode) {
      return sourceNode.isAdjacent(destinationNode);
    }

    return false;
  }
}

Graph.UNDIRECTED = Symbol("undirected graph"); // 无向图
Graph.DIRECTED = Symbol("directed graph"); // 有向图
```


有向图的邻接表由于有方向，我们以顶点为弧尾来存储边表，这样很容易就可以得到每个顶点的出度。但也有时为了便于确定顶点的入度或以顶点为弧头的弧，我们可以建立一个**有向图的逆邻接表，即对每个顶点都建立一个连接为$v_i$ 为弧头的表**

## 参考

[1\][Refina Furness.  Representing a Weighted Graph with an Adjacency Matrix in JavaScript](https://reginafurness.medium.com/representing-a-weighted-graph-with-an-adjacency-matrix-in-javascript-8a803bfbc36f)  


[2\][AdrianMejia.com Graph Data Structures in JavaScript for Beginners.](https://adrianmejia.com/data-structures-for-beginners-graphs-time-complexity-tutorial/#Adjacency-List-Graph-OO-Implementation)

[3\][Github.com amejiarosario.  dsa.js-data-structures-algorithms-javascript](https://github.com/amejiarosario/dsa.js-data-structures-algorithms-javascript/tree/master/src/data-structures/graphs) 

[4\][Blog.sessionstack.com. How JavaScript works: introduction to Graphs and Trees](https://blog.sessionstack.com/how-javascript-works-introduction-to-graphs-and-trees-708da0020cf8)

[5\][程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/) 

 
