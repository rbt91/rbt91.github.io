---
layout: post
title:  "C++ arrays"
date:   2016-07-21 12:37:00
tags:   coding
---

## Array to Pointer decay  

```int a[n]``` is equivalent to ```int *p```  
where ```p``` points to the first element of array ```a```.

Anywhere a `T*` is required, you can provide a `T[n]`, and the compiler will silently provide that pointer:

                      +---+---+---+---+---+---+---+---+
    the_actual_array: |   |   |   |   |   |   |   |   |   int[8]
                      +---+---+---+---+---+---+---+---+
                        ^
                        |
                        |
                        |
                        |  pointer_to_the_first_element   int*


This conversion is known as "array-to-pointer decay"

The size of the array is lost in this process, since it is no longer part of the type (`T*`). Pro: Forgetting the size of an array on the type level allows a pointer to point to the first element of an array of *any* size. Con: Given a pointer to the first (or any other) element of an array, there is no way to detect how large that array is or where exactly the pointer points to relative to the bounds of the array.

### Type of pointer

As a rule of thumb, to determine the type of pointer ```p``` look at the type of first element of array. for example, here, type of (first element of) array is ```int```, hence ```p``` is a pointer to type ```int```.  

Example 1:  

```
int a[i][j];
// a is an array of length i where each element is of type: another array of length j where each element is of type int  

int (*p)[j]; 
// p has to point to first element of a, which is an array of ints. p is a pointer to array of length j where each element is of type int
```

Example 2:  

```
int* a[n];
// a is an array of length n where eah element is of type: pointer to int

int** p;
// p is a pointer to type 'pointer to int'
```

### & operator

One important context in which an array does *not* decay into a pointer to its first element is when the `&` operator is applied to it. In that case, the `&` operator yields a pointer to the *entire* array, not just a pointer to its first element. Although in that case the *values* (the addresses) are the same, a pointer to the first element of an array and a pointer to the entire array are completely distinct types:

    static_assert(!std::is_same<int*, int(*)[8]>::value, "distinct element type");

The following ASCII art explains this distinction:

          +-----------------------------------+
          | +---+---+---+---+---+---+---+---+ |
    +---> | |   |   |   |   |   |   |   |   | | int[8]
    |     | +---+---+---+---+---+---+---+---+ |
    |     +---^-------------------------------+
    |         |
    |         |
    |         |
    |         |  pointer_to_the_first_element   int*
    |
    |  pointer_to_the_entire_array              int(*)[8]


## Accessing array elements

### Real way: pointer arithmetic

Given a pointer `p` to the first element of an array, the expression `p+i` yields a pointer to the i-th element of the array. By dereferencing that pointer afterwards, one can access individual elements:

    std::cout << *(x+3) << ", " << *(x+7) << std::endl;

If `x` denotes an *array*, then array-to-pointer decay will kick in, because adding an array and an integer is meaningless (there is no plus operation on arrays), but adding a pointer and an integer makes sense:

       +---+---+---+---+---+---+---+---+
    x: |   |   |   |   |   |   |   |   |   int[8]
       +---+---+---+---+---+---+---+---+
         ^           ^               ^
         |           |               |
         |           |               |
         |           |               |
    x+0  |      x+3  |          x+7  |     int*

### Sytactical sugar: Indexing operator []

    &x[i]  ==  &*(x+i)  ==  x+i


## Multidimensional arrays

Named vs. Anonymous  

Named: ```int a[6][7];```  
*All* dimensions must be known at compile time. From the point of view of C++, memory is a "flat" sequence of bytes. The elements of a multidimensional array are stored in row-major order. That is, a[0][6] and a[1][0] are neighbors in memory. In fact, a[0][7] and a[1][0] denote the same element! This means that you can take multi-dimensional arrays and treat them as large, one-dimensional arrays. Though visualizing them as 2D grid can help.  

Anonymous: ```int (*p)[7] = new int[H][7];```  
All dimensions *except the first* must be known at compile time. 

### Array of pointers

You can overcome the restriction of fixed width by introducing another level of indirection.  

Named:  

    int* triangle[5];
    for (int i = 0; i < 5; ++i)
    {
        triangle[i] = new int[5 - i];
    }

    // ...

    for (int i = 0; i < 5; ++i)
    {
        delete[] triangle[i];
    }


              +---+---+---+---+---+
              |   |   |   |   |   |
              +---+---+---+---+---+
                ^
                | +---+---+---+---+
                | |   |   |   |   |
                | +---+---+---+---+
                |   ^
                |   | +---+---+---+
                |   | |   |   |   |
                |   | +---+---+---+
                |   |   ^
                |   |   | +---+---+
                |   |   | |   |   |
                |   |   | +---+---+
                |   |   |   ^
                |   |   |   | +---+
                |   |   |   | |   |
                |   |   |   | +---+
                |   |   |   |   ^
                |   |   |   |   |
                |   |   |   |   |
              +-|-+-|-+-|-+-|-+-|-+
    triangle: | | | | | | | | | | |
              +---+---+---+---+---+

Since each line is allocated individually now, viewing 2D arrays as 1D arrays does not work anymore.

Anonymous:  

    int n = calculate_five();   // or any other number
    int** p = new int*[n];
    for (int i = 0; i < n; ++i)
    {
        p[i] = new int[n - i];
    }

    // ...

    for (int i = 0; i < n; ++i)
    {
        delete[] p[i];
    }
    delete[] p;   // note the extra delete[] !


              +---+---+---+---+---+
              |   |   |   |   |   |
              +---+---+---+---+---+
                ^
                | +---+---+---+---+
                | |   |   |   |   |
                | +---+---+---+---+
                |   ^
                |   | +---+---+---+
                |   | |   |   |   |
                |   | +---+---+---+
                |   |   ^
                |   |   | +---+---+
                |   |   | |   |   |
                |   |   | +---+---+
                |   |   |   ^
                |   |   |   | +---+
                |   |   |   | |   |
                |   |   |   | +---+
                |   |   |   |   ^
                |   |   |   |   |
                |   |   |   |   |
              +-|-+-|-+-|-+-|-+-|-+
              | | | | | | | | | | |
              +---+---+---+---+---+
                ^
                |
                |
              +-|-+
           p: | | |
              +---+

## Passing arrays as parameters

**Arrays cannot be passed by value**. You can either pass them by pointer or by reference.  

Pass by pointer
---------------

Since arrays themselves cannot be passed by value, usually a pointer to their first element is passed by value instead. This is often called "pass by pointer". Since the size of the array is not retrievable via that pointer, you have to pass a second parameter indicating the size of the array (the classic C solution) or a second pointer pointing after the last element of the array (the C++ iterator solution):

    #include <numeric>
    #include <cstddef>

    int sum(const int* p, std::size_t n)
    {
        return std::accumulate(p, p + n, 0);
    }

    int sum(const int* p, const int* q)
    {
        return std::accumulate(p, q, 0);
    }

Pass by reference
-----------------

Arrays can also be passed by reference:

    int sum(const int (&a)[8])
    {
        return std::accumulate(a + 0, a + 8, 0);
    }

In this case, the array size is significant. Since writing a function that only accepts arrays of exactly 8 elements is of little use, programmers usually write such functions as templates:

    template <std::size_t n>
    int sum(const int (&a)[n])
    {
        return std::accumulate(a + 0, a + n, 0);
    }

Note that you can only call such a function template with an actual array of integers, not with a pointer to an integer. The size of the array is automatically inferred, and for every size `n`, a different function is instantiated from the template.

