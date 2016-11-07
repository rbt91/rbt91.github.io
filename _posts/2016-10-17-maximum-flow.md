---
layout: post
title:  "Maximum Flow in a graph"
date:   2016-10-17 13:33:39
categories: 
---

Every edge is labeled with _capacity_, the max amount of stuff it can carry. We call this graph as __Flow Network__. The goal is to figure out how much stuff can be pushed from given source vertex to given sink vertex.  

For every edge: capacity >= flow >= 0  
For every vertex (except source & sink): Flow in = Flow out.  

![flow network](http://i.imgur.com/gKjEXSf.png)

<blockquote>
For simplicity, we assume that there are no anti-parallel edges i.e. if there's an edge from u to v, there's no back edge from v to u. Flow networks that do contain anti-parallel edges, can be very easily converted to equiavalent networks without anti-parallel edges.  
</blockquote>
![anti-parallel](http://i.imgur.com/WrUQraG.png)

<blockquote>
We also assume single source and sink. Networks with multiple sources and sinks can be reduced to networks with single source and sink.  
</blockquote>
![multiple source-sinks](http://i.imgur.com/GFtTdhU.png)

## Greedy algorithm for calculating maximum flow

A greedy algorithm makes a sequence of myopic and irrevocable decisions, with the hope that everything somehow works out at the end. For most problems, greedy algorithms do not generally produce the best-possible solution. But it’s still worth trying them, because the ways in which greedy algorithms break often yields insights that lead to better algorithms.  

```
//start with all-zero flow
initialize f=0 for all edges
repeat:
  search for a path from source s to sink t such that every edge on this path has some left-over capacity
  // can use BFS/DFS, O(|E|)
  
  if no such path exists
    break with current flow
  else
    delta = minimum of leftover capacity of all edges in path
    increase flow by delta for all edges in path
```

Where can this greedy algorithm break? If it terminates with a path that's not the best. Can we find a counter-example? Yes, we can see that greedy algorithm returns suboptimal result if first path picked is s-v-w-t.  
![greedy](http://i.imgur.com/v1FaeJf.png)


## Ford Fulkerson algorithm a.k.a. add "undo" support

![FFA](https://youtu.be/dorq_YA6plQ?t=34m9s)

https://youtu.be/dorq_YA6plQ?t=34m9s

# Correctness of Ford-Fulkerson algorithm

## Cut

A cut is a partition of vertices of a graph into two disjoint sets A, and B with source in one set and target in the other. A cut buckets the edges of graph into 4 categories:  

1. Edges that start in A and end in A  
2. Edges that start in B and end in B  
3. Edges that start in A and end in B  
4. Edges that start in B and end in A  

Capacity or Value of a cut is defined as sum of capacity of all edges going from A to B. Edges going from B to A are exluded from capacity calculation. Different cuts of the same graph can have different capacity. For example, in below graph, {s,w}->{v,t} has capacity 3. While cut {s}->{w,v,t} has capacity 101.  

![cut](http://i.imgur.com/0ydWesA.png)

_A min-cut is a cut with minimum capacity_.

## Max-flow/Min-cut theorem

`Value of maximum flow = Capacity of min-cut`  

If f is a flow in G such that the residual network G<sub>f</sub> has no s-t path, then f is a maximum flow.  
Given a maximum flow, minimum cut can be computed in linear time using BFS/DFS.

## Edmonds-Karp algorithm

Above algorithm is not optimal as paths are chosen arbitrarily. Edmonds-Karp algorithm improves this by always choosing shortest path first. Shortest path can be found in linear time using BFS.  

## References
https://youtu.be/dorq_YA6plQ?t=14m13s  
http://theory.stanford.edu/~tim/w16/l/l1.pdf  
