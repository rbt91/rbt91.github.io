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

Path"s" because we're looking at multiple paths from given source vertex to every other vertex. Moreover, between two vertices also there may be multiple shortest paths with same weight.  

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
init(G, s)
for every vertex v in G
  v.d = INF
  v.p = NULL
s.d = 0
```

These algorithms use a technique called **relaxation**. The process of _relaxing_ an edge (u,v) consists of testing whether we can improve the shortest path to v found so far by going through u. In other words, the estimate for shortest path is gradually replaced by more accurate values, eventually reaching the optimal solution.  

```
relax(u, v)
if v.d > u.d + w(u,v)
  v.d = u.d + w(u,v)
  v.p = u
```

Algorithms differ in how many times they relax each edge and in which order. They assume adjacency list representation of graph, where we also store weight with each edge to make querying that easier.  

- Bellman Ford: general case where edges can have negative weights. Can detect negative cycles.  
- Linear time: No cycles.  
- Dijkstra: faster than Bellman-Ford but no negative edges.  

## Bellman Ford

It's remarkably simple, in that it simply relaxes _all_ the edges and does this `|V|-1` times, as we know that shortest path will contain at most that many edges. With each iteration, number of vertices with correct shortest-path weight grows, from which it follows that eventually all vertices will have correctly calculated shortest path weights.  

```
BellmanFord(G, s)

init(G, s)

// relax every edge v-1 times
for i=1 to G.V.length-1  // shortest path will contain at least 1 and at most |V|-1 edges
  for every edge (u,v) in G.E
    relax(u, v)

// detect negative cycle
for every edge (u,v) in G.E
  if v.d > u.d + w(u,v)
    throw new Exception("Negative cycle detected")
```

O(VE)  

![Bellman Ford](http://i.imgur.com/OsO22xb.png)  

Proof of Bellman-Ford's correctness is two-fold:  
* if there are no negative cycles, it computes correct shortest-path weights for all vertices reachable from the source.  

With every iteration i, we find shortest paths of at most i edges starting from source vertex. Since shortest path can contain at most V-1 edges, we need V-1 iteration. After V-1 iterations, we'd have found shortest paths with at most V-1 edges. Below example illustrates this for a simple graph:  

![Passes](https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Bellman-Ford_worst-case_example.svg/330px-Bellman-Ford_worst-case_example.svg.png)

2. it can correctly check for presence of negative cycles.  

If we can still improve the estimate for some vertex even after V-1 passes over all edges, it must imply the presence of a negative cycle.  

Reference: [Wikipedia](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm)  

## Single source shortest paths in DAG

Shortest paths are always well defined in a dag, since even if there are negative-weight edges, no negative-weight cycles can exist.  

This _linear time_ algorithm starts by topologically sorting the dag to impose a linear ordering on the vertices. If dag contains a path from u to v, then u precedes v in the topological sort. We make just one pass over the vertices in the topologically sorted order. As we process each vertex, we relax each edge that leaves the vertex. O(V+E)  

```
DAG-Shortest-Paths(G, s)
init(G, s)
For every vertex u in topologically sorted ordering of G
  for all edges (u,v) in G.E
    relax(u, v)
```

![DAG shortest paths](http://i.imgur.com/eZgTl07.png)  

## Dijkstra's algorithm

Greedy algorithm for the case where all weights are non-negative. (Can contain cycles)  

Maintains a set S of vertices whose shortest-path weights have already been determined. Maintains a priority queue Q of vertices ordered by their `d` values. S starts as empty and every step adds the vertex with minimum weight estimate to S.  

```
Dijkstra(G, s)
init(G, s)

S = {}
Q = G.V

while Q is not empty
  u = Q.extractMin()
  S.insert(u)
  
  for all edges (u,v) in G.E // edges starting from u
    relax(u, v)
```

![Dijkstra](http://i.imgur.com/RnKcunX.png)  

Running time depends on implementation of min-priority queue. Using binary heap: O(E lg V)  

Notice resemblence with BFS and Prim's algorithm for finding minimum spanning tree. 
