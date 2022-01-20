---
title: 常用算法
tags: 
    - algorithm
    - interview
# article_header:
#   type: cover
#   image:
#     src: /assets/images/helloworld.jpg
---

<!-- write excerpt here -->
面试常用算法

<!--more-->

1. 排序算法（快排、选择、冒泡、堆排序、二叉排序树、桶排序）
2. DFS/BFS 也就是搜索算法，剪枝务必要学！ 学宽搜的时候学一下哈希表!
3. 树
   1. 遍历：前、中、后序遍历
   2. 二叉树（最近公共祖先）
   3. 二叉排序树（查找、生成、删除（分三种情况））
   4. 堆（二叉堆、堆排序）
   5. Trie树（即字典树）
4. 图（图论建模）
   1. 最小生成树：Kruskal、Prim
   2. [最短路径](https://www.baidu.com/s?wd=%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)：Dijkstra（`O(2*|E|+|V|*log|V|)`时间复杂度（加优先级队列优化））、Floyd、SPFA、Bellman-Ford（后两种可以解决负边问题）
   3. [连通分量](https://www.baidu.com/s?wd=%E8%BF%9E%E9%80%9A%E5%88%86%E9%87%8F&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)（其中要掌握并查集技术）：[无向图](https://baike.baidu.com/item/%E6%97%A0%E5%90%91%E5%9B%BE/1680427)G的极大连通子图称为G的**连通分量**( Connected Component)。任何[连通图](https://baike.baidu.com/item/%E8%BF%9E%E9%80%9A%E5%9B%BE/6460995)的连通分量只有一个，即是其自身，非连通的[无向图](https://baike.baidu.com/item/%E6%97%A0%E5%90%91%E5%9B%BE/1680427)有多个连通分量。
   *强[连通分量](https://www.baidu.com/s?wd=%E8%BF%9E%E9%80%9A%E5%88%86%E9%87%8F&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)tarjin*：

      - 对原图取反，从任意一个顶点开始对反向图进行逆后续DFS遍历

      - 按照逆后续遍历中栈中的顶点出栈顺序，对原图进行DFS遍历，一次DFS遍历中访问的所有顶点都属于同一强连通分量。

   4. 拓扑排序（依次pop入度为0的点并删除相应的边）、关键路径（指设计中从输入到输出经过的延时最长的逻辑路径。）
   5. [哈密尔顿](https://www.baidu.com/s?wd=%E5%93%88%E5%AF%86%E5%B0%94%E9%A1%BF&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)环：欧拉回路是指不重复的走过所有路径的回路，而哈密尔顿环是指不重复地走过所有的点，并且最后还能回到起点的回路。使用简单的深度优先搜索，就能求出一张图中所有的哈密尔顿环
   6. 欧拉回路（USACO 3.3 题1 Fence）：
   7. 二分图（匈牙利算法）(USACO 4.2 题2 stall)
5. [动态规划](https://www.baidu.com/s?wd=%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)（[背包问题](https://www.baidu.com/s?wd=%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)只是其中一种）
   1. 线性动规：最长上升子序列
   2. 区间动规：石子合并
   3. 树形动规：数字三角形
   4. 图形动规：颁奖典礼`I`
6. 分治（掌握了动规分治就好学了）：快速排序、汉诺塔
7. 贪心
8. 位运算（可以用来进行优化）