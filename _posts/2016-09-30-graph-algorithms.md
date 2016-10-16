---
layout: post
title:  "Graph Algorithms"
date:   2016-09-30 12:32:39
categories: 
---

# Graph representation

Maximum number of edges in a G with vertices V is V^2.  
Dense graph -> |E| is close to |V|^2 -> represented as **Adjacency Matrix**. Consumes more space but easy to query if exists edge between vertices u and v.  
Sparse graph if |E| is much less than |V|^2. Represented as **Adjacency List**. Less space.  
Choice of representation also depends on the particular algorithm.  

# BFS

Note the use of queue to arrange elements in the order in which we want to traverse them.  

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <queue>
#include <map>
#include <set>
#include <utility>

using namespace std;

template<typename V>
class Graph
{
  map<V, list<V>> adj;

public:
  void addVertex(V v)
  {
    adj[v] = list<V>{};
  }

  void addEdge(V u, V v)
  {
    adj[u].push_back(v);
  }

  void print()
  {
    for (auto i=adj.begin(); i != adj.end(); i++)
    {
      cout << i->first << ": ";
      for (auto j=i->second.begin(); j != i->second.end(); j++)
      {
        cout << *j << ", ";
      }
      cout << endl;
    }
  }

  void BFS(V s)
  {
    using F = pair<V, int /*distanceFromRoot*/>;
    queue<F> frontier;
    frontier.push(make_pair(s, 0));

    set<V> visited;

    while(!frontier.empty())
    {
      auto f = frontier.front(); frontier.pop();
      auto v = f.first;
      int w = f.second;

      visited.insert(v);

      cout << v << "(" << w << ") ";

      for (auto i = adj[v].begin(); i != adj[v].end(); i++)
      {
        if (visited.find(*i) == visited.end())
        {
          frontier.push(make_pair(*i, w+1));
        }
      }
    }
  }
};

int main()
{
  Graph<int> g;
  g.addVertex(1);
  g.addVertex(2);
  g.addVertex(3);
  g.addVertex(4);
  g.addEdge(1,2);
  g.addEdge(2,3);
  g.addEdge(3,4);
  g.addEdge(4,1);
  g.addEdge(1,4);
  g.print();

  g.BFS(1);
}
```

# DFS

DFS is used as preliminary step in many graph algorithms to first understand the structure of graph. Note the use of recursion to create depth-first tree representation.  

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <queue>
#include <map>
#include <set>
#include <utility>

using namespace std;

// Undirected, weighted graph
template<typename V>
class Graph
{
  struct E
  {
    V start, end;
    int weight;

    E(V u, V v, int w=1) : start(u), end(v), weight(w) {}
  };

  map<V, list<E>> adj;

public:
  void addVertex(V v)
  {
    adj[v] = list<E>{};
  }

  void addEdge(V u, V v, int w=1)
  {
    adj[u].push_back(E(u, v, w));
    adj[v].push_back(E(v, u, w));
  }

  void BFS(V s)
  {
    using F = pair<V, int /*distanceFromRoot*/>;
    queue<F> frontier;
    frontier.push(make_pair(s, 0));

    set<V> visited {s};

    while(!frontier.empty())
    {
      auto f = frontier.front(); frontier.pop();
      auto v = f.first;
      int w = f.second;

      cout << v << "(" << w << ") ";

      for (auto i = adj[v].begin(); i != adj[v].end(); i++)
      {
        auto v2 = i->end;
        if (visited.find(v2) == visited.end())
        {
          frontier.push(make_pair(v2, w+(i->weight)));
          visited.insert(v2);
        }
      }
    }
  }

  void DFS()
  {
    set<V> visited;
    for (auto i=adj.begin(); i != adj.end(); i++)
    {
      if (visited.find(i->first) == visited.end())
      {
        DFS(i->first, visited);
      }
    }
  }

  void DFS(V s, set<V>& visited)
  {
    cout << s << " ";
    visited.insert(s);

    for (auto i=adj[s].begin(); i != adj[s].end(); i++)
    {
      auto v = i->end;
      if (visited.find(v) == visited.end())
      {
        DFS(v, visited);
      }
    }
  }
};

int main()
{
  Graph<int> g;
  g.addVertex(1);
  g.addVertex(2);
  g.addVertex(3);
  g.addVertex(4);
  g.addEdge(1,2);
  g.addEdge(2,3,5);
  g.addEdge(3,4);
  g.addEdge(4,1);
  g.addEdge(1,4,3);
  //g.print();

  g.BFS(1);
  g.DFS();
}
```

