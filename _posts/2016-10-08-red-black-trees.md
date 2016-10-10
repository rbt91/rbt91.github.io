---
layout: post
title:  "Red Black Trees"
date:   2016-10-08 16:45:39
categories: 
---

Balanced binary search tree. Other balanced binary tree schemes: AVL, 2/3 trees, Splay trees etc.  

![Red Black Tree](https://upload.wikimedia.org/wikipedia/commons/thumb/6/66/Red-black_tree_example.svg/750px-Red-black_tree_example.svg.png)

## Properties

1. Each node is either red or black (trivial)
2. All leaves, represented as NIL, are black (trivial)
3. Root is black
4. Red nodes can have only black children
5. All paths have same black height

These properties guarantee that the longest path is no more than twice as long as the shortest path, making the tree approximately height-balanced.  

To see why this is guaranteed, consider properties 4 and 5. The shortest path will consist of all black nodes. To create longer paths, we'll need to insert red nodes, but property 4 makes it impossible to insert more than 1 consecutive red nodes. Hence, longest path can be only twice as long as the shortest path.  

## Insertion

New nodes are inserted as if it were an ordinary BST. Then we use __recoloring and rotations__ to make sure that red-black properties are preserved. Insertion consists of two phases. The first phase goes down the tree from the root, inserting the new node as a child of an existing node. The second phase goes up the tree, changing colors and performing rotations to maintain the red-black properties.  

By default, we set the color of new node as red. Choosing black node would violate black-height property which id difficult to fix.  

By choosing a red node, we ensure that properties 1, 2 and 5 continue to be preserved. Only 3 or 4 can be violated.

Algorithm to fixup red-black properties maintains 3-part invariant:  
1. Node z is red  
2. Root is black  
3. If the tree violates property 3, it is because z is the root and is red. If the tree violates property 4, it is because both z and its parent are red.  

```java
FIX(T, z)
while parent is red
  if uncle is red // Case 1
    set both parent and uncle as black and grandparent as red  // maintains properties 4&5
    Fix(T, grandparent) // recursive call as grandparent may be violating property 4
  else // uncle is black 
    if z is right child  // Case 2
      z = parent and Left-rotate(z)
    set parent as black and grandparent as red  // Case 3, while loop exits
    Right-rotate(grandparent)

set root as black
```

![Fix RB tree](http://i.imgur.com/DhFDrWL.png)

## Deletions

more complicated than insertion

## Augmenting Red Black Trees

_Augmenting a basic data structure to support additional functionality_

1. Choosing right underlying data structure  
2. Determine what additional information to maintain  
3. Ensure that additional information can be maintained for basic modification operations on underlying data structure i.e. recalculating additional information is asymptotically not worse than modifying operation  
4. Develop new operations on data structure  

In context of red black trees, if the calculation of additional information for node x depends only on x, x.left and x.right, then we can maintain this information during insertion and deletion without asymptotically affecting the O(lg n) performance of these operations.

## Dynamic order statistics tree

Supports two operations:  
- Retrieve an element with given rank  
- Determine rank of an element  
_where rank is the position of element in in-order traversal of the tree, or simply, position in the ordered set_  

We augment red-black tree with additional attribute that contains the number of nodes in the subtree rooted at x, including x
itself.

![order statistics tree](http://i.imgur.com/XR9nTwe.png)  

`x.size = x.left.size + x.right.size + 1`  

Note the choice of maintaining size attribute - size can be calculated directly from direct child nodes. If we directly stored rank in each node, queries will be simpler, but inserting a new element will require updating rank for O(n) nodes, impacting the running time of insertion operation.

### Retrieve ith element

```java
select(x, i)
rank = x.left.size + 1
if (i == rank) return x
return (i<rank) ? select(x.left, i) : select(x.right, i-rank)
```

### Determine rank

```java
rank(x)
rank = x.left.size + 1
while (x != root)
  if (x is right child)
    rank += x.parent.left.size + 1
  x = x.parent
return rank
```

Need to traverse up the tree - suppose x is the node `30` in above image. Rank will be 1+1+12+1=15!  

## Interval trees

Search for overlapping intervals.  

Given two intervals i and j, exactly one of the following three properties holds:  
1. i is to the left of j -> i ends before j begins -> i.end < j.begin  
2. i is to the right of j -> i begins after j ends -> i.begin > j.end  
3. i and j overlap -> i.begin <= j.end && j.begin <= i.end  

We use a red-black tree where each node contains interval _int_ and key is the beginning of interval. Thus in-order traversal of tree lists intervals in sorted order by their beginning.  
For each node, we store 2 values: `min` which is the earliest beginning of any interval in this subtree, and `max` which is the latest end. We can ensure that this information can be calculated from direct child nodes.  

### Seach overlapping interval

```java
search(i)
x = root
return x if i overlaps with x

while x != null
  if x.left != null && x.left.max >= i.beginning               // explanation below
    x = x.left
  else
    x = x.right
  return x if i overlaps with x
  
return null
```

Given interval does not overlap with current node x. Either i ends before x begins. Or i begins after x ends. First case is trivial, we want to go down the left subtree. In second case, left substree contains all intervals which began before x. But, if left subtree also contains some interval which is ending after given interval begins, we surely have an overlap.  

Also notice that we didn't really need to store min for each subtree.  
