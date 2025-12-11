---
title: "Binary Search"
date: 2025-12-11T13:11:25-05:00
draft: false
tags: ["Big tech", "FAANG", "MAANG", "Interview", "Binary Search", "C++"]
---

One of the most fundamental algorithms that appears in interviews at big tech companies is [Binary Search](https://en.wikipedia.org/wiki/Binary_search). Of course, nobody will ask you to implement binary search directly. However, as [Jon Bentley](https://en.wikipedia.org/wiki/Jon_Bentley_\(computer_scientist\)) cited in his famous book [Programming Pearls](https://www.amazon.com/Programming-Pearls-2nd-Jon-Bentley/dp/0201657880), only a small percentage of people he asked—10% to be precise—were able to code this algorithm without any errors.

Here, I will show you a correct implementation of binary search, its variants that appear in interviews, and [LeetCode](https://leetcode.com/) problems that you can solve using these techniques.

# Binary Search

Given an array of items, we can find the position of a specific item linearly by iterating over the array, comparing each item with the desired item, and returning the position when the current item equals the desired item.

We must also consider the case where the desired item is not in the array; in this case, we can return any value. Since we are talking about positions in an array, we can return -1 if the item is not present. This is a very simple concept and implementation. However, if the array is sorted, we can do better.

We can find the position of an item in logarithmic time (O(log n)), where n is the number of items in the array, when the array is sorted.

To do this, we repeatedly compare the item we are looking for with the middle element of the array. If the middle element is the one we are looking for, we have our result. Otherwise, we need to update the range of the array we are considering accordingly.

If the element we are looking for is greater than the middle element, we need to search in the higher part of the array. Otherwise, we need to search in the lower part of the array.

What we just described is the Binary Search algorithm. Below is the implementation of the binary search algorithm in C++:

```c++
template <typename T>
int binary_search(std::vector<T>& items, T item) {
    int low = 0;
    int high = items.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (items[mid] == item) {
            return mid;
        }
        if (items[mid] < item) {
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return -1;
}
```

This is less than 20 lines of code that you need to understand very well if you want to be successful in an interview. Note that I said "understand," not "memorize."

For instance, the line:

```c++
int mid = low + (high - low) / 2;
```

could be implemented in different ways, such as:

```c++
int mid = (low + high) / 2;
```

Can you explain why the first implementation is better than the second?

The answer is [integer overflow](https://en.wikipedia.org/wiki/Integer_overflow). When we add two integer numbers, we can exceed the maximum value allowed in the computer's representation of an int. Knowing these details and how to answer such questions puts you in a much better position compared to other candidates.

As you can see, even a small detail can demonstrate your expertise and experience. All these small details together help you get the position you want.

Despite these details, as I said, nobody will ask you to implement binary search directly. However, there are some important variants of the binary search algorithm that appear as part of problem-solving. I will describe four of them in the next sections.

# Finding the Smallest Index with Duplicates

Let's consider we have a sorted array with duplicate elements, such as [10, 20, 30, 30, 30, 40, 50]. Also, let's assume we are searching for the element 30 and we need to find the smallest position of this element in case of duplication.

With the binary search algorithm implementation shown above, we cannot guarantee we will always return the smallest position of an element in case of duplication. However, we can modify the original algorithm to answer this question correctly.

And here is a very important point: You can only make this modification if you know the binary search algorithm and its implementation well.

Thinking about the problem, instead of returning the position when we find the element in the array, we need to update our result and continue the search in the lower part of the array.

Updating the result instead of returning immediately is easy to see. You cannot return immediately because there may be duplicate elements in the array. Since we want the smallest position, we cannot just return.

So, why go to the lower part of the array instead of the higher part? The answer is because of the problem definition: we want the smallest index. So, if we find the element, in case of duplication, we are interested in the presence of this element before the current position.

The implementation in C++ could be:

```c++
template <typename T>
int binary_search(std::vector<T>& items, T item) {
    int low = 0;
    int high = items.size() - 1;
    int result = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (items[mid] < item) {
            low = mid + 1;
        } else {
            if (items[mid] == item) {
                result = mid;
            }
            high = mid - 1;
        }
    }
    return result;
}
```

# Finding the Largest Index with Duplicates

This problem is similar to the one above, but now we are interested in the higher part of the array in case of duplication. This is because we want the largest index instead of the smallest one.

In this case, the implementation in C++ could be:

```c++
template <typename T>
int binary_search(std::vector<T>& items, T item) {
    int low = 0;
    int high = items.size() - 1;
    int result = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (items[mid] > item) {
            high = mid - 1;
        } else {
            if (items[mid] == item) {
                result = mid;
            }
            low = mid + 1;
        }
    }
    return result;
}
```

# Finding the Largest Value Less Than a Given Value

This is a little different, but only slightly. Now, we are not interested in finding an element, but the largest value less than the given element. So, every time we find an element that is smaller than our desired element, we need to update our result. We also need to update the considered range of the array.

In this case, we are interested in the higher part of the array, since we are trying to find the largest value less than our element.

The implementation in C++ could be:

```c++
template <typename T>
int binary_search(std::vector<T>& items, T item) {
    int low = 0;
    int high = items.size() - 1;
    int result = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (items[mid] < item) {
            result = mid;
            low = mid + 1;
        } else {
            high = mid - 1;
        }
    }
    return result;
}
```

# Finding the Smallest Value Greater Than a Given Value

Similar to the problem above, but now we are interested in finding the smallest value greater than the desired value. So, we need to update our result when we find a larger element instead of a smaller one.

The implementation in C++ could be:

```c++
template <typename T>
int binary_search(std::vector<T>& items, T item) {
    int low = 0;
    int high = items.size() - 1;
    int result = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (items[mid] <= item) {
            low = mid + 1;
        } else {
            result = mid;
            high = mid - 1;
        }
    }
    return result;
}
```

# LeetCode Problems

Here are some interesting problems to solve on LeetCode using the techniques explained in this post:

- [35. Search Insert Position](https://leetcode.com/problems/search-insert-position/)
- [69. Sqrt(x)](https://leetcode.com/problems/sqrtx)
- [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
- [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)
- [81. Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/)
- [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array)
- [154. Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii)

# References
- [Binary Search]()
- [Binary Search Lecture notes - Carnegie Mellon University](https://www.cs.cmu.edu/~15122-archive/n17/lec/06-binsearch.pdf)
- [Integer overflow](https://en.wikipedia.org/wiki/Integer_overflow)
- [Jon Bentley: Computer Scientist, Author of Programming Pearls](https://en.wikipedia.org/wiki/Jon_Bentley_\(computer_scientist\))
- [Programming Pearls](https://www.amazon.com/Programming-Pearls-2nd-Jon-Bentley/dp/0201657880)