# Connected components

A graph is **strongly connected** if every vertex is reachable from every other vertex.  

**Strongly connected components** of an arbitrary graph are the collection of sub-graphs where each sub-graph is strongly connected.  

```
|-> 2 -> 1 -> 0 -> 3 <- 4  
|-------------|  
```

Above graph has three SCC. (0, 1, 2), (3), (4). Vertices 3 & 4 are two separate SCC because 3 can be reached from 4, but not vice-versa. 

Several algorithms compute SCC in linear time. The simplest one - [Kosaraju's algorithm](https://en.wikipedia.org/wiki/Strongly_connected_component#Algorithms) - uses two passes of DFS. [Other algorithms](https://en.wikipedia.org/wiki/Strongly_connected_component#Algorithms) require only one pass by making use of one or more stacks. 

## Kosaraju's algorithm for finding SCC

It makes use of the fact that the transpose graph (the same graph with the direction of every edge reversed) has exactly the same strongly connected components as the original graph.  
Recall that DFS is used as preliminary step in many graph algorithms to first understand the structure of graph.  
1. call DFS on G  
2. compute G' by inverting all the edges  
3. call DFS on G' by choosing vertices in decreasing order of their finishing time computed in step 1

Step 1: If we call DFS on above graph, starting with 2 and then 4, we'll get two depth-first trees: 2-1-0-3 and 4.  
Step 2: Create G'  

```
|--2 <- 1 <- 0 <- 3 -> 4  
|------------^  
```

Step 3: call DFS on G', starting with vertex 4, then 2 followed by 1 and 0 and 3.
4: (4)  
2: (2, 0, 1)  
1: ()  
0: ()  
3: (3) ) is already visited  

# Minimum Spanning Tree

Given an undirected graph G=(V,E) where each edge (u,v) has weight w(u,v). Find a subset of E that connects all vertices and total weight of edges is minimized. The “minimum spanning tree” actually means “minimum-weight spanning tree.” We are minimizing the weight, not the number of edges, since all spanning trees have exactly V-1 edges.  

## Kruskal's algorithm

Consider that every vertex is its own tree. At every step, we pick the edge with minimum weight that combines two distinct trees.  
By nature, intermediate MST will consist of multiple sub-trees which will eventually join to form a single MST. It's an example of greedy algorithm because at each step we pick the edge with minimum weight.  

```
MST-KRUSKAL
1 A = empty
2 for each vertex v in G
3   MAKE-SET(v)                    // create |V| distinct trees
4 sort the edges of G into nondecreasing order by weight
5 for each edge (u,v) in G, taken in nondecreasing order by weight
6   if FIND-SET(u) != FIND-SET(v)  // if u & v are not in the same tree
7     add (u, v) to A
8     UNION(u, v)                  // combine two trees
9 return A
```

Running time O(E lg V)  
It uses a disjoint-set data structure to maintain several disjoint sets of elements. The running time depends on how we implement the disjoint-set data structure. (See section 21.3)  

## Prim's algorithm

Another greedy algorithm. Edges in A always form a signle tree unlike Kruskal's algorithm.  
We start with an arbitrary root, and at each step, grow A by picking a min weight edge which connects A to a new edge not already in A.  
During execution of algorithm, all vertices that are _not_ in A reside in a min-priority queue Q. Q is sorted using _rank_ attribute. For each vertex v, rank is the min weight of any edge connecting v to any vertex in A.  
To begin with, all vertices have rank INFINITE except the starting vertex which has rank 0.  

```
MST-PRIM(G, r /*starting vertex*/)
1 for each u in G
2   u.key = INF
3   u.parent = NIL
4 r:key = 0
5 Q = G.V                  // min priority queue of all vertices ordered by key/rank
6 while Q is not empty
7   u = EXTRACT-MIN(Q)
8   for each v in Adj(u)   // for all edges not already in A
9     if v in Q and w(u,v) < v.key  // fix rank and parent attributes
10      v.parent = u
11      v.key = w(u,v)
```

When the algorithm terminates, Q is empty and MST can be build by following the `parent` edges.  

Running time O(E lg V)  
If we build the min-priority queue using Fibonacci Heap instead of more common Binary Heap, running time improves to O(E + V lg V).  

**Fibonacci Heaps** have better _amortized_ running times for Insert, Decrease-key, and Union operations as compared to plain Binary Heaps. Though only of theoretical interest due to programming complexity.  


