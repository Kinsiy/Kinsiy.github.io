---
title: 图论 - 最小生成树
mathjax: true
date: 2021-12-17 20:15:58
tags: ["Prim算法","Kruskal算法"]
categories: 
    - ["数据结构", "图"]
description:
photos:
---


> **最小生成树**是一副[连通](https://zh.wikipedia.org/wiki/连通图)[加权无向图](https://zh.wikipedia.org/wiki/图_(数学))中一棵权值最小的[生成树](https://zh.wikipedia.org/wiki/生成树)。
>
> 在一给定的无向图 $G = (V,E)$中，$(u,v)$代表连接顶点$u$与顶点$v$的边(即$(u,v)\in E$)，而$w(u,v)$代表此边的权重，若存在$T$为$E$的字迹(即$T \subset E$)且$(V,T)$为树，使得
> $$
> w(T) = \sum_{\mathclap{(u,v)\in T}} w(u,v)
> $$
> 的 $w(T)$最小，则此$T$为$G$的最小生成树
>
> 最小生成树其实是最小权重生成树的简称
>
> ——Wikipedia. 最小生成树

<!--more-->

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/UntitledDiagram92.png)

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/UntitledDiagram93.png)


## 普里姆(Prim)算法


**普里姆算法**（Prim's algorithm），可在**加权连通图**里搜索最小生成树。


{% note warning %}


简单实现，核心思想很简单，从任一顶点出发，每次选择所有顶点指向未选中顶点的最小权邻接边, 直至所有顶点军被选中即可. 图定义见附录


{% endnote %}


```javascript
// 优先队列节点
class QueueNode {
  constructor(vertex, key) {
    this.vertex = vertex; // 顶点下标
    this.key = key; // 权
  }
}


function primsMst(Graph) {
  const nodes = [...Graph.nodes.values()]; // 图所有节点的数组形式
  const mstVertex = new Array(Graph.nodes.size).fill(false); // 标志已经遍历过的顶点
  const queueNodes = Array.from(
    { length: Graph.nodes.size },
    (_, i) => new QueueNode(i, Infinity)
  ); // 优先队列节点数组


  const parent = new Array(Graph.nodes.size).fill(-1); // 以下标标识边，parent[n] → n


  mstVertex[0] = true; // 初始化，从下标为 0 的顶点开始生成
  queueNodes[0].key = 0; // 初始化，队首节点key(weight)最小


  const queue = [...queueNodes]; // 优先队列


  // 当优先队列不为空
  while (queue.length !== 0) {
    const curQueueNode = queue.shift(); // 出队key(weight)最小的节点


    mstVertex[curQueueNode.vertex] = true; // 此顶点选中为最小生成树顶点(标志遍历)


    // 遍历此顶点的邻接顶点
    for (const [adjacent, weight] of nodes[curQueueNode.vertex].getAdjacents()) {
      const curIndex = nodes.indexOf(adjacent); // 获取邻接顶点在顶点数组中的下标
      if (mstVertex[curIndex]) continue; // 邻接顶点在选中最小生成树顶点中(已遍历过)


      if (queueNodes[curIndex].key <= weight) continue;
      queueNodes[curIndex].key = weight; // 更新优先队列节点key(weight)
      queue.sort((a, b) => a.key - b.key); // 排序优先队列，保证在队首的是已选出的连通子图里的顶点
      parent[curIndex] = curQueueNode.vertex; // 记录邻接顶点权值最小的边
    }
  }
  for (let i = 1; i < parent.length; i++) {
    console.log(nodes[parent[i]].value + " - " + nodes[i].value + " - " + queueNodes[i].key);
  }
}


const graph = new Graph(Graph.UNDIRECTED);


graph.addEdge(0, 1, 4);
graph.addEdge(0, 7, 8);
graph.addEdge(1, 7, 11);
graph.addEdge(1, 2, 8);
graph.addEdge(2, 8, 2);
graph.addEdge(7, 8, 7);
graph.addEdge(7, 6, 1);
graph.addEdge(8, 6, 6);
graph.addEdge(6, 5, 2);
graph.addEdge(2, 5, 4);
graph.addEdge(2, 3, 7);
graph.addEdge(3, 4, 9);
graph.addEdge(3, 5, 14);
graph.addEdge(4, 5, 10);

primsMst(graph);

// 0 - 1 - 4
// 0 - 7 - 8
// 5 - 2 - 4
// 2 - 8 - 2
// 7 - 6 - 1
// 6 - 5 - 2
// 2 - 3 - 7
// 3 - 4 - 9
```


