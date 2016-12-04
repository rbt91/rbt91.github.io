---
layout: post
title:  "Maximum subsequence"
date:   2013-05-01 21:00:00
categories: coding
---

Given an array, find a subarray sum of whose values is greatest.

Problem is trivial if array contains just positive numbers; whole array contains the largest sum. But problem becomes more serious when negative numbers are also present in the array.

## Brute force

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

## Improve

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

## Alternative improvement

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

## Divide and Conquer

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

## Dynamic programming or scanning algorithm

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

<script src="https://gist.github.com/rbt91/d5859654ed4298cfbe171ff00a9e69f0.js"></script>
