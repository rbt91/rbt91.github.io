---
layout: post
title:  "Red Black Trees"
date:   2013-09-14 19:33:39
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

New nodes are inserted as if it were an ordinary BST. Then we use __recoloring and rotations__ to make sure that red-black properties are preserved.  

By default, we set the color of new node as red. Choosing black node would violate black-height property which id difficult to fix.  

By choosing a red node, we ensure that properties 1, 2 and 5 continue to be preserved. Only 3 or 4 can be violated.

Algorithm to fixup red-black properties maintains 3-part invariant:
1. Node z is red
2. Root is black
3. If the tree violates property 3, it is because z is the root and is red. If the tree violates property 4, it is because both z and its parent are red.

```
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