## 克鲁斯克尔(Kruskal)算法


克鲁斯克尔算法要求使用 [图论 - 存储结构 - Ⅱ](https://kinsiy.github.io/Graph-store-2/) 中介绍的边集数组存储图数据。


1. 新建图 $G$,$G$中拥有原图中相同的节点，但没有边
2. 将原图中所有的边按权值从小到大排序
3. 从权值最小的边开始，如果这条边连接的两个节点于图$G$中不在同一个连通分量中，则添加这条边到图$G$中
4. 重复3，直至$G$中所有的节点都在同一个连通分量中


```JavaScript
function KruskalMST(vertexs, edges) {
  edges.sort((a, b) => a[2] - b[2]);


  const parent = Array(vertexs.length).fill(0);
  const result = [];


  let src, dest;
  for (let i = 0; i < edges.length; i++) {
    src = find(parent, edges[i][0]);
    dest = find(parent, edges[i][1]);
    if (src != dest) {
      parent[src] = dest;
      result.push(edges[i]);
    }
  }
  return result;


  function find(parent, index) {
    while (parent[index] > 0) {
      index = parent[index];
    }
    return index;
  }
}


const vertexs = [0, 1, 2, 7, 8];
const edges = [
  [0, 1, 4],
  [0, 7, 8],
  [1, 7, 11],
  [1, 2, 8],
  [7, 8, 7],
  [7, 6, 1],
  [2, 8, 2],
  [2, 3, 7],
  [2, 5, 4],
  [8, 6, 6],
  [6, 5, 2],
  [3, 5, 14],
  [3, 4, 9],
  [4, 5, 10],
]; // [[src,dest,weight]]


console.log(KruskalMST(vertexs, edges)); //[[7, 6, 1], [2, 8, 2], [6, 5, 2], [0, 1, 4], [2, 5, 4], [2, 3, 7], [0, 7, 8], [3, 4, 9]]
```


## 附录


普里姆(Prim)算法中用到的图结构


```JavaScript
class Node {
  constructor(value) {
    this.value = value;
    this.adjacents = new Map(); // 邻接链表
  }


  /**
   * @description: 新增邻接节点
   * @param {Node} node 邻接节点
   * @param {Number} weight 权
   */
  addAdjacent(node, weight = Infinity) {
    this.adjacents.set(node, weight);
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
    return [...this.adjacents.entries()];
  }


  /**
   * @description: 获取某邻接节点边权重
   * @param {Node} node
   * @return {Number} 边权重
   */
  getAdjacentWeight(node) {
    return this.adjacents.get(node);
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
  addEdge(source, destination, weight) {
    const sourceNode = this.addVertex(source);
    const destinationNode = this.addVertex(destination);


    sourceNode.addAdjacent(destinationNode, weight);


    if (this.edgeDirection === Graph.UNDIRECTED) {
      destinationNode.addAdjacent(sourceNode, weight);
    }


    return [sourceNode, destinationNode, weight];
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
    let weight;


    if (sourceNode && destinationNode) {
      weight = sourceNode.getAdjacentWeight(destinationNode);
      sourceNode.removeAdjacent(destinationNode);


      if (this.edgeDirection === Graph.UNDIRECTED) {
        destinationNode.removeAdjacent(sourceNode);
      }
    }


    return [sourceNode, destinationNode, weight];
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

## 参考

[1\][程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/)

[2\][Wikipedia. 最小生成树.](https://zh.wikipedia.org/wiki/%E6%9C%80%E5%B0%8F%E7%94%9F%E6%88%90%E6%A0%91)

[3\][Wikipedia. 克鲁斯克尔算法](https://zh.wikipedia.org/wiki/%E5%85%8B%E9%B2%81%E6%96%AF%E5%85%8B%E5%B0%94%E6%BC%94%E7%AE%97%E6%B3%95)

[4\][geeksforgeeks.org. Prim’s MST for Adjacency List Representation | Greedy Algo-6](https://www.geeksforgeeks.org/prims-mst-for-adjacency-list-representation-greedy-algo-6/)

