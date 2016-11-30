---
layout: post
title:  "Problem classification"
date:   2016-11-30 16:00:00
categories: 
---

# [Zebra puzzle](https://en.wikipedia.org/wiki/Zebra_Puzzle)

We have a fixed set of variables. These variables can be arranged in multiple different ways, but total number of combinations is finite. Only one of those combinations satisfies our result criteria.  

# Water pouring problem

We begin at an initial state and we want to reach a goal state. At every intermediate state, we can go into multiple different directions. Optionally, we also want to be able to trace our path from initial state to goal state.  

<blockquote>**Exploration or search problem**</blockquote>

<iframe width="560" height="315" src="https://www.youtube.com/embed/oGgonJ0DTG8" frameborder="0" allowfullscreen></iframe>

Concept inventory:  
* initial state  
* goal state(s)  
* expanding the frontier, breadth first search in disguise  
* explored states; to avoid unnecessary repetition  

To keep track of actions that lead to a particular state, tag on _action_ to state as well.  
To be able to trace the path from initial to final state, add indirection. Instead of frontier being a queue of states, make it a queue of paths, where path is a list of states. Last state of a path represents the state in frontier while allowing to thread them together to build the path from initial to final state.  

```
build frontier from initial state
while(frontier is not empty)
{
  pop first item from frontier
  apply set of possible actions to calculate resulting states
  for each resulting state
  {
    skip if we've already seen this state before
    if this is our goal state, return
    otherwise, add state to explored set
    add to frontier queue
  }
}
return empty state
```

# Foxes and Hens

