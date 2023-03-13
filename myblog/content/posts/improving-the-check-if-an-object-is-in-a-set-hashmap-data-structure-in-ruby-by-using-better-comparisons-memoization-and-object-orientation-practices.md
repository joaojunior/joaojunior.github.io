---
title: "Improving the check if an object is in a Set/HashMap data structure in Ruby by using better comparisons, memoization, and object orientation practices"
date: 2023-03-13T14:04:03-04:00
draft: false
tags: ["Ruby", "Data structure", "Set", "HashMap", "Improving comparisons", "Improving performance", "FlameGraph"]
---

I was verifying the performance of an endpoint in an API using [framegraph](https://rbspy.github.io/profiling-guide/using-flamegraphs.html) when I noticed that checking if an object is in a set or a hash data structure was consuming a lot of time.
This post explains the problem and the solution to it.

First of all, checking if an object is in a [Set](https://en.wikipedia.org/wiki/Set_(abstract_data_type)) or [HashMap](https://en.wikipedia.org/wiki/Hash_table) is `O(1)` when we don't consider collisions. Even in the presence of collisions, it should be much less than `O(n)`, where `n` is the number of items in the set. Otherwise, we are operating as a linked list.

In my case, checking if an object is in a Set of 1000 objects was consuming 1.62 seconds or 37% of my request time. Only for us to compare, checking if an integer is in a Set with 1000 integers consumes less than 1 millisecond.

So, what is the problem? To dive into it below is the class I was working with. I removed everything from the business problem and changed the class name only to allow us to concentrate on the problem and the solution here.

```ruby
class CustomID
  attr_reader :id

  def initialize(id)
    @id = id
  end

  def ==(other)
    other.class == self.class && other.id == id
  end

  def hash
    self.class.hash | id.hash
  end
end
```

There is nothing wrong with this code. However, if we use it frequently, it could be a bottleneck.

The method `==` makes a string comparison which is `O(n)`, where `n` is the size of the string, and in case of success, it compares two integers. I improved it to make a faster integer comparison and use only one machine instruction. Integer comparison is `O(b)`, where `b` is the number of bits used to represent the integer.

The method `hash` is very expensive. It calculates the hash of a string and the hash of an integer and does an [`or`](https://en.wikipedia.org/wiki/Logical_disjunction) operation.

Both methods cited above are where our performance problem starts. For a Set of 1000 elements, we need to calculate the object's hash and use the method `==` to check if an object is inside it. Again, this is fine if we check this a few times. However, doing it a few million times daily could be a significant bottleneck.

The solution is to remove the string comparison in the method `==` and memoize the expansive hash operation. The new code is below:

```ruby
class ImprovedCustomID
  attr_reader :id

  def initialize(id)
    @id = id
  end

  def ==(other)
    other.hash == hash
  end

  def hash
    @hash ||= self.class.hash | id.hash
  end

  def freeze
    hash
    super
  end
end
```

The new code overrides the method `freeze` to call `hash` and only after it, call `super`. The override is necessary because the `hash` memoized version adds a new instance variable `@hash` when it is called the first time.

To compare both versions, we can use the code below, which uses the library [benchmark-ips](https://github.com/evanphx/benchmark-ips):

```ruby
require "benchmark/ips"
custom_id = CustomID.new(61000)
improved_custom_id =ImprovedCustomID.new(61000)
custom_ids = (1..1000).map {|i| CustomID.new(i*1000) }
improved_custom_ids = (1..1000).map {|i| ImprovedCustomID.new(i*1000) }
Benchmark.ips do |x|
  x.report("include? before improvements") { custom_ids.include?(custom_id) }
  x.report("include? after improvements") { improved_custom_ids.include?(improved_custom_id) }
  x.compare!
end
```

With this benchmark, we got the results:
```
Warming up --------------------------------------
include? before improvements
                        16.564k i/100ms
include? after improvements
                        23.623k i/100ms
Calculating -------------------------------------
include? before improvements
                        165.641k (± 0.3%) i/s -    828.200k in   5.000032s
include? after improvements
                        235.784k (± 0.3%) i/s -      1.181M in   5.009491s

Comparison:
include? after improvements:   235784.5 i/s
include? before improvements:   165640.9 i/s - 1.42x  slower
```

Therefore, in this simple case, the new code is 42% faster. The new code was 2x more quickly in my production case, from 1.62 seconds to 866.16 milliseconds.
