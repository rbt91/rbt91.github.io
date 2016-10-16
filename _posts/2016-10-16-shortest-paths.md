---
layout: post
title:  "Shortest Paths"
date:   2016-10-16 12:17:39
categories: 
---

Given a weighted, directed graph G, shortest path from u to v is defined as any path p which begins at u and ends at v and has minimum weight, where weight of a path is defined as the sum of weights of its constituent edges. Weights can represent distances, time, cost, or any quantity that accumulates linearly along a path.  

The breadth-first-search algorithm is a shortest-paths algorithm that works on unweighted graphs, that is, graphs in which each edge has unit weight.  

### Variants:  
- Single source shortest path: shortest path from given source vertex to every other vertex  
- Single destination shortest path: shortest path to a given destination vertex from every other vertex. It's basically the same problem as single source shortest path with edges reversed.  
- Single pair shortest path: shortest path from given source vertex to given destination vertex. basically the same problem as single source shortest path with given source and picking path that ends at given destination vertex. No better known algorithm.  
- All pairs shortest path: Though same problem as single source shortest path run multiple times, once for every vertex, better algorithms available.  

# Single source shortest paths

Path**s** because multiple paths are possible.  

## Optimal substructure of shortest path

Given a shortest path from vertex u to v that goes through vertices i and j, the subpath from vertex i to j is also the shortest path from i to j.  

This optimal substructure indicates possible application of dynamic programming and greedy algorithms. Dijkstra's greedy, Floyd-Warshall is DP.  

## Cycles
### Negative cycles

If path from u to v can go through a negative cycle, shortest path is not defined.  
![negative weight cycle](http://i.imgur.com/zhM5pDs.png)

### Positive cycles

Shortest path can't contain positive weight cycle. As for every path that contains positive weight cycle, there exists another _shorter_ path by excluding the weight cycle.  

Therefore, we can assume that shortest paths are simple paths, i.e. no cycles. Thus we can focus on paths with at most V-1 edges.  

## Relaxation

For every vertex v, we maintain two properties:  
v.d = shortest-path estimate (for path from source to v)  
v.p = predecessor  

```
init(G, root)
for every vertex v in G
  v.d = INF
  v.p = NULL
root.d = 0
```

These algorithms use a technique called **relaxation**. The process of _relaxing_ an edge (u,v) consists of testing whether we can improve the shortest path to v found so far by going through u.  

```
relax(u, v)
if v.d > u.d + w(u,v)
  v.d = u.d + w(u,v)
  v.p = u
```

Algorithms differ in how many times they relax each edge and in which order.
