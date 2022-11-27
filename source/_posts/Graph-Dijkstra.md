---
title: 图论 - 最短路径(Dijkstra)
mathjax: true
date: 2022-11-27 09:52:47
tags: ["Dijkstra算法"]
categories:
    - ["数据结构", "图"]
description:
photos:
---

最短路径,是指两顶点之间经过的边上权值之和最少的路径

> 戴克斯特拉的原始版本仅适用于找到两个顶点之间的最短路径，后来更常见的变体固定了一个顶点作为源结点然后找到该顶点到图中所有其它结点的最短路径，产生一个[最短路径树](https://zh.wikipedia.org/wiki/最短路径树)
>
> 该算法解决了图  $G = (V,E)$上带权的单源最短路径问题。
>
> 具体来说，戴克斯特拉算法设置了一顶点集合$S$，在集合$S$中所有的顶点与源点$s$之间的最终最短路径权值均已确定。算法反复选择最短路径估计最小的点$u \in V - S$并将$u$加入$S$中。
>
> ——Wikipedia. 戴克斯特拉算法

<!-- more -->

## 代码实现

Dijkstra算法要求使用 [图论 - 存储结构 - Ⅰ ](/Graph-store-1) 中介绍的邻接矩阵存储图数据。

![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Dijkstra_Animation.gif)

<div style="font-style: italic;text-align: center;">由Ibmua - Work by uploader.，公有领域</div>

{% note warning %}

基于 [图论 - 存储结构 - Ⅰ ](/Graph-store-1) 的简单实现, 仅用于学习算法思想. 

{% endnote %}

```js
/**
 * @description: Dijkstra 算法
 * @param {Graph} graph 邻接矩阵存储的图
 * @param {Node} start 起始顶点
 * @param {Map<Node, Node>} path 最短路径链
 * @param {Map<Node, number>} minWeight 最短路径对应的权
 * @return {void}
 */
function dijkstra(graph, start, path, minWeight) {
  const { nodes, matrix } = graph;

  const final = new Map();	// 表示是否找到最短路径 Map<Node, 0 | 1>
  let k = nodes.indexOf(start);
  if (k === -1) return;

  // 初始化
  for (let i = 0; i < nodes.length; i++) {
    const weight = matrix[k][i] === Graph.UNBOUND ? Infinity : matrix[k][i];
    final.set(nodes[i], 0);
    minWeight.set(nodes[i], weight);
    path.set(nodes[i], start);
  }

  minWeight.set(start, 0);
  final.set(start, 1);

  for (let i = 1; i < nodes.length; i++) {
    let min = Infinity;
    for (let j = 0; j < nodes.length; j++) {
      if (final.get(nodes[j]) || minWeight.get(nodes[j]) >= min) continue;
      k = j;
      min = minWeight.get(nodes[j]);
    }

    final.set(nodes[k], 1);

    for (let j = 0; j < nodes.length; j++) {
      const weight = matrix[k][j] === Graph.UNBOUND ? Infinity : matrix[k][j];
      if (final.get(nodes[j]) || min + weight >= minWeight.get(nodes[j])) continue;

      minWeight.set(nodes[j], min + matrix[k][j]);
      path.set(nodes[j], nodes[k]);
    }
  }
}
```
### 验证

![image-20221127114233075](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/image-20221127114233075.png)

```js
const graph = new Graph(Graph.UNDIRECTED);

const start = graph.addVertex(0);

graph.addEdge(0, 1, 1);
graph.addEdge(0, 2, 5);
graph.addEdge(1, 2, 3);
graph.addEdge(1, 3, 7);
graph.addEdge(1, 4, 5);
graph.addEdge(2, 4, 1);
graph.addEdge(2, 5, 7);
graph.addEdge(3, 4, 2);
graph.addEdge(3, 6, 3);
graph.addEdge(4, 6, 6);
graph.addEdge(4, 7, 9);
graph.addEdge(4, 5, 3);
graph.addEdge(5, 7, 5);
graph.addEdge(6, 7, 2);
graph.addEdge(6, 8, 7);
graph.addEdge(7, 8, 4);

const path = new Map();
const minWeight = new Map();

dijkstra(graph, start, path, minWeight);

const end = graph.addVertex(8);
// v0 - v8
console.log("minWeight: ", minWeight.get(end));	// minWeight:  16

let shortestPath = "shortestPath: ";
let prev = end;
while (prev !== start) {
  shortestPath += prev.value + " -> ";
  prev = path.get(prev);
}
shortestPath += start.value;

console.log(shortestPath);	// shortestPath: 8 -> 7 -> 6 -> 3 -> 4 -> 2 -> 1 -> 0
```



## 参考

[1\][程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/)

[2\][Wikipedia. 戴克斯特拉算法.](https://zh.wikipedia.org/wiki/%E6%88%B4%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95)
