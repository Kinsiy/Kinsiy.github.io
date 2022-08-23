---
title: 图论 - 环
mathjax: true
date: 2021-12-27 18:48:19
tags: ['环']
categories: 
- ['数据结构','图']
description:
photos:
---


> 一个回路是一条非空的路径，其中第一个顶点和最后一个顶点相同。令图$G = (V,E,\empty)$，一个回路是一条非空路径$(e_1,e_2,...,e_n)$，其顶点序列为$(v_1,v_2,...,v_n,v_1)$
>
> 一个环路或简单回路是只有第一个与最后一个顶点相同的回路
>
> 回路和环的长度是它们经过的边的个数
>
> ——Wikipedia. 环(图论)


<!--more-->


## 环探测


使用深度优先遍历的方法，通过寻找一条连接当前顶点与之前顶点的边(它包含了一条后向边)，我们可以探测有向图和无向图上环的存在。


```JavaScript
class Graph{
  // ...
  isCyclic() {
    const visited = new Map(); // 是否遍历过
    const recStack = new Map(); // 是否在递归栈中


    for (const key of this.nodes.keys()) {
      visited.set(key, false);
      recStack.set(key, false);
    }


    for (const key of this.nodes.keys()) {
      if (this.isCyclicUtil(key, visited, recStack)) return true;
    }
    return false;
  }


  isCyclicUtil(key, visited, recStack) {
    if (recStack.get(key)) return true;
    if (visited.get(key)) return false;


    visited.set(key, true);
    recStack.set(key, true); // 入栈


    for (const adjcent of this.nodes.get(key).getAdjacents()) {
      if (this.isCyclicUtil(adjcent.value, visited, recStack)) return true;
    }


    recStack.set(key, false); // 出栈


    return false;
  }
}


Graph.UNDIRECTED = Symbol("undirected graph"); // 无向图
Graph.DIRECTED = Symbol("directed graph"); // 有向图


let g = new Graph(Graph.DIRECTED);


g.addEdge(0, 1);
g.addEdge(0, 2);
g.addEdge(1, 2);
g.addEdge(2, 0);
g.addEdge(2, 3);
g.addEdge(3, 3);


console.log(g.isCyclic()); // true
```


## 寻找简单环(有向图)


算法基于强连通图，能够找出图中所有的简单环，强连通图可以用[图论 - Tarjan算法](https://kinsiy.github.io/Graph-Tarjan/)得到。


{% note primary %}


英文论文(科学上网)[Finding all the elementary circuits of a directed graph.](https://www.cs.tufts.edu/comp/150GA/homeworks/hw1/Johnson%2075.PDF)


{% endnote %}


```JavaScript
function simple_cycles(G) {
  /**
   * @description: 解锁顶点
   * @param {*} thisnode 要解锁的顶点
   * @param {Set} blocked 已锁顶点集合
   * @param {Map} blockedMap 连锁顶点对照
   */
  function _unblock(thisnode, blocked, B) {
    const stack = [thisnode];
    while (stack.length !== 0) {
      const node = stack.pop();
      if (blocked.has(node)) {
        blocked.delete(node);
        if (B.has(node)) {
          stack.push(...B.get(node));
          B.delete(node);
        }
      }
    }
  }


  // G = _.cloneDeep(G) 为了不影响原图，可以复制一份
  const sccs = G.SCC();
  const result = [];


  while (sccs.length !== 0) {
    const scc = sccs.pop(); // 强连通分量(数组 [顶点值, 顶点值...])
    if (scc.length < 2) continue; // 只有一个顶点的强连通分量无环
    const H = G.subGraph(scc); // 强连通分量(图)        Graph.subGraph 见附录


    const startNode = scc.pop(); // 遍历起点
    const path = [startNode]; // 深度遍历路径


    const blocked = new Set(); // 已封锁顶点
    const closed = new Set(); // 环内顶点


    blocked.add(startNode);
    const B = new Map(); // 连锁顶点对照


    const SNAdj = [];
    for (const vertex of H.nodes.get(startNode).getAdjacents()) {
      SNAdj.push(vertex.value);
    }


    const stack = [[startNode, SNAdj]];


    while (stack.length !== 0) {
      const [thisnode, nbrs] = stack[stack.length - 1];


      if (nbrs.length !== 0) {
        // 邻接顶点(未遍历过的)不为空
        const nextNode = nbrs.pop();


        if (nextNode === startNode) {
          // 找到环
          result.push([...path]);
          path.forEach((e) => closed.add(e));
        } else if (!blocked.has(nextNode)) {
          // 邻接顶点未被封锁
          path.push(nextNode);
          const NNAdj = [];
          for (const vertex of H.nodes.get(nextNode).getAdjacents()) {
            NNAdj.push(vertex.value);
          }
          stack.push([nextNode, NNAdj]);


          closed.delete(nextNode);
          blocked.add(nextNode);
          continue;
        }
      } else {
        // 邻接顶点为空(顶点的所有邻接顶点均遍历过了)
        if (closed.has(thisnode)) {
          _unblock(thisnode, blocked, B); // 解锁顶点
        } else {
          for (const adjacent of H.nodes.get(thisnode).getAdjacents()) {
            const blockedChain = B.get(adjacent) ?? new Set();


            B.set(adjacent, blockedChain); // 连锁
          }
        }


        stack.pop();
        path.pop();
      }
    }
    G.removeVertex(startNode);
    H.removeVertex(startNode);
    sccs.push(...H.SCC());
  }
  console.log(result);
  return result;
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


simple_cycles(g4); //[[0, 3, 2], [0, 1, 2], [6, 4], [6, 4, 5], [9, 8]]
```


## 附录


```javascript
//
// 图 - 邻接表实现
//


class Graph{
  // ...
  subGraph(vertexKeyList) {
    const subG = new Graph();
    for (const key of vertexKeyList) {
      let vertex = this.nodes.get(key);
      subG.addVertex(key);
      for (const adjacent of vertex.getAdjacents()) {
        if (vertexKeyList.includes(adjacent.value)) {
          subG.addEdge(key, adjacent.value);
        }
      }
    }
    return subG;
  }
}


```


## 参考


[1\][GeeksforGeeks. Detect Cycle in a Directed Graph.](https://www.geeksforgeeks.org/detect-cycle-in-a-graph/?ref=gcse)

[2\][NetworkX. Source code for networkx.algorithms.cycles](https://networkx.org/documentation/networkx-1.10/_modules/networkx/algorithms/cycles.html#simple_cycles)
