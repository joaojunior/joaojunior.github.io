---
title: "Is CPython Faster Without the Global Interpreter Lock(GIL)?"
date: 2024-07-17T12:40:06-04:00
draft: false
tags: ["Python", "Threads", "GIL", "NoGIL"]
---

The [Global interpreter lock(GIL)](https://docs.python.org/3/glossary.html#term-global-interpreter-lock) is a mechanism in [CPython](https://github.com/python/cpython) implementation to ensure only one thread is running at a time.
This means that even if we use [threads](https://docs.python.org/3/library/threading.html) to try to speed up our programs' runtime, this will not work because, in the end, the interpreter will block that two or more threads will be executed in parallel.

As the name suggests, the GIL is a locker that a thread gets when it starts and releases at some moment. There is only one locker, so two threads cannot get it simultaneously. As a result, our program with threads will be slower than the same program without threads because of the time to create threads and not take advantage of them.
However, this is true only for [CPU bound problems](https://en.wikipedia.org/wiki/CPU-bound), which requires a large amount of CPU time to process, like [inverse matrix](https://en.wikipedia.org/wiki/Invertible_matrix). For I/O bound problems, CPython releases the locker as soon as an I/O call happens. We still have only one thread running at once on the CPU, but now, a thread does not block other threads while waiting for the I/O result.

For many years, a great effort has been made to implement CPython without the GIL. The good news is that we are almost there. In last March, [this](https://github.com/python/cpython/pull/116338) pull request was merged into CPython and it allows to disable the GIL. More details about it can be found on the [PEP 703 - Making the Global Interpreter Lock Optional in CPython](https://peps.python.org/pep-0703/).
So, here, I will test the performance of CPython without threads using the recursive Fibonacci algorithm. There are better ways to calculate Fibonacci numbers than the recursive algorithm. However, the recursive approach is a CPU-bound implementation in which we can easily reduce the process time by using threads.

First, I will show how to download and compile the CPython with the option to disable the GIL. After that, I will run the Fibonacci implementation with and without threads, and the GIL enabled. With this test, I'm creating a base benchmarking to see the effect of the modifications to disable the GIL. The threads' runtime will likely be slow since the GIL is enabled.

After this, I will run the exact implementation as above but with the GIL disabled. Now, I'm trying to see the GIL's effect. My hypothesis here is that the runtime without GIL will be much faster.

# Download and Install
To test CPython without GIL, we need to compile it with the option to disable GIL. As of this post, the latest version is [v3.13.0b3](https://docs.python.org/3.13/whatsnew/3.13.html). So, we can download CPython on this version with the command below. We need to have git installed.

```bash
git clone --depth 1 --branch v3.13.0b3 https://github.com/python/cpython
```

Now, we need to compile the CPython with the option to turn the GIL on and off. We can do that from the folder we downloaded CPython above:

```bash
./configure --disable-gil
make
make test
```

Last, we need to install this new CPython.
```bash
sudo make install
```

By default, the CPython has the GIL enabled. But we can also call the interpreter and enable the GIL with the command:
```
PYTHON_GIL=1 python3.13t
```

To run the interpreter with the GIL disabled, we can use the command:
```
PYTHON_GIL=0 python3.13t
```

# Calculating Fibonacci with a recursive algorithm and GIL enabled

The [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_sequence) is defined by the recurrence relation:
\[
\begin{aligned}
F_n = F_{n - 1} + F_{n - 2}, \forall n >= 2 \\
F_1 = 1,
F_0 = 0
\end{aligned}
\]

Where `n` is the Nth Fibonacci number to be calculated.

The code below implements this relation in Python using recursive calls. There are better ways to calculate Fibonacci numbers than this one, but it is perfect for our performance test.

```Python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

We also need an algorithm with threads to test the performance of CPython with GIL enabled. The implementation is below, using the exact Fibonacci implementation above but now with threads. This algorithm does not create threads to calculate the Fibonacci number; instead, it receives the `times` parameter representing the number of times we will calculate the Nth Fibonacci number. It also receives the `threads` parameter: the number of threads it will create.

```Python
from concurrent.futures import ThreadPoolExecutor

def fib_with_threads(n, times, threads):
    with ThreadPoolExecutor(max_workers=threads) as executor:
        for _ in range(times):
            executor.submit(fib, n)
```

So, I  will calculate the same Fibonacci number multiple times but using threads. To make a fair comparison, I can run both algorithms the same number of times and compare the runtime.

I will calculate the Fibonacci number for our performance test when `n=42`. I will also run both implementations the same number of times, 8, in this performance test.

The table below shows the result. Each line of this table represents an algorithm. The `test_fib` is the calculation without any threads. While `test_fib_with_threads[t]` represents the calculation using threads, where `t` is the number of threads. As I expected, there is no difference in performance between these algorithms with the GIL enabled.

--------------------------------------------------------------------------------------
| Name (time in s)            | Min              | Max              | Mean             | StdDev           | Median           |
|-----------------------------|------------------|------------------|------------------|------------------|------------------|
| test_fib_with_threads[1]    | 599.8635 (1.0)   | 603.9185 (1.0)   | 602.2655 (1.0)   | 1.6091 (4.21)    | 602.5483 (1.0)   |
| test_fib                    | 609.0794 (1.02)  | 610.7804 (1.01)  | 609.9484 (1.01)  | 0.6173 (1.61)    | 610.0402 (1.01)  |
| test_fib_with_threads[2]    | 619.3041 (1.03)  | 621.8827 (1.03)  | 620.3884 (1.03)  | 1.1031 (2.88)    | 620.1811 (1.03)  |
| test_fib_with_threads[4]    | 637.1945 (1.06)  | 638.5143 (1.06)  | 637.9775 (1.06)  | 0.5297 (1.38)    | 637.9275 (1.06)  |
| test_fib_with_threads[8]    | 637.8160 (1.06)  | 638.7403 (1.06)  | 638.0755 (1.06)  | 0.3825 (1.0)     | 637.9066 (1.06)  |
--------------------------------------------------------------------------------------


# Calculating Fibonacci with a recursive algorithm and GIL disabled

I ran the same algorithms from the previous section, but now with the GIL disabled. This performance test will show us the performance of running threads in CPython with the GIL disabled.

The table below shows the result; the rows represent the same as in the section above. As expected, the calculation using more threads is quicker than using fewer or no threads. The calculation using 8 threads consumed less than 25% of the time than the calculation with no threads. A huge difference.

--------------------------------------------------------------------------------------
| Name (time in s)            | Min              | Max              | Mean             | StdDev           | Median           |
|-----------------------------|------------------|------------------|------------------|------------------|------------------|
| test_fib_with_threads[8]    | 147.8931 (1.0)   | 150.5152 (1.0)   | 149.1743 (1.0)   | 1.0378 (2.80)    | 149.0570 (1.0)   |
| test_fib_with_threads[4]    | 182.7308 (1.24)  | 186.0723 (1.24)  | 184.9141 (1.24)  | 1.2782 (3.45)    | 185.3121 (1.24)  |
| test_fib_with_threads[2]    | 317.9565 (2.15)  | 318.7826 (2.12)  | 318.3247 (2.13)  | 0.3705 (1.0)     | 318.2227 (2.13)  |
| test_fib_with_threads[1]    | 594.9699 (4.02)  | 598.6776 (3.98)  | 597.2791 (4.00)  | 1.4761 (3.98)    | 597.5705 (4.01)  |
| test_fib                    | 600.2422 (4.06)  | 604.5510 (4.02)  | 602.0399 (4.04)  | 1.6212 (4.38)    | 601.9162 (4.04)  |
--------------------------------------------------------------------------------------

# Conclusion

I have worked with Python since 2008, but whenever I need to resolve a CPU-bound problem, I need to use another programming language or write C extensions to be used in Python, which essentially means using another programming language.
This is no longer the case, as the option to disable the GIL is now available. Threads in CPython perform great when we disable the GIL.

This development couldn't have come at a better time, especially in the context of the current AI boom. With AI demanding numerous parallel processes to accelerate algorithms, the option to disable the GIL in Python is a huge improvement.

# Reproducing this experiment

To reproduce this experiment, you need to download and compile CPython, as explained at the beginning of this article. Also, you need to install the [pytest](https://docs.pytest.org/en/8.2.x/) and [pytest-benchmark](https://pypi.org/project/pytest-benchmark/). You can do this with the command below in the command line:
```
python3.13t -m pip install pytest pytest-benchmark
```

The entire code used in this post is below; you need to save it into a file, like `fib.py`:

```Python
from concurrent.futures import ThreadPoolExecutor

import pytest

N = 42
FIB_42 = 267914296


def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)


def fib_times(n, times):
    for _ in range(times):
        assert FIB_42 == fib(n)


def fib_with_threads(n, times, threads):
    with ThreadPoolExecutor(max_workers=threads) as executor:
        for _ in range(times):
            executor.submit(fib_times, n, 1)


def parameters():
    times = 8
    return times


def test_fib(benchmark):
    times = parameters()

    benchmark(fib_times, N, times)


@pytest.mark.parametrize("number_of_threads", [1, 2, 4, 8])
def test_fib_with_threads(number_of_threads, benchmark):
    times = parameters()
    benchmark(fib_with_threads, N, times, number_of_threads)
```

So, you can run the performance test with the GIL enabled and disabled, respectively, with the commands: `PYTHON_GIL=1 python3.13t -m pytest fib.py` and `PYTHON_GIL=0 python3.13t -m pytest fib.py`
