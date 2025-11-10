---
title: "Builder Design Pattern and Cyclomatic Complexity Reduction"
date: 2025-11-10T10:00:00-05:00
draft: false
tags: ["Builder design pattern", "Cyclomatic Complexity", "Bit Mask", "Design Pattern", "C++"]
---

In this post, I will explain in detail how I used the [builder design pattern](https://en.wikipedia.org/wiki/Builder_pattern) together with the [bit mask field](https://en.wikipedia.org/wiki/Bit_field) technique to reduce complexity. As a result, I decreased the [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) from 24 to 0 and reduced development time from hours to minutes.

The rest of this post is organized as follows sections:
- [**The problem**](#the-problem) explains the problem I was solving.
- [**Creating the bit mask field**](#creating-the-bit-mask-field) presents a solution to the problem.
- [**Cyclomatic Complexity**](#cyclomatic-complexity) discusses cyclomatic complexity and calculates this metric for the solution presented in the section above.
- [**Builder design pattern**](#builder-design-pattern) explains the builder design pattern and describes the refactoring process that removed complexity.
- [**Conclusion**](#conclusion) summarizes this post.
- [**References**](#references) provides references related to the topics discussed here.

All the examples shown here are in C++.

# The problem

I'm working in an extreme scenario where every millisecond and byte counts. In this type of scenario, we need to design our solutions with extra care regarding the details.
For example, how can we represent 32 Boolean variables for storage and transmission over computer networks?

In C++, the simplest way is to create 32 different Boolean variables. A Boolean variable in C++ occupies 1 byte (8 bits). So, we will need 32 × 1 = 32 bytes to represent all of these variables.
This is not so bad. However, these variables are part of an object, and we could have one billion of these objects. Then, we would consume 32 gigabytes (GB) only to represent these Boolean variables.

We can improve this by using a technique called a bit mask field, where each bit of an integer variable is used to represent a Boolean variable. Considering an integer variable of 4 bytes (32 bits), we need only one integer variable to represent all 32 Boolean variables. With one billion objects, we would now consume 4 GB instead of 32 GB. This is an 87.5% savings.

This is a huge saving, considering that we need to store these variables on disk and transport them via a network between many different systems. The savings here are in both money and time. For the end user, this means they can get results more quickly.

You may be wondering, why don’t we use this technique in all systems to represent our set of Booleans? The reason is pretty simple: we are reducing simplicity to cut costs.
In a non-extreme scenario, using this technique is over-engineering. Instead of saving money, you are creating complexity for your team to handle.

When we decide to use this technique, we need to resolve the problem: how can we create an integer variable to represent Boolean variables in our codebase in an easy-to-maintain way? The short answer is to encapsulate the complexity, which is explained in the rest of this post.

# Creating the bit mask field
For a bit mask field, each bit of an integer variable represents a field. We need a way to set and check if a field is true in this variable. For both operations, we need to use bitwise operators.
For example, let's say we have the int variable `fields`, and we need to set and check the value of field 3 (bit 2, since we start counting from 0).
To set field 3 to true in `fields`, we can do:

```c++
fields |= (1 << 2);
```

The `<<` is the [left shift operator](https://en.cppreference.com/w/cpp/utility/bitset/operator_ltltgtgt.html) that shifts the number by 2 positions to the left. So, this will create the number 4 (100 in binary).

To check if field 3 is true in `fields`, we can do:

```c++
int mask = (1 << 2);
(fields & mask) == mask;
```

The `&` is the [bitwise AND operator](https://en.cppreference.com/w/cpp/language/operator_arithmetic.html#Bitwise_logic_operators) that returns 1 when both corresponding bits in `fields` and `mask` are 1, and 0 otherwise. In other words, field 3 is true only when the bitwise AND operator returns `mask` as the result.

These are the operations in isolation; our solution needs to create the variable `fields` and set each field based on some conditions, as shown below:

```c++
bool condition1(){
    //call complexity methods here
    return false; // just as an example
};

bool condition2(){
    //call complexity methods here
    return true; // just as an example
};

int createFields(){
    int fields = 0;

    if(condition1()){
        fields |= (1 << 1);
    }

    if(condition2()){
        fields |= (1 << 2);
    }

    return fields;
}
```

# Cyclomatic complexity
The cyclomatic complexity metric was developed by [Thomas J. McCabe](https://ieeexplore.ieee.org/author/37860859000) in the paper ["A Complexity Measure"](https://ieeexplore.ieee.org/document/1702388). Basically, it is a metric used to calculate how many paths your code has. To calculate this metric in your codebase, you can use [lizard](https://github.com/terryyin/lizard), which is an extensible Cyclomatic Complexity Analyzer for many programming languages.

The function `createFields`, presented in the last section, has a cyclomatic complexity of 3: 1 for the function itself, +1 for the first condition, and +1 for the second condition.

One question naturally emerges: What is a good cyclomatic complexity? The answer is not simple. There is no magic number, and this metric is just an indicator of potential issues in your codebase. Generally, smaller numbers are better.

In my opinion, the most important aspect is not just the number or the conditions in the code. The biggest problems are complexity and dependencies, which this metric can help you discover. Of course, you do not need this metric to identify complexity; you can do it yourself by reading the code. However, in a large codebase, this metric helps you decide which places to review first.

Let's go back and analyze the function `createFields`. The cyclomatic complexity metric counts +1 for each condition, which is only an indicator. For each of these conditions, the function `createFields` is coupled with the functions `condition1` and `condition2`, and the path our program will take depends on the result of these functions.

In other words, the client of this code—the caller—is responsible for knowing all these details and handling them. The most basic client is the tests. In this case, to test if `fields1` is set correctly, we need to either set up everything that the functions `condition1` and `condition2` need or mock these functions. Even though I want to check just `field1`, I need to set up all the dependencies of `createFields`, not just the dependencies for `field1`.

We can reduce the complexity of the function `createFields` by using the Builder design pattern, as explained in the next section.

# Builder design pattern

The [Builder design pattern](https://en.wikipedia.org/wiki/Builder_pattern) is a creational patterns that helps encapsulate the complexity of creating an object. It is described in the famous book [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns) by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, also known as the Gang of Four (GoF). This book is a classic and a must-read for experienced software engineers. I agree with Martin Fowler - it's a great book, but not an easy read. It's best approached when you're comfortable with object-oriented design. As [Martin Fowler](https://martinfowler.com/aboutMe.html) noted in 2004, ["In my view, the Gang of Four is the best book ever written on object-oriented design"](https://martinfowler.com/bliki/GangOfFour.html).

The function `createFields` is responsible for creating the variable `fields`, which maps many Boolean variables into a single integer variable to save memory. This is a very good candidate to be simplified using the Builder design pattern.
So, we can rewrite `createFields` using the Builder design pattern to encapsulate and reduce complexity. Below, we present the new code.
You can notice some differences here. Now, we have classes instead of just a function. The `BitMaskField` class encapsulates the integer variable that represents the Boolean variables and provides a clear API for clients. The `BitMaskFieldBuilder` class is used to construct `BitMaskField` objects. The `withField1` and `withField2` methods handle the creation of each field, respectively.

Besides the improved organization, one major benefit is that the code is now decoupled. To use `field1`, callers do not need to know anything about the dependencies of other fields. All the complexity of creating an integer variable to map many Boolean variables is encapsulated in the `BitMaskFieldBuilder` class, and each field is completely decoupled from the others. Adding a new field is an easy task now. We just need to add a new method in the builder class.

```c++
class BitMaskField {
    private:
        int data = 0;

    public:
        void setField(int position){
            this->data |= (1 << position);
        }

        bool isFieldUsed(int position){
            int mask = (1 << position);
            return (this->data & mask) == mask;
        }
};

class BitMaskFieldBuilder {
    private:
        BitMaskField bitMaskField = BitMaskField();

    public:
        BitMaskFieldBuilder withField1(){
            if(condition1()){
                bitMaskField.setField(1);
            }
            return *this;
        }

        BitMaskFieldBuilder withField2(){
            if(condition2()){
                bitMaskField.setField(2);
            }
            return *this;
        }

        BitMaskField build(){
            return bitMaskField;
        }
};

BitMaskField createFields(){
    BitMaskField fields = BitMaskFieldBuilder().withField1()
        .withField2()
        .build();
    return fields;
}
```

# Conclusion
In this post, I demonstrated how combining the Bit Mask technique with the Builder design pattern can reduce cyclomatic complexity and simplify the codebase. The benefits are multiples: developers can easily test new implementations without needing to know the details of other fields, the company saves on storage and network transport costs, and users experience faster system responses.

The bit mask field is a valuable technique for minimizing memory and storage usage. However, it can introduce complexity if not implemented thoughtfully. Here, I showed how leveraging abstraction and the Builder design pattern can encapsulate this technique, making it more manageable and reducing overall complexity.

# References
- [A Complexity Measure](https://ieeexplore.ieee.org/document/1702388)
- [Bit Mask Field Technique](https://en.wikipedia.org/wiki/Bit_field)
- [Bitwise AND operator](https://en.cppreference.com/w/cpp/language/operator_arithmetic.html#Bitwise_logic_operators)
- [Builder Design Pattern](https://en.wikipedia.org/wiki/Builder_pattern)
- [Cyclomatic Complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns)
- [Left shift Operator](https://en.cppreference.com/w/cpp/utility/bitset/operator_ltltgtgt.html)
- [Martin Fowler notes about the Gang of Four](https://martinfowler.com/bliki/GangOfFour.html)
- [Martin Fowler](https://martinfowler.com/aboutMe.html)
- [Thomas J. McCabe](https://ieeexplore.ieee.org/author/37860859000)
- [lizard: extensible Cyclomatic Complexity Analyzer for many programming languages](https://github.com/terryyin/lizard)
