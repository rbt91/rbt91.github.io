---
layout: post
title:  Backtracking
date:   2016-12-17 00:13:00
categories: coding
---

General technique for solving [contraint satisfaction problems](https://en.wikipedia.org/wiki/Constraint_satisfaction_problem) like N queens, Sudoku, Map coloring and other logic puzzles. 

While these problems can also be solved using exhaustive search techniques by generating all possible combinations, time complexity in some cases is too high. So it's important to prune the search space and only  look at elements that really matter.

Solution generally looks like a vector a = {a1, a2,..., an} where this vector might represent:  
- an arrangement of n items where ai represents the i<super>th</super> element of the arrangement  
- a subset of n elements where ai is true iff i<super>th</super> element is present in subset  
- sequence of moves in a game  
- path in a graph

At each step of backtracking algorithm, we have a partial solution of first k elements. We find all possible options for k+1<super>st</super> element and try them one by one. 

Once we find a solution with n elements, we've reached our goal. We can stop and print the result, or optionally, we can continue finding remaining solutions. For backtracking to terminate, we need to guard all recursions with a static boolean like `finished`.  

## Generic structure of backtracking algorithm

```
backtrack(a, k, n)
  // a contains partial solution with k elements
  if k == n-1, solution found.
  else
    k++ // build partial solution with k+1 elements
    find all possible options for element k
    for each option
      assume option is partial solution ie. a[k] = option
      backtrack(a, k, n)
      reset option
```

## Backtracking DFS flow for 4 queens problem:  

Backtracking can be viewed as depath first search over a graph of partial solutions.  

![4queens](http://i.imgur.com/OWm0yUD.png)

