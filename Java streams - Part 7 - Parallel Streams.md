# Java streams - Part 7 - Parallel Streams
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://gist.github.com/stavalfi/969539b245fd71f18ecd14f48eed2a5d)

### Topics

1. [Introduction](#introduction)
2. [Example for parallel Java stream](#chapter-1-example-for-parallel-java-stream)
3. [Concurrent vs parallel programming](#chapter-2-concurrent-vs-parallel-programming)
4. [Fork/Join framework introduction](#chapter-3-forkjoin-framework-introduction)
5. [Conclusion](#conclusion)
6. [Legal](#legal)

---

### Introduction

Streams can split the source collection to multiple threads by letting each thread work on the different area in the source collection. We use the intermediate operation `Stream.parallel` to get a stream that will __try to__ be a parallel stream. Each stream operation will examine the stream flags and the spliterator characteristics to see if parallel execution is worth. If intermediate operation `A` agreed to execute in parallel, all the stateless intermediate operations after `A` would do too until we meet a stateful operation `B` that has its own rules. 

---

### Chapter 1: Example for parallel Java stream

```java
int sum = Arrays.asList(1,2,3,4,5,6,7,8) // ORDERED SIZED SUBSIZED
                // every collection have .stream() and
                //.parallelStream() methods.
                .parallelStream()
                // stream know the source collection size and
                // how to split it equeal so filter may run
                // in parallel if the amount of elements is not
                // too small.
                .filter(x -> x > 0) // ORDERED
                // 1. because the ORDER characteristic is still
                // on, limit will return the first 3 elements
                // from the source collection that survived until
                // limit operation. It may be that there
                // are N threads involved in this execution.
                // In case the first thread contain the first 3
                // elements, then even if the N-1 threads
                // got in here, they will not pass the limit
                // because limit is still waiting for the first 3
                // elements from the first thread.
                // As a result, all the threads are stuck here.
                .limit(3) // SIZED ORDERED
                // stream know how many elements are at this moment
                // so filter may run in parallel. if the amount of
                // elements is too small, he will choose not to.
                .filter(x -> x < 5) // ORDERED
                .mapToInt(x-> x) // ORDERED
                .sum(); // ORDERED

System.out.println(sum);

// output: (for every execution - the output won't change)
// 6
```

### Chapter 2: Concurrent vs parallel programming

There are a lot of definitions to _concurrent_ and _parallel_ programming. I will introduce a set of definitions that will serve us best.

> **Definition 8.1.** **_Cuncurrent programming_** is the ability to solve a task using additional synchronization algorithms.

> **Definition 8.2.** **_Parallel programming_** is the ability to solve a task without using additional synchronization algorithms.

Stream library is written using additional synchronization algorithms, but we can use its methods without any. When we write _reducers_ or _collectors_ or make use of the built-in _reducers_/_collectors_ stream gave us, we do not need to use any additional synchronization algorithms to solve our task. 
The only thing we need to do is to understand how to split the big problem into smaller tasks.

The question we need to ask is when do we need to upgrade sequential solution to parallel one? 
* You have lot's of data to calculate so the parallel calculation will make a difference.
* Not every problem that can be solved in _Cuncurrent programming_ can be solved with Parallel programming; Splitting a task to independent smaller tasks is not always possible.
* Even if we succeed to split the task into smaller independent tasks, we still need to store our temporary results. Keep in mind the space complexity needed.

Write each pipeline to support parallelism although it won't be executed in parallel by your or the library choice.

---

### Chapter 3: Fork/Join framework introduction

  Stream library as other Java's standard libraries using the _common thread pool_ to assign tasks to threads. As its name suggests, it's threads are used by everyone. Keep in mind that if you are using parallel stream on the common thread pool to calculate task that may be stuck because they wait for a server response or they call to I/O blocking operation, then you will stack each part of your system that uses the common thread pool also at the same time.
  
With  Fork / Join Framework we can create new thread pools. The advantage is solving the problem I introduced above, but the drawback is we may create too many threads that do not work.

The thread pool assigns tasks to threads which are not busy at the moment. After a thread finishes its task, it will wait for the next task and so on. Each thread pool manages a stuck of tasks (functions) for each of his threads - a thread will take a task from __his__ stuck, and if it's empty, he will take from other thread's stuck.

The _common thread pool_ contains `N` threads. `N` is equal to the number of cores the computer has `- 1`.  The `-1` is for letting the `main` thread to also run without getting into a fight for execution time.

You can force the stream library to use different thread-pool by invoking the stream inside a task that will run in different thread-pool as we will see soon.

In this chapter, we won't examine each class and method this library proved but only the relevant for our need. I recommend any reader to examine for himself the parts that are not covered here after reading this tutorial.

The following `Executors` functions let us create new, different thread pools:

| `public static ExecutorService newSingleThreadExecutor()`                                                                                                                                                         |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Create and return a new thread pool with only one thread that will execute the tasks. The thread pool can't be configured to use more than one thread. If the thread is terminated, a new thread will be created. |

| `public static ExecutorService newFixedThreadPool(int nThreads)`  |
| :---------------------------------------------------------------- |
| Create and return a new thread pool containing `nThreads` threads such that each of them takes tasks from __one__ stuck. If any thread is terminated, a new thread will be created. |

```java
// example:

ExecutorService myThreadPool =
             Executors.newFixedThreadPool(10);
             
myThreadPool.submit(()->{
    // if this stream will run in parallel,
    // it will use myThreadPool threads
    // instead the common thread-pool threads.
    Stream.of(1,2,3,4,5)
            .parallel()
            .peek((Integer x) -> doSomethingeHeavy(x))
            ....
            .forEach(....);
})
```

---

### Legal

© Stav Alfi, 2018. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Java streams - Part 7 - Parallel Streams" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/4744a928f80ca209ce0ffbfb3e27ed96.