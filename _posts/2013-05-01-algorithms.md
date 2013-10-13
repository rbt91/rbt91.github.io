---
layout: post
title:  "Algorithms 101"
date:   2013-05-01 21:00:00
categories: coding
---

## Insertion Sort

Assuming A[0..i] is sorted, insert A[i+1] into its appropriate place in the sorted subarray.

This closely resembles the way some people arrange their hand of cards. Initial, unsorted array represents all cards on the table. Then you pick cards one by one and insert them into your hand such that cards in hand are always in sorted order.

```java
sort(a)
{
  for(int i=0; i<n-1; i++)
  {
    // a[0..i] is sorted at this point
    // a[i+1] needs to find its appropriate place in this subarray
    key = a[i+1];
    for(int j=i; j>=0; j--)
    {
      if(a[j] > key) a[j+1] = a[j];
      else
      {
        a[j+1] = key; 
        break;
      }
    }
  }
}
```

a more succint version can be represented as:

```java
sort(a)
{
  for(int i=1; i<n; i++)
  {
    // a[0..i-1] is sorted at this point
    // a[i] needs to find its appropriate place in this subarray
    key = a[i];
    for(int j=i; j>0 && a[j-1] > key; j--)
      a[j] = a[j-1];
    // a[j-1] is less than value of interest (key), insert key after a[j]
    a[j] = key;
  }
}
```

O(n^2) time complexity, but very efficient if the input is nearly sorted  
Stable => does not change the order of equal elements  
In-place => requires constant amount of additional memory space  
*Can sort a list as it receives it*  

