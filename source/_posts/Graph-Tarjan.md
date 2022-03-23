---
title: 图论 - Tarjan算法
mathjax: true
date: 2021-12-26 11:42:07
tags: ['Tarjan算法']
categories:
  - ['数据结构','图']
description:
photos:
---

## 定义


Tarjan算法是一个在图中查找强连通分量的算法


![](https://media.geeksforgeeks.org/wp-content/cdn-uploads/kosaraju.jpg)


{% note info %}


在有向图$G$中，如果对于每一对$v_i,v_j \in V, v_i \not = v_j$，从$v_i$到$v_j$和从$v_j$到$v_i$都存在路径，则称$G$是强连通图。有向图中的极大连通子图称作有向图的强连通分量


{% endnote %}


<!--more-->


## 代码实现


图数据结构使用[图论 - 存储结构 - Ⅰ ](https://kinsiy.github.io/Graph-store-1/)中的邻接表


```JavaScript
class Graph{
  // ...
  
  SCC() {
    const index = new Map();
    const lowLink = new Map();


    for (const key of this.nodes.keys()) {
      index.set(key, -1);
      lowLink.set(key, -1);
    }


    const stack = [];
    const result = [];


    let time = 0;


    for (let key of this.nodes.keys()) {
      if (index.get(key) === -1) {
        time = this.SCCUtil(key, index, lowLink, stack, time, result);
      }
    }
    console.log(result);
    return result;
  }


  SCCUtil(key, index, lowLink, stack, time, result) {
    index.set(key, time);
    lowLink.set(key, time);
    time++;
    stack.push(key);


    for (let adjcent of this.nodes.get(key).getAdjacents()) {
      let adjcentKey = adjcent.value;


      if (index.get(adjcentKey) === -1) {
        time = this.SCCUtil(adjcentKey, index, lowLink, stack, time, result);
        lowLink.set(key, Math.min(lowLink.get(key), lowLink.get(adjcentKey)));
      } else if (stack.includes(adjcentKey)) {
        lowLink.set(key, Math.min(lowLink.get(key), index.get(adjcentKey)));
      }
    }


    let w = Symbol("NUll");
    if (index.get(key) === lowLink.get(key)) {
      let scc = [];
      while (w !== key) {
        w = stack.pop();
        scc.push(w);
      }
      result.push(scc);
    }


    return time;
  }
}


let g4 = new Graph(Graph.DIRECTED);


g4.addEdge(0, 1);
g4.addEdge(0, 3);
g4.addEdge(1, 2);
g4.addEdge(1, 4);
g4.addEdge(2, 0);
g4.addEdge(2, 6);
g4.addEdge(3, 2);
g4.addEdge(4, 5);
g4.addEdge(4, 6);
g4.addEdge(5, 6);
g4.addEdge(5, 7);
g4.addEdge(5, 8);
g4.addEdge(5, 9);
g4.addEdge(6, 4);
g4.addEdge(7, 9);
g4.addEdge(8, 9);
g4.addEdge(9, 8);


g4.SCC(); // [[8, 9], [7], [5, 4, 6], [3, 2, 1, 0]]
```


## 参考

[1\][GeeksforGeeks. Tarjan’s Algorithm to find Strongly Connected Components.](https://www.geeksforgeeks.org/tarjan-algorithm-find-strongly-connected-components/)
