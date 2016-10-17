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

* it can correctly check for presence of negative cycles.  

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



# All pairs shortest paths

These algorithms use adjacency matrix representation of graph. We also create a predecessor matrix P.  

_For the moment, assume that negative edges may be present but there are no negative cycles_

## Dynamic programming algorithm based on matrix multiplication

Recap of steps for developing a dynamic-programming algorithm:  
1. Characterize the structure of an optimal solution.  
2. Recursively define the value of an optimal solution.  
3. Compute the value of an optimal solution in a bottom-up fashion.  

### Structure of optimal solution

Given a shortest path from vertex i to j that passes through vertex k, the subpath from i to k is also the shortest path from i to k.  

Shortest path from i to j can be defined as the minimum of shortest paths from i to all predecessors k of j plus w(k, j).

To make sure our sub-problem is of smaller size, take into account the path length. We know that shortest path from i to j can contain at most n-1 edges. A shortest path from i to predecessors k of j will be of length one less, to account for edge from k to j.

### Recursive solution

Let l<sub>ij</sub><sup>(m)</sup> be the minimum weight of any path from i to j with at most m edges.  

When m=1, i.e. the base case:  
l<sub>ij</sub><sup>(1)</sup>  =  w(i,j)  
where weight function w is defined as weight of edge i to j if edge exists, INF otherwise  

For m>1, we can compute l<sub>ij</sub><sup>(m)</sup> as minimum of l<sub>ij</sub><sup>(m-1)</sup> and minimum weight of any path from i to j with at most m edges obtained by looking at all predecessors of j.  