[wikipedia page](https://en.wikipedia.org/wiki/Insertion_sort)

## Selection Sort

Insertion sort: A[0..i] now contains *first* i elements in sorted order. **Insert** A[i+1] in its appropriate place in this subarray.

Selection sort: A[0..i] contains *smallest* i elements in sorted order. **Select** smallest element from A[i+1..n] and place it in A[i+1] position.

Consider the hand of cards again. Now all cards are in our hand to start with but unordered. We select the smallest card and move it into first position. Then we select the smallest from remaining, and move it into second place and so on.

Whereas insertion sort picks up cards one by one, selection sort starts with all cards in hand.

```java
sort(a)
{
  for(int i=0; i<n-1;i++)
  {
    smallest = i;
    for(int j=i+1; j<n; j++)
    {
      if(a[j] < a[smallest])
        smallest = j;
    }
    swap(i,smallest);
  }
}
```

O(n^2) time complexity  
in-place => output overwrites original data structure, requires contant extra storage space  
non-stable => order of equal elements is not maintained  

### Stable version

Selection sort can be implemented as stable sort, if instead of swapping the elements, each intervening elements is moved one place and smallest elements is then inserted at ith place. This is useful if data structure supports efficient insertion/deletion, like a linked list.  
  
[on wikipedia](https://en.wikipedia.org/wiki/Selection_sort)

## Bubble Sort

Make n passes over the list. In each pass, compare adjacent elements and swap their positions if they are not in the right order.

```java
sort(a)
{
  for(int i=0; i<n-1; i++)
  {
    for(int j=1; j<n-i; j++)
    {
      if(a[j-1] > a[j]) swap(j-1,j);
    }
  }
}
```

After each iteration, one more biggest element will be moved to the right side of list. Notice that inner loop needs to run ```n-i``` times only as after each pass i, i biggest elements have been placed in their final position and hence can be ignored in further passes.  

To optimize further, we can break out of loop, if no swapping is required in entire pass; ie. list is already sorted by this time.  

A more complex optimization: after every pass, the elements *after the last swap* are already in sorted order and can be skipped by inner loop.  

O(n^2) time complexity  
Ability to detect if a list is sorted in O(n)  
In-place  
Stable  

[wikipedia page](https://en.wikipedia.org/wiki/Bubble_sort)

## Heaps

Heap data structure is a nearly complete binary tree where every node is smaller than its parent node. Hence, root contains the largest value.  

Though binary trees are generally represented as linked data structures, heaps lend themselves to a much simpler representation as an array. For a node at index i, its children can be found at 2i and 2i+1.  

### Maintaining a heap

Given a heap, when a new element is added at the root, root element is no longer in its correct position. It needs to be sifted down the tree to move it into its correct position.

```java
// consider the binary subtree t rooted at i
// left and right subtrees of t are heaps
// but t itself is not a heap as root of t, a[i] may be smaller than its children
// this method lets value at a[i] "sift down" into the heap
// so that t becomes a heap
siftdown(a,i)
{
  left = 2*i;
  right = left+1;
  
  // determine largest amongst root, left and right
  largest = i;
  
  if(left <= a.heapsize && a[left] > a[largest])
    largest = left;
  
  if(right <= a.heapsize && a[right] > a[largest])
    largest = right;
    
  if(largest == i) return; // binary tree at i is a heap
  
  // swap root with largest element
  // i is now a heap except one subtree may not be a heap anymore
  swap(i, largest);
  
  // use same method to convert subtree into a heap
  siftdown(a,largest);
}

/* Iterative version */
...

```

Running time of this algorithm is propotional to height of tree. Since height of binary tree if O(lg n), complexity of above algorithm is also O(lg n).  
  
Similarly if a new element is added at the end, it needs to be sifted up the tree to make it a heap again.  

```java
SiftUp(a,i)
{
}
```

### Building a heap

Another important property of heap is that it's a nearly complete binary tree.  
In a nearly complete binary tree, all elements in right half of array represent the leaves.  
Each leaf is already a 1-element heap. So we just need to convert left half of array into a heap.

```java
// given an unordered array, convert it into a heap
// since all leaf nodes are already a heap, 
// we need to call siftdown on each internal parent node.
heapify(a)
{
  int n = a.length;
  
  for(int i=n/2-1; i>=0; i--)
  {
    siftdown(a,i);
  }
}
```

Total complexity of building a heap is O(n)*O(lg n) = O(n.lg n)  
This upper bound, though correct, is not asymptotically tight:  


> An n element heap has height lg n.  
> nh = Number of nodes at height h = n/(2^h)  
> So for each level l, siftdown will be called nh times where each siftdown is O(l)  
> Integrating l=0 -> lg n, we see that tighter bound for heapify is O(n)  
> (refer CLRS p.157)  

Heap sort algorithm first converts an array into heap. This leaves the largest element in A[0]. Sorting works as follows:  
Swap first i.e. largest element to its final correct position at the end.  
Ignore this element from heap i.e. Decrease heapsize by 1.  
Since new root element may be smaller than its children, use siftdown method to move it into its correct position.  

### Heap Sort

```java
heapsort(a)
{
  heapify(a);
  
  for(int i=n-1; i>0; i--)
  {
    swap(0, i);
    
    --a.heapsize;
    siftdown(a,0);
  }
}
```

First heapify is O(n). Each siftdown is O(lg n).  
O(heapsort) = O(n) + (n-1)*O(lg n) = O(n.lg n)  

## Priority Queues

Normal sorted array/list supports removing min/max in O(1) but insertion is costly O(n).  
For unsorted array/list, it's reverse; insertion is O(1) but removal is O(n).  
Heap data structure supports inserting new elements and removing min/max element in O(lg n).

```java
insert(data)
{
  // a[1..heapsize] is already a heap
  // insert new element at end and sift it up the tree
  a[++heapsize] = data;  
  
  SiftUp(a, heapsize);
  // when SiftUp completes, a[1..heapsize] is a heap
}

remove()
{
  data = a[0];
  
  // to remove a[0], replace it with last element and 
  // then sift it down to its correct position
  a[0] = a[heapsize];
  SiftDown(a, 0);
  
  return data;
}
```
  
## Maximum subsequence

Given an array, find a subarray sum of whose values is greatest.

Problem is trivial if array contains just positive numbers; whole array contains the largest sum. But problem becomes more serious when negative numbers are also present in the array.

### Brute force

Calculate sums of all possible subarrays and select the subarray with largest sum. Since an array of length n has n^2 subarrays, and calculating sum of an array is O(n), total complexity of this algorithm is O(n^3).  

```java
largest = 0;

for(i=0; i<n; i++)
  for(j=i; j<n; j++)
    // a[i..j] is the subarray of interest
    // calculate sum of all values in a[i..j]
    sum = 0;
    for(k=i; k<=j; k++)
      sum += a[k]
    
    // check if a[i..j] is the largest subarray we saw so far
    largest = max(sum, largest)
```

### Improve

Above algorithm calculates sum of each subarray independently. If we know the sum of subarray a[i..j-1], sum of array a[i..j] is simply sum of a[i..j-1] and a[j]  

```java
largest = 0;

for(i=0; i<n; i++)
  sum = 0;
  for(j=i; j<n; j++)
    // a[i..j] is the subarray of interest
    // sum contains sum of a[i..j-1] at this point; use it to get sum of a[i..j]
    sum += a[j]
    
    // check if a[i..j] is the largest subarray we saw so far
    largest = max(sum, largest)
```

So we use an important observation to reduce the complexity of brute force algorithm to O(n^2), but we're still looking at all possible subarrays.  

### Alternative improvement

Another way we can improve original brute force algorithm is by pre-processing the input.  
If we build a cumulative array where c[i] = sum of a[0..i], problem statement gets redefined to finding two indices i & j, such that c[j] - c[i-1] is largest.  

```java
// build cumulative array
c[0] = a[0];
for(i=1; i<n; i++)
  c[i] = c[i-1] + a[i];
  
// find largest subarray
largest = 0;

for(i=0; i<n; i++)
  for(j=i; j<n; j++)
    sum = c[j]-c[i-1]; // sum now contains sum of a[i..j] as in original solution
    largest = max(sum, largest)
```

O(n^2).

### Divide and Conquer

> Binary search is a solution that is looking for problems. -- Programming Pearls, Power of primitives.

If we break original array into 2 subarrays a & b, 3 possibilities exist:  
* maximum subarray is in a
* maximum subarray is in b
* maximum subarray starts in a and ends in b

First 2 can be handles using recursion, last needs some processing.

```java
find_max_subarray(a)
{
  return max(a, 0, n);
}

max(a, i, j)
{
  if(i>j) return 0; // empty array
  
  if(i==j) return max(a[i], 0); // just 1 value, handle negative values
    
  mid = (i+j)/2;
  
  leftmax = max(a,i,mid);
  rightmax = max(a, mid+1, j);
  
  // find maximum subarray that starts in a and ends in b
  midmax = ...
  
  return max(leftmax, midmax, rightmax);
}

/* finding midmax */

// max subarray starting in left half and ending at mid
largestl = 0;

sum = 0;
for(int i=mid; i>=0; i--)
  sum += a[i];
  largestl = max(largestl, sum)
  
// max subarray starting at mid and ending in right half
largestr = 0;

sum = 0;
for(int i=mid; i<n; i++)
  sum += a[i];
  largestr = max(largestr, sum)

midmax = largestl + largestr
```

Time complexity of above algorithm is O(n.lg n)  

### Dynamic programming or scanning algorithm

*Assuming that we've found the solution for a[0..i-1], how do we extend it to find the solution for a[0..i]?*  
Assuming that we know the max sum of first i-1 elements, how do we calculate max sum of i elements?  

Maximum sum of first i elements is either the max sum of first i-1 elements or that of the subarray ending at i.  
To find max sum of subarray ending at i, we use reasoning similar to second algorithm above and compute it from subarray that ends in i-1.

```java
// maxSoFar is the biggest subarray we've seen so far
maxSoFar = 0;

// maxEndingHere is the biggest subarray if current element is included
maxEndingHere = 0;

for(i=0; i<n; i++)
{
  // maxEndingHere is the max sum of subarray ending at i-1
  // find max sum of subarray ending at i if current element was to be included
  maxEndingHere = max(maxEndingHere+a[i], 0); // max sum is 0 if negative
  
  // check if this is better than current max sum
  maxSoFar = max(maxSoFar, maxEndingHere);
}
```

O(n)
