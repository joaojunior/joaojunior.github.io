---
title: "Composition, Inheritance, and Liskov Substitution Principle"
date: 2025-04-22T11:27:45-04:00
draft: false
tags: ["Inheritance", "Composition", "Liskov Substitution Principle", "Barbara Liskov"]
---

In software development, organizing our codebase in a way that is easy to maintain is always a good practice, and this idea is not new.
Actually, I think this started in 1968 when the term [software crisis](https://en.wikipedia.org/wiki/Software_crisis) was coined. Since then, we have developed many principles to help us achieve this goal.
One person who has made many contributions to this field is [Barbara Liskov](https://pmg.csail.mit.edu/~liskov/). She is very famous for the [Liskov Substitution Principle (LSP)](https://en.wikipedia.org/wiki/Liskov_substitution_principle), but she also invented the [Abstract Data Type(ADT)](https://dl.acm.org/doi/pdf/10.1145/800233.807045), which is the core concept of classes and [object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming).
Here, I want to discuss composition compared to inheritance and how understanding the Liskov Substitution Principle can help you develop better software.

First of all, currently, it is already a common practice and advice to prefer [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance). However, this was not always true. At least not for me; when I learned about object-oriented programming in earlier 2002, I was told that [inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)) was the best approach to reuse code. I was an inexperienced software engineer at that moment, and it took me a long time to understand the inheritance issues. Barbara Liskov talked about her preference for using [composition](https://en.wikipedia.org/wiki/Object_composition) over inheritance long ago, in the late 60s or early 70s. This reminds us about the importance of learning the foundation well and reading the past to understand how we arrived at the present.

However what is precisely the problems with inheritance? As Barbara Liskov said, the problem is that inheritance is used for two completely different ideas: implementation and type hierarchy.
She mentioned that the implementation is the bad part of inheritance while the type hierarchy is the good one.

One problem with the implementation is that it usually breaks the concept of [encapsulation](<https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)>): When we have a sub-class that uses direct variables of a super-class, we are breaking the encapsulation. The problem here is that we may need to modify the sub-class when we change the super-class. Another issue is that we can make a sub-class from a super-class when they are not directly related. Then, we can use composition over inheritance to avoid these problems, and Barbara Liskov mentioned exactly this point. Therefore, using composition over inheritance is also an excellent and old idea by Barbara Liskov.

The type hierarchy is good because it allows us to write a piece of software that could be replaced without problems. Currently, we have ways to implement type hierarchy that try to mitigate the problems of implementation she mentioned. In Java, we have the concept of [Interfaces](<https://en.wikipedia.org/wiki/Interface_(Java)>); in other languages, such as C++ and Python, we have [abstract classes](<https://en.wikipedia.org/wiki/Class_(computer_programming)\#Abstract_and_concrete>), for example. However, in the end, it is the developer's responsibility to use these concepts correctly. We can use Interfaces and abstract classes wrongly too.

These two concepts, implementation and type hierarchy, are not easy to see as different concepts because we were usually taught they are the same; when it is clear, it is not the truth.
So, this is the importance of the Liskov Substitution Principle and understanding it well. I'm quoting below what she originally said:

```text
"Objects of subtypes should behave like those of supertypes if used via supertype methods"
Barbara Liskov
```

As she said, It is not enough to have only the syntax; we should have semantics too.
A wrong example of applying LSP she mentioned was a paper that made a [stack](<https://en.wikipedia.org/wiki/Stack_(abstract_data_type)>) and a [queue data structure](<https://en.wikipedia.org/wiki/Queue_(abstract_data_type)>) a subtype of each other by naming the methods generically enough. Here, the syntax is correct when we use one type or another in our program; however, semantically, it is wrong. Stack is a Last in First out(LIFO) data structure, while queue is a First in First out(FIFO) data structure. This could create many problems in our software because one's behaviour is different from the others.

A good example of applying LSP is a different implementation of the same concept. For example, a [Binary Search Tree(BST)](<https://en.wikipedia.org/wiki/Binary_search_tree>) and a [Balanced Binary Search Tree(BBST)](<https://en.wikipedia.org/wiki/Self-balancing_binary_search_tree>). Both are syntactically and semantically identical to our program. The difference is only the performance in the implementation. While the BST has a linear time complexity in the worst case, the BBST has a logarithmic time complexity. Then, we can use either a BST or a BBST in our program without any problem. This is a good example of sub-types and the Liskov Substitution Principle.

If you want to learn more about Barbara Liskov, her work on Abstract data types, and her principle, I recommend watching [her speech during her ACM Turing Award acceptance](https://www.youtube.com/watch?v=qAKrMdUycb8), entitled "The Power of Abstraction." You can find the slides she used [here](https://pmg.csail.mit.edu/~liskov/turing-09-5.pdf).
In my opinion, this is a must-watch talk for all software engineers. Barbara Liskov described how she invented the concept of Abstract Data Type and the Liskov Substitution Principle. She also showed the history of software development and object-oriented programming in this talk. It is a brilliant talk!

You can also read the [original paper](https://www.cs.tufts.edu/~nr/cs257/archive/barbara-liskov/data-abstraction-and-hierarchy.pdf) where she created the LSP to go deeper.

If you are starting to learn object-oriented programming, choose composition over inheritance. When you use inheritance, always pay attention to whether you are breaking the concept of encapsulation; if you are, it is time to check your decisions.
