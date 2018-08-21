# Java Streams In Depth

#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures)

### Chapters:

1. [What is a stream?](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Part%201%20-%20What%20is%20a%20java%20stream.md)
2. [Primitive steams](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Part%202%20-%20Primitive%20steams.md)
3. [Optional class](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Part%203%20-%20Optional%20class.md)
4. [Reduction](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Part%204%20-%20Reduction.md)
5. [Collectors](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Part%205%20-%20Collectors.md)
6. [Spliterator](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Part%206%20-%20Spliterator.md)
7. [Parallel Streams](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Part%207%20-%20Parallel%20Streams.md)


### Additional

1. [Introduction](#introduction)
2. [Quiz](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Quiz.md)
3. [Complete terms table](#complete-terms-table)
4. [Spliterator characteristics table](#spliterator-characteristics-table)
5. [Collections characteristics table](#collections-characteristics-table)
6. [Stream methods characteristics table](#stream-methods-characteristics-table)
7. [Stream class methods](https://github.com/stavalfi/lectures/blob/master/Java%20Streams%20-%20Series/Java%20streams%20-%20Stream%20class%20methods.md)
8. [Collectors class methods](#collectors-class-methods)
9. [Spliterator interface methods](#spliterator-interface-methods)
10. [Optional class methods](#optional-class-methods)
11. [Legal](#legal)

----

### Introduction

In this tutorial, we will cover the stream library from the basic operations and learn each relevant classes. The tutorial designates to developers with absolutely no experience with streams to more experienced developers who want to clarify their knowledge. Throughout this series, I will introduce new phrases and explain more deeply each Java stream method.

At the end of the tutorial, the reader will have all the necessary information about how each operation works and how to build a query that uses everything stream can give you.

**Requirements**

It is highly recommended that you will have basic knowledge about templates
Also, a full understanding of lambda expressions is a must.

[Lambda Expressions In Depth By Stav Alfi](https://gist.github.com/stavalfi/e24178288973d104042d4162a02fd135)

---

### Complete terms table

> **Definition 1.1.** _**Pipeline**_ is a couple of methods that will be executed by a specific order.

> **Definition 1.2.** _**Intermidiate operations**_ will be located everywhere in the stream expect at the end. They return a stream object and does not execute any procedure in the pipeline.

> **Definition 1.3.** _**terminal operations**_ will be located only at the end of the stream. They execute the pipeline. They do not return stream object so no other_Intermidiate operations_ or _terminal operations_ can be added after them.

> **Definition 1.4.** Intermediate operations will be called _**short-circuiting operation**_ if that operation produces a new stream with a finite amount of elements from `n` elements where `n` can be an infinite number.

> **Definition 1.5.** Terminal operations will be called _**short-circuiting operation**_ if it can finish his execution after a finite amount of time.

> **Definition 5.1.** **_Reduce-left_** operation determine the first temporary result will be the first element in the collection from the left and the reduce operation will visit each item in the collection that he didn't visit yet from left to right.

> **Definition 5.2.** **_Reduce-right_** operation determine the first temporary result will be the first element in the collection from the right, and the reduce operation will visit each item in the collection that he didn't visit yet from right to left.

> **Definition 8.1.** **_Cuncurrent programming_** is the ability to solve a task using additional synchronization algorithms.

> **Definition 8.2.** **_Parallel programming_** is the ability to solve a task without using additional synchronization algorithms.

###### Aditional definitions

> **Definition 1.** A function or expression is said to have a ___side effect___ if it modifies some state outside its scope or has an observable interaction with its calling functions or the outside world besides returning a value.

> **Definition 2.** A function may be considered a ___pure function___ if both of the following statements about the function hold:
> 1. The function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change while a program execution proceeds or between different executions of the program, nor can it depend on any external input from I/O devices.
> 
>2. Evaluation of the result does not cause any semantically observable side effect or output, such as mutation of mutable objects or output to I/O devices.

> **Definition 3.** ___Lambda Expression___ equals to object of a local class implements an  existing _Functional Interface_. The lambda is the implementation of the single instance method in that interface. _Lambda Expression_  is a function with zero or more parameters and a body that will be (or not) executed at a later stage. We have access to its _free variables_ inside the body.

---

### Spliterator characteristics table

| Characteristic|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :-------------| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CONCURRENT`  | It's possible to modify (delete/insert/replace elements) the source of the spliterator concurrently without any additional synchronization.
| `NONNULL`     | The source of the spliterator can't insert null elements, so each of the elements is guaranteed to be a non-null value. |
| `IMMUTABLE`   | The source of the spliterator prohibit modification (delete/insert/replace elements). |
| `DISTINCT`    | For each 2 different elements `x` and `y` in the source of the spliterator we can tell that `x.equals(y) == false`. |
| `ORDERED`     | If we print the elements in the source of the spliterator multiple times by iterating it, we will print the elements in the same order time after time. That means all the elements are stored in the source of the spliterator in a specific order. We can refer to that situation by specifying that the source of the spliterator has _encounting order_. For example: `Array`, `List`,`Stuck` has _encounting order_ but `HashMap` doesn't. We can't iterate over `HashMap`. |
| `SORTED`      | The source of the spliterator is ordered by a comparator or in the natural order. |
| `SIZED`       | We know the estimated amount of iterations left in the spliterator. |
| `SUBSIZED`  |  If we split the task into two chunks, we would know the estimated amount of iterations each spliterator would have. Note: If `SUBSIZED` is on, then `SIZED` also. The reason is if we know how many iterations will be needed after the split for each spliterator, we must know first how many iterations left without splitting in the first place. The claim doesn't follow for the opposite direction as mentioned above. For example - binary trees. |

---

### Collections characteristics table

The following table list all the characteristics of each collection's spliterator.

|                          |`ORDERED`| `DISTINCT`  | `SORTED`    | `SIZED`     | `NONULL`    | `IMMUTABLE` | `CONCURRENT`| `SUBSIZED`  |
| :----------------------- |:-------:| :----------:| :----------:| :----------:| :----------:| :----------:| :----------:| :----------:|
|`Stream.of`|`I`      |  |  | `I` |  | `I` |  | `I` |
|`Arrays.asList`|`I`      |  |  | `I` |  |  |  | `I` |
|`HashSet`|         | `I` |  | `I` |  |  |  |  |
|`ArrayList`|`I`      |  |  | `I` |  |  |  | `I` |
|`ArrayDeque`|`I`      |  |  | `I` | `I` |  |  | `I` |
|`ArrayBlockingQueue`|`I`      |  |  |  | `I` |  | `I` |  |
|`ArrayDeque`|`I`      |  |  | `I` | `I` |  |  | `I` |
|`ConcurrentLinkedDeque`|`I`      |  |  |  | `I` |  | `I` |  |
|`ConcurrentLinkedQueue`|`I`      |  |  |  | `I` |  | `I` |  |
|`DelayQueue`|         |  |  | `I` |  |  |  | `I` |
|`CopyOnWriteArraySet`|         | `I` |  | `I` |  | `I` |  | `I` |
|`CopyOnWriteArrayList`|`I`      |  |  | `I` |  | `I` |  | `I` |
|`ConcurrentSkipListSet`|`I`      | `I` | `I` |  | `I` |  | `I` |  |
|`LinkedBlockingDeque`|`I`      |  |  |  | `I` |  | `I` |  |
|`LinkedBlockingQueue`|`I`      |  |  |  | `I` |  | `I` |  |
|`LinkedHashSet`|`I`      | `I` |  | `I` |  |  |  | `I` |
|`LinkedList`|`I`      |  |  | `I` |  |  |  | `I` |
|`Stack`|`I`      |  |  | `I` |  |  |  | `I` |
|`RoleUnresolvedList`|`I`      |  |  | `I` |  |  |  | `I` |
|`RoleList`|`I`      |  |  | `I` |  |  |  | `I` |
|`PriorityQueue`|         |  |  | `I` | `I` |  |  | `I` |
|`PriorityBlockingQueue`|         |  |  | `I` | `I` |  |  | `I` |
|`LinkedTransferQueue`|`I`      |  |  |  | `I` |  | `I` |  |
|`SynchronousQueue`|         |  |  | `I` |  |  |  | `I` |
|`TreeSet`|`I`      | `I` | `I` | `I` |  |  |  |  |
|`Vector`|`I`      |  |  | `I` |  |  |  | `I` |

* `I` - Injects. 

---

### Stream methods characteristics table

The following table shows which characteristics and flags each _intermediate operation_ may turn on and off: (`SHORT_CIRCUIT` is relevant only in the context of `StreamOpFlag` flags)

|                  |  DISTINCT |  SORTED |  ORDERED |  SIZED |  SHORT_CIRCUIT |
| ---------------- | ----------| --------| ---------| -------| ---------------|
|  `filter`          |           |         |          |  `C`     |                |
|  `forEach`         |           |         |  `C`       |        |                |
|  `forEachOrdered`  |           |         |          |        |                |
|  `allMatch`        |           |         |  `C`       |        |  `I`             |
|  `distinct`        |  `I`        |         |          |  `C`     |                |
|  `flatMap`         |  `C`        |  `C`      |          |  `C`     |                |
|  `anyMatch`        |           |         |  `C`       |        |  `I`             |
|  `collect`         |           |         |          |        |                |
|  `unOrdered`       |           |         |  `C`       |        |                |
|  `count`           |  `C`        |  `C`      |  `C`       |  `C`     |                |
|  `findAny`         |           |         |  `C`       |        |  `I`             |
|  `findFirst`       |           |         |          |        |  `I`             |
|  `flatMapToXXX`    |  `C`        |  `C`      |          |  `C`     |                |
|  `limit`           |           |         |          |  `C`     |  `I`             |
|  `map`             |  `C`        |  `C`      |          |        |                |
|  `mapToXXX`        |  `C`        |  `C`      |          |        |                |
|  `max`             |           |         |          |        |                |
|  `min`             |           |         |          |        |                |
|  `noneMatch`       |           |         |  `C`       |        |  `I`             |
|  `peek`            |           |         |          |        |                |
|  `reduce`          |           |         |          |        |                |
|  `skip`            |           |         |  `C`       |  `I`     |                |
|  `sorted`          |           |  `I`      |  `I`       |        |                |
|  `toArray`         |           |         |          |        |                |

* `C` - May clears. 
* `I` - May injects. 
---

### Collectors class methods

------

### Spliterator interface methods

----

### Optional class methods

----

### Legal

© Stav Alfi, 2018. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear
credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Streams In Depth" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://github.com/stavalfi/lectures/tree/master/Java%20Streams%20-%20Series.

