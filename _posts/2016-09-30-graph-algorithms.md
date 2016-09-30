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

```C++
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

Note the use of recursion to create depth-first tree representation.  

```C++
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

