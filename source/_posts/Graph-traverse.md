---
title: 图论 - 遍历
date: 2021-12-08 20:51:46
mathjax: true
tags: ['BFS','DFS']
categories: ['数据结构','图']
description:
photos:
---


## 图的遍历


从图中某一顶点出发访遍图中其余顶点，且使每个顶点仅被访问一次，这一过程就叫做图的遍历(Traversing Graph)


### 深度优先遍历


深度优先遍历(Depth-First-Search)，也有称为深度优先搜索，简称为DFS。


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Depth-First-Search.gif)

<!--more-->

### 代码实现


存储结构采用邻接表


```JavaScript
//
// 图 - 邻接表实现
//
class Graph{
  
        ...
  
  /**
   * @description: 深度优先遍历，生成器函数
   * @param {Node} first 第一个开始遍历的节点
   */  
  *dfs(first){
    const visited = new Map
    const visitList = [first]


    while(visitList.length !== 0){
      const node = visitList.pop()
      if(node && !visited.has(node)){
        yield node
        visited.set(node)
        visitList.push(...node.getAdjacents())
      }
    }
  }
}
```


### 广度优先遍历


广度优先遍历(Breadth-First-Search)，又称广度优先搜索，简称BFS


![](https://kinsiy-blog-img.oss-ap-southeast-1.aliyuncs.com/img/Breadth-First-Search-Algorithm.gif)


### 代码实现


```JavaScript
//
// 图 - 邻接表实现
//
class Graph{
  
        ...
  
  /**
   * @description: 深度优先遍历，生成器函数
   * @param {Node} first 第一个开始遍历的节点
   */  
  *dfs(first){
    const visited = new Map
    const visitList = [first]


    while(visitList.length !== 0){
      const node = visitList.pop()
      if(node && !visited.has(node)){
        yield node
        visited.set(node)
        visitList.push(...node.getAdjacents())
      }
    }
  }
}
```


## 参考


[1\][程杰. 大话数据结构. ](https://book.douban.com/subject/6424904/)


[2\][AdrianMejia.com Graph Data Structures in JavaScript for Beginners.](https://adrianmejia.com/data-structures-for-beginners-graphs-time-complexity-tutorial/#Adjacency-List-Graph-OO-Implementation)

[3\][Github.com amejiarosario.  dsa.js-data-structures-algorithms-javascript](https://github.com/amejiarosario/dsa.js-data-structures-algorithms-javascript/tree/master/src/data-structures/graphs)

