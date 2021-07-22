---
title: "Quicksort"
date: 2021-07-22T15:54:55-04:00
draft: false
---

The [Quicksort](https://doi.org/10.1145/366622.366644) algorithm was created by [Tony Hoare](https://www.cs.ox.ac.uk/people/tony.hoare/) in 1961 and it is a very powerful algorithm to sort an array in memory until today. Here, I will describe the Quicksort algorithm, show an implementation in Python, and analyze the time complexity of this algorithm in the worst and best scenarios.

The idea behind the Quicksort is so simple: to sort an array \\(V\\) of \\(n\\) elements, it is easier to sort smaller sub-arrays of \\(V\\).
We can think of this approach as [divide-and-conquer](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm), where we break a problem into smaller sub-problems and combine their solutions to form the solution to the original problem.

The procedure that Quicksort uses to create smaller sub-problems is called partition. Given an array \\(V\\) of \\(n\\) elements, this procedure choices an element \\(v \in V\\) and produce two arrays \\(V_1\\) and \\(V_2\\) such that
all elements in \\(V_1\\) are smaller than or equal \\(v\\) and all elements in \\(V_2\\) are greater than \\(v\\). After this procedure, the element \\(v\\) will stay in its correct position in \\(V\\).
Therefore, every time Quicksort calls partition, the problem to be solved is reduced to two smaller problems.

Besides creating sub-problems, another important part of the divide-and-conquer technique is deciding when to stop to divide sub-problems and resolve them. In the Quicksort algorithm, repeating the partition procedure in the sub-problems
will eventually produce arrays of \\(1\\) element. At this point, it is not necessary to divide the sub-problem because an array with \\(1\\) element is already sorted.

We already explain how Quicksort breaks the array in smaller sub-arrays and when this procedure stop. One question is now open: How Quicksort combines two sorted sub-arrays? The trick here is that combine the solution is not necessary.
Actually, the Quicksort doesn't generate new arrays. It always works in the same array \\(V\\).
What the Quicksort does to divide the array \\(V\\) in sub-arrays is only use indexes \\(l_1\\), \\(l_2\\), \\(r_1\\), and \\(r_2\\) with \\(l_1 \le r_1 \lt l_2 \le r_2\\).
So the sub-array \\(V_1\\) is the array \\(V\\) but only between the indexes \\(l_1\\) and \\(r_1\\). The same is valid to array \\(V_2\\) but considering the indexes \\(l_2\\) and \\(r_2\\).

Using only the array \\(V\\) is part of the power of the Quicksort because this algorithm does not use extra memory. One drawback of it is that we cannot parallelize this algorithm in multiple machines or processes. At least, not with any modifications.


## Choosing \\(v\\)

As we described, the partition procedure choices an element \\(v \in V\\) to divide the array \\(V\\) into \\(2\\) sub-arrays. So, what are the best and worst values to \\(v\\)?
We will analyze it in more detail in the Run-time complexity section, but for now, it is only important for us to know that, usually, in the divide-and-conquer technique, we would like to divide a problem into sub-problems of the same size.

On one hand, the best choosing of \\(v\\) is the element that divide \\(V\\) in two sub-arrays \\(V_1\\) and \\(V_2\\) of the same size. In this case, the element \\(v\\) is called [median or p-50](https://en.wikipedia.org/wiki/Median) of \\(V\\).
On the other hand, the worst choice of \\(v\\) is one that the array \\(V_1\\) contains all, except \\(v\\), elements of \\(V\\) and the array \\(V_2\\) is an empty array. In this case, the \\(v\\) is the greatest value of \\(V\\).
Also \\(v\\) could be the smallest element of \\(V\\) and in this case, we change \\(V_1\\) with \\(V_2\\). To resolve this problem, the author suggest to choice \\(v\\) randomly.

## Python Implementation

In this implementation, we have 2 methods: `quicksort` and `partition`. The `quicksort` is responsible to call the `partition` when necessary. It happens always that the sub-array has at least one element.
It is important to remember that the `quicksort` method does not return anything, because it sorts the array `items` in place.
The `partition` is responsible to choice an element \\(v\\) of `items` and move all values that are less than or equal \\(v\\) to the left side of \\(v\\) in `items` and move all values that are greater than \\(v\\) to the right side of \\(v\\) in `items`.
Also, the `parition` finishes returning the position of \\(v\\) in `items`.

```python

import random


def quicksort(items, l=0, r=None):
    if r is None:
        r = len(items) - 1
    i = partition(items, l, r)
    if i - 1 > l:
        quicksort(items, l, i - 1)
    if i + 1 < r:
        quicksort(items, i + 1, r)


def partition(items, l, r):
    divided_idx = random.randint(l, r)
    v = items[divided_idx]
    items[divided_idx], items[l] = items[l], items[divided_idx]
    i = l + 1
    j = r
    while i <= j:
        while i <= r and items[i] <= v:
            i += 1
        while j > l and items[j] > v:
            j -= 1
        if i < j:
            items[i], items[j] = items[j], items[i]
    items[l], items[j] = items[j], v
    return j
```

## Run-time complexity

To analyze the time complexity of the Quicksort we will use the [Big-O notation](https://en.wikipedia.org/wiki/Big_O_notation) and we will consider how many times this algorithm access an element in the array.
Also, \\(T(n)\\) here is the function which calculates the amount of time to sort an array \\(V\\) of size \\(n\\).
Considering these, we will divide our analysis into the worst and the best-case scenarios.

### Worst scenario

The worst scenario happens when the partition procedure always choices \\(v\\) as the smallest or biggest element of an array \\(V\\). In this case, this procedure will produce one sub-array of size \\(|V| - 1\\) and another empty array.

As the partition procedure is always generating an array to be sorted removing only one element of the input array. This procedure will run \\(n - 1\\) times.
Furthermore, in each call to partition procedure with an array \\(V\\), one of the two nested loops runs exactly \\(|V| - 1\\) times, because \\(v\\) is the smallest or biggest element of \\(V\\).
The other nested loop runs exactly 1 time because the check-in of this loop will be not true. Also, in each run, both nested loops access an element of the array. Therefore, in total, the two nested loops access an element of the array |V| times.

In the first time that Quicksort calls the partition procedure, the array \\(V\\) has size \\(n\\). In the second time it has size \\(n-1\\). The last time it has size 2.
So
\\[T(n) = \sum_{i=1}^{n-1} n + 1 - i\\]
\\[T(n) = n + (n - 1) + (n - 2) + ... (2)\\]
\\[T(n) = \frac{(n - 1) \times (n - 2)}{2} \\]
\\[T(n) = \frac{n^2 + 2n -n - 2}{2} \\]
\\[T(n) = \frac{n^2 +n - 2}{2} \\]
\\[O(T(n)) = n^2 \\]


### Best scenario

The best scenario happens when the partition procedure always choices \\(v\\) as the p-50 element of the array. In this case, this procedure divides the array in the middle, generating two sub-arrays of the same size.
Actually, if the size of the array \\(V\\) is odd, the two sub-arrays generated have the same size. Otherwise, the size of the two sub-arrays will have a difference of \\(1\\). Without loss of generality, we will consider sub-arrays of the same size here.

In this scenario, to sort an array of \\(n\\) elements, the partition produces two arrays to be sorted in each iteration \\(i\\). Both of these arrays has \\(\frac{n}{2^i}\\) elements. This happens until the partition produces arrays of only one element.
Therefore this happens until \\(\frac{n}{2^i}=1\\), which is \\(\log_n\\) times. Besides, in each iteration \\(i\\), the partition access all elements of the array, which is a total of \\(2^i \times \frac{n}{2^i}\\) times.
So
\\[T(n) = \sum_{i=0}^{\log_n} 2^i \times \frac{n}{2^i}\\]
\\[T(n) = \sum_{i=0}^{\log_n} n\\]
\\[T(n) = (\log_n + 1) \times n\\]
\\[O(T(n)) = n \times log_n\\]