![recursive solution](http://i.imgur.com/2pTA9qm.png)  

_We dont really need to find all predecessors of j, instead we can look at all vertices in graph since w(k, j) will be INF if the edge (k,j) does not exist. Weight function w(i,j) is defined as weight of edge i to j if the edge exists, INF otherwise._  
Latter equality holds because first part of equation is actually captured in second part of equation when k=j and w<sub>jj</sub> is 0.  

### Computing the shortest path weights bottom up

---
**Sidebar: Matrix multiplication**  
Suppose we wish to compute the matrix product C = A . B of two n * n matrices A and B. Then, for i,j= 1,2,3,...,n  
c<sub>ij</sub> = Sum for all k=1 till n (a<sub>ik</sub> . b<sub>kj</sub>  
![matrix multiplication](https://upload.wikimedia.org/wikipedia/commons/thumb/e/eb/Matrix_multiplication_diagram_2.svg/470px-Matrix_multiplication_diagram_2.svg.png)  

---  

Taking graph as input in the form of adjacency matrix W = (w<sub>ij</sub>), we want to compute a series of matrices L<sup>(m)</sup> = (l<sub>ij</sub><sup>(m)</sup>) where m=1,2,3,...,n-1. The final matrix L<sup>(n-1)</sup> contains the final shortest path weights for all pairs of vertices.

Observe that L<sup>(1)</sup> = W  

```
All-Pairs-Shortest-Paths(W)
L = W
for m = 2 to n-1
  L = Extend-Shortest-Paths(L, W)
return L

Extend-Shortest-Paths(L, W)
let L' be a new n*n matrix
for i=[i..n]
  for j=[i..n]
    L'(i,j) = INF
    for k=[1..n]
      L'(i,j) = MIN{ L'(i,j), L(i,k) + w(k,j) }   // this is similar to matrix multiplication
return L'
```

O(n<sup>4</sup>)  

#### Improving the running time by repeated squaring

Given a graph with n=9, instead of computing 

L1 = W  
L2 = L1+W   
L3 = L2+W  
L4 = L3+W  
..  
..  
..  
L8 = L7+W  

we can simply compute 

L1 = W  
L2 = L1.L1  
L4 = L2.L2  
L8 = L4.L4  

O(n<sup>3</sup> lg n).  Ref. CLRS p689  


## Floyd-Warshall Algorithm

Another dynamic programming algorithm with O(n<sup>3</sup>) running time that formulates recursive solution in the terms of intermediate vertices of a shortest path.  

We start by building a shortest path from i to j where all intermediate vertices of this path are in set 1 to k, where k begins at 0 and goes till n.

0. Shortest path from i to j with no intermediate vertices  
1. Shortest path from i to j with only v<sub>1</sub> as intermediate vertex  
2. Shortest path from i to j with only v<sub>1</sub>, and v<sub>2</sub> as intermediate vertices  
3. ...  
9. Shortest path from i to j with all v<sub>1</sub> to v<sub>n</sub> as intermediate vertices  

Let d<sub>ij</sub><sup>(k)</sup> be a shortest path from i to j where all intermediate vertices are in 1..k

#### base case

When k=0, meaning no intermediate vertices, only direct edges, we get d<sub>ij</sub><sup>(0)</sup> = w<sub>ij</sub>  

#### computing next state (bottom up)

Given d<sub>ij</sub><sup>(k-1)</sup>, we can formulate d<sub>ij</sub><sup>(k)</sup> by looking at vertex k as following:  
* vertex k does not fall on a shortest path from i to j, meaning d<sub>ij</sub><sup>(k)</sup> = d<sub>ij</sub><sup>(k-1)</sup>  
* vertex k does fall on a shortest path from i to j, meaning path from i to j is the combined path from i to k and k to j, i.e.  d<sub>ij</sub><sup>(k)</sup> = d<sub>ik</sub><sup>(k-1)</sup> + d<sub>kj</sub><sup>(k-1)</sup>

![floyd warshall](http://i.imgur.com/SI0SJBQ.png)

```
Floyd-Warshall(W)
D = W
for k = 1..n
  for i = 1..n
    for j = 1..n
      D(i,j) = MIN{ D(i,j), D(i,k)+D(k,j) }
return D
```

### Constructing shortest path

Above algorithm only gives shortest path weights. Additionally, we may also need to construct predecessor matrix.  

Let P<sub>ij</sub><sup>(k)</sup> be the predecessor of vertex j on a shortest path from vertex i with all intermediate vertices in 1..k  
When k=0, meaning no intemediate vertices, only direct edges  
P<sub>ij</sub><sup>(0)</sup> = i if edge (i,j) exists, NULL otherwise  

For k>=1,  
* if k falls on shortest path from i to j, predecessor of j on shortest path from i->j is same as predecessor of j on shortest path from k->j  
* if k does not fall on shortest path from i to j, predecessor of j is same as predecessor of j on shortest path from i->j with k-1

```
Floyd-Warshall-With-Predecessor-Matrix(W)
D = W

for each edge u,v in graph
  P(u,v) = v

for k = 1..n
  for i = 1..n
    for j = 1..n
      if D(i,k) + D(k,j) < D(i,j)
        D(i,j) = D(i,k) + D(k,j)
        P(i,j) = P(i,k)
return D,P
```

## Transitive Closure of a directed graph

![graph](http://i.imgur.com/uicov5B.png)  
Let W be input graph. W<sub>ij</sub> = 1 if there's an edge from i to j ![adjacency matrix](http://i.imgur.com/vuMJM4X.png)  
Let T be transitive clousre of graph. T<sub>ij</sub> = 1 if there's a *path* from i to j ![transitive closure](http://i.imgur.com/6NXwGPV.png)

Simplest way to compute transitive clousre is to assign unit weight to all edges and run Floyd-Warshall algorithm in O(n<sup>3</sup>).  
We can improve a little bit on time and space by substituting bit-wise operations in FW algorithm.  

Let t<sub>ij</sub><sup>(k)</sup> be 1 if there exists a path from i to j with all intermediate vertices in 1..k, 0 otherwise  

When k=0, t<sub>ij</sub><sup>(0)</sup> = 1 if i==j or i,j is an edge  

For k>=1, again, either k is on the path or not:    
t<sub>ij</sub><sup>(k)</sup> = t<sub>ij</sub><sup>(k-1)</sup> OR (t<sub>ik</sub><sup>(k-1)</sup> AND t<sub>kj</sub><sup>(k-1)</sup>)  
