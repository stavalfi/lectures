# Java streams in depth

#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures)

### Series:

1. [What is a stream?](https://gist.github.com/stavalfi/6c385f8360091cc33b6701bd4ee5c242)
2. [Primitive steams](https://gist.github.com/stavalfi/98c170f8f13c302c76f8bf92cda62557)
3. [Optional class](https://gist.github.com/stavalfi/0e80e3df73e7c2ab3c3310ddef573bba)
4. [Reduction](https://gist.github.com/stavalfi/0c63570c2db4a31d2177ba921cdd9e11)
5. [Collectors](https://gist.github.com/stavalfi/3ba04d7d114132d6f57413052bd8daa2)
6. [Spliterator](https://gist.github.com/stavalfi/5f357213c3275fe923727a6221eafecd)
7. [Parallel Streams](https://gist.github.com/stavalfi/4744a928f80ca209ce0ffbfb3e27ed96)


### Additional

1. [Introduction](#introduction)
2. [Quiz](#quiz)
3. [Complete terms table](#complete-terms-table)
4. [Spliterator characteristics table](#spliterator-characteristics-table)
5. [Collections characteristics table](#collections-characteristics-table)
6. [Stream methods characteristics table](#stream-methods-characteristics-table)
7. [Stream class methods](#stream-class-methods)
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
  ### Quiz
  The Answers are at the end of the quiz.

###### Question 1

Name one difference between the following pipes. Do they print the same output? If yes, which version would you prefer - name one reason.

```java
Stream.of(1, 2, 3)
        .map((Integer number) -> number * 10)
        .findFirst()
        .ifPresent((Integer number) -> System.out.println(number));

Stream.of(1, 10, 100, 1000)
        .findFirst()
        .map((Integer number) -> number * 10)
        .ifPresent((Integer number) -> System.out.println(number));
```

[Answer 1](#1-question-with-answer)

###### Question 2

Name one difference between the following pipes. Do they print the same output? If yes, which version would you prefer - name one reason.

```java
Stream.of(1, 2, 3)
        .filter((Integer number) -> number > 1)
        .findFirst()
        .ifPresent((Integer number) -> System.out.println(number));

Stream.of(1, 10, 100, 1000)
        .findFirst()
        .filter((Integer number) -> number > 1)
        .ifPresent((Integer number) -> System.out.println(number));
```

[Answer 2](#2-question-with-answer)

###### Question 3

Does the following code compile? If yes, what does it print? If not, what is the reason and Java implementors didn't allow it?
```java
Stream.of(1, 2, 3)
        .filter((Integer number) -> number > 1)
        .findFirst()
        .sorted()
        .ifPresent((Integer number) -> System.out.println(number));
```

[Answer 3](#3-question-with-answer)

###### Question 4

What methods must the `Person` class implement to make the following code works? (I mean, what `sorted` operation invokes in `Person` class?)

```java
Stream.of(new Person(1), new Person(2))
        .sorted()
        .findFirst()
        .ifPresent((Person person) -> System.out.println(person));
```

[Answer 4](#4-question-with-answer)

###### Question 5

What does the following code prints? Could we made it more readable?
```java
Predicate<Integer> p1 = (Integer number) -> number % 2 == 0;
Predicate<Integer> p2 = (Integer number) -> number % 3 == 0;

Stream.of(1, 2, 3)
        .peek((Integer number) -> System.out.println(number))
        .filter(p1.and(p2)) // <<<----- and
        .forEach((Integer number) -> System.out.println(number));
```

[Answer 5](#5-question-with-answer)

###### Question 6

What does the following code prints? Could we made it more readable?
```java
Predicate<Integer> p1 = (Integer number) -> number % 2 == 0;
Predicate<Integer> p2 = (Integer number) -> number % 3 == 0;

Stream.of(1, 2, 3)
        .peek((Integer number) -> System.out.println(number))
        .filter(p1.or(p2)) // <<<----- or
        .forEach((Integer number) -> System.out.println(number));
```

[Answer 6](#6-question-with-answer)

###### Question 7

What does the following code prints?
```java
Stream<Integer> stream1 = Stream.of(1, 2)
        .peek((Integer number) -> System.out.println(number));

Stream<Integer> stream2 = Stream.of(1, 2)
        .peek((Integer number) -> System.out.println(number));

stream1.flatMap(number -> stream2.filter(x -> x != number))
        .forEach((Integer number) -> System.out.println(number));
```

[Answer 7](#7-question-with-answer)

###### Question 8

How can you improve the readability of the following pipeline:
  
```java
List<Integer> l1 = Arrays.asList(1,2,3,4,5);

int sum1 = Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream()
                .map((Integer x) -> x * 10)
                .peek(System.out::println)
        )
        .mapToInt(x->x)
        .sum();
  ```
  
[Answer 8](#8-question-with-answer)

###### Question 9

Does both the pipelines print the same elements in the same order? Is it the _encounting order_?
 
```java
List<Integer> l1 = Arrays.asList(1,1,2,2);

Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream()
                .sorted()
                .distinct()
        )
        .forEach(System.out::println)

Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream())
        .sorted()
        .distinct()
        .forEach(System.out::println)
  ```
  
[Answer 9](#9-question-with-answer)

###### Question 10

Does this code compile?

```java
List<Integer> l1 = Arrays.asList(1,1,2,2);

Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream())
        .mapToInt(Function.identity())
        .forEach(System.out::println);

// the source code of Function.identity():
//
// static <T> Function<T, T> identity() {
//     return t -> t;
// }
```

[Answer 10](#10-question-with-answer)

###### Question 11

Can we improve the readability and the safety of the following code?

```java
List<Integer> list1=new ArrayList<>();

// list1 is used only after the following code is done.
// your co-worker says that this pipeline has no
// reason/can't run for some reason in parallel.
Stream.of(1,2,3,4,5)
        .filter(x->x>3)
        .peek(x-> callingHeavyFunction(x))
        .sorted()
        .forEach(x-> list1.add(x));
```

[Answer 11](#11-question-with-answer)

###### Question 12

Write a `Map` collector in the following way (implement the `Collector` interface):
* The keys are the elements from the calling stream.
* The values are `List<T>`. `T` is the type of the elements from the calling stream.
* Each list accept only five items at most.

[Answer 12](#12-question-with-answer)

#### Answers: (With the questions)

###### 1. Question with Answer

###### Question

Name one difference between the following pipes. Do they print the same output? If yes, which version would you prefer - name one reason.

```java
Stream.of(1, 2, 3)
        .map((Integer number) -> number * 10)
        .findFirst()
        .ifPresent((Integer number) -> System.out.println(number));

Stream.of(1, 10, 100, 1000)
        .findFirst()
        .map((Integer number) -> number * 10)
        .ifPresent((Integer number) -> System.out.println(number));
```

######  Answer

The difference is while the first pipeline run `map` on `Stream<Integer>`, the second pipeline run `map` on `Optional<Integer>`. 

 `map` only creates a new stream with new content with each cell. so if `map` operation is running on a `Stream` with 0+ elements or an `Optional` with 0-1 elements, we will get the same result.

I would prefer the first one because for after the `findFirst` operation, we are not operating on stream anymore, so the operation `findFirst` is a delimiter between `Stream` pipeline and `optional` pipeline. My recommendation is to highlight this delimiter by locating it as low as we can because the amount of operation in `Optional` pipeline is small.

[Question 2](#question-2)

###### 2. Question with Answer

###### Question

Name one difference between the following pipes. Do they print the same output? If yes, which version would you prefer - name one reason.

```java
Stream.of(1, 2, 3)
        .filter((Integer number) -> number > 1)
        .findFirst()
        .ifPresent((Integer number) -> System.out.println(number));

Stream.of(1, 10, 100, 1000)
        .findFirst()
        .filter((Integer number) -> number > 1)
        .ifPresent((Integer number) -> System.out.println(number));
```

######  Answer

The difference is while the first pipeline run `filter` on `Stream<Integer>`, the second pipeline run `filter` on `Optional<Integer>`. 

Both the pipelines calculate different results. The first will print `2` and the second will print nothing.

[Question 3](#question-3)

###### 3. Question with Answer

###### Question

Does the following code compile? If yes, what does it print? If not, what is the reason and Java implementors didn't allow it?
```java
Stream.of(1, 2, 3)
        .filter((Integer number) -> number > 1)
        .findFirst()
        .sorted()
        .ifPresent((Integer number) -> System.out.println(number));
```

######  Answer

The code doesn't compile because `Optional` class doesn't have `sorted` method. The reason for that is because `Optional` instance represents an amount of 0-1 elements, so there is no point in sorting them.

[Question 4](#question-4)

###### 4. Question with Answer

###### Question

What methods must the `Person` class implement to make the following code works? (I mean, what `sorted` operation invokes in `Person` class?)

```java
Stream.of(new Person(1), new Person(2))
        .sorted()
        .findFirst()
        .ifPresent((Person person) -> System.out.println(person));
```

######  Answer

`Person` class need to implement `Comparable` functional interface.

```java
public class Person implements Comparable<Person> {
    private int id;

    public Person(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    @Override
    public int compareTo(Person person) {
        Objects.requireNonNull(person);

        if (getId() == person.getId())
            return 0;
        else if (getId() > person.getId())
            return 1;
        return -1;
    }
}
```

[Question 5](#question-5)

###### 5. Question with Answer

###### Question

What does the following code prints? Could we made it more readable?
```java
Predicate<Integer> p1 = (Integer number) -> number % 2 == 0;
Predicate<Integer> p2 = (Integer number) -> number % 3 == 0;

Stream.of(1, 2, 3)
        .peek((Integer number) -> System.out.println(number))
        .filter(p1.and(p2)) // <<<----- and
        .forEach((Integer number) -> System.out.println(number));
```

######  Answer

The above code does not print anything. Only elements which divide by 2 and three will pass the `filter` operation. That means that only items which divided by six will. There any no such elements here, so nothing is printed.

We could write it in different way (Personally, I would prefer the way it was in the question):

```java
Predicate<Integer> p1 = (Integer number) -> number % 2 == 0;
Predicate<Integer> p2 = (Integer number) -> number % 3 == 0;

Stream.of(1, 2, 3)
        .peek((Integer number) -> System.out.println(number))
        .filter(p1)
        .filter(p2)
        .forEach((Integer number) -> System.out.println(number));
```

In case we are not using `p1` and `p2` anywhere else in our code, we could write the following:

```java
Stream.of(1, 2, 3)
        .peek((Integer number) -> System.out.println(number))
        .filter((Integer number) -> number % 6 == 0)
        .forEach((Integer number) -> System.out.println(number));
```

[Question 6](#question-6)

###### 6. Question with Answer

###### Question

What does the following code prints? Could we made it more readable?
```java
Predicate<Integer> p1 = (Integer number) -> number % 2 == 0;
Predicate<Integer> p2 = (Integer number) -> number % 3 == 0;

Stream.of(1, 2, 3)
        .peek((Integer number) -> System.out.println(number))
        .filter(p1.or(p2)) // <<<----- or
        .forEach((Integer number) -> System.out.println(number));
```

######  Answer

It will print: 
```java
2
3
```
The reason is similar to the last answer. But here we could not break it down to 2 `filter` operations because it will be equal to `and`.

In case we are not using `p1` and `p2` anywhere else in our code, we could write the following:

```java
Stream.of(1, 2, 3)
        .peek((Integer number) -> System.out.println(number))
        .filter((Integer number) -> number % 2 == 0 || number % 3 == 0)
        .forEach((Integer number) -> System.out.println(number));
```

[Question 7](#question-7)

###### 7. Question with Answer

###### Question

What does the following code prints?
```java
Stream<Integer> stream1 = Stream.of(1, 2)
        .peek((Integer number) -> System.out.println(number));

Stream<Integer> stream2 = Stream.of(1, 2)
        .peek((Integer number) -> System.out.println(number));

stream1.flatMap(number -> stream2.filter(x -> x != number))
        .forEach((Integer number) -> System.out.println(number));
```

######  Answer

It would print the following __if and only if__ Java would allow using the same stream (`stream2`) more than once:
```java
1
1
2
2
2
1
1
2
```

But because Java doesn't allow it, then it would print: 
```java
"java.lang.IllegalStateException: stream has already been operated upon or closed.."
```

[Question 8](#question-8)

###### 8. Question with Answer

###### Question

How can you improve the readability of the following pipeline:
  
```java
List<Integer> l1 = Arrays.asList(1,2,3,4,5);

int sum1 = Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream()
                .map((Integer x) -> x * 10)
                .peek(System.out::println)
        )
        .mapToInt(x->x)
        .sum();
  ```

######  Answer

Each node of `l1` is a list. Inside the `flatMap` method we return every inner list of l1 with a minor change on each element. All in all, all the items in all the inner lists will be changed in the same way so we can do that change outside of the flat map.
  
```java
List<Integer> l1 = Arrays.asList(1,2,3,4,5);
  
int sum1 = Arrays.asList(l1,l1,l1)
      .stream()
      .flatMap(list1 -> list1.stream())
      .mapToInt((Integer x) -> x * 10)
      .peek(System.out::println)
      .sum();
```

[Question 9](#question-9)

###### 9. Question with Answer

###### Question

Does both the pipelines print the same elements in the same order? Is it the _encounting order_?
 
```java
List<Integer> l1 = Arrays.asList(1,1,2,2);

Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream()
                .sorted()
                .distinct()
        )
        .forEach(System.out::println)

Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream())
        .sorted()
        .distinct()
        .forEach(System.out::println)
  ```

######  Answer

No, inside the first pipeline, the `flatMap` flat the following lists: `1,2`,`1,2`,`1,2`to: `1,2,1,2,1,2,1,2` and then print it. The second pipeline sort the elements from all the origin inner __lists__ : `1,1,2,2,1,1,2,2,1,1,2,2` to `1,1,1,1,1,1,2,2,2,2,2,2`and then call `distinct()`: `1,2` and then print it.

[Question 10](#question-10)

###### 10. Question with Answer

###### Question

```java
List<Integer> l1 = Arrays.asList(1,1,2,2);

Arrays.asList(l1,l1,l1)
        .stream()
        .flatMap(list1 -> list1.stream())
        .mapToInt(Function.identity())
        .forEach(System.out::println);

// the source code of Function.identity():
//
// static <T> Function<T, T> identity() {
//     return t -> t;
// }
```

######  Answer

The code doesn't compile. `mapToInt` receive a lambda which gets an `Integer` and returns `int`. Since `Integer` isn't `int` then we can't use `Function.identity()`; The parameter type does not equal the return type.

[Question 11](#question-11)

###### 11. Question with Answer

###### Question

Can we improve the readability and the safety of the following code?

```java
List<Integer> list1=new ArrayList<>();

// list1 is used only after the following code is done.
// your co-worker says that this pipeline has no
// reason/can't run for some reason in parallel.
Stream.of(1,2,3,4,5)
        .filter(x->x>3)
        .peek(x-> callingHeavyFunction(x))
        .sorted()
        .forEach(x-> list1.add(x));
```

######  Answer

Yes. `list1` is used only after filling him, so there is no reason to declare `list` before the stream. We should convert the stream to a list using `Collectors.toList()`. Also, this code does not support parallelism. Even if `callingHeavyFunction` function does not support parallelism now, it can happen in the future. We should write uniform pipelines.

###### 12. Question with Answer

###### Question

Write a `Map` collector in the following way (implement the `Collector` interface):
* The keys are the elements from the calling stream.
* The values are `List<T>`. `T` is the type of the elements from the calling stream.
* Each list accept only five items at most.

######  Answer

  --missing--
  
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

> **Definition 1.** ___side-effect___  is a concept where a function mutates varaibles which are not defined in the function scope.

> **Definition 2.** A ___pure function___ is a function's return value that is **only** dependent on the input parameters. Also, Evaluating the result does not mutate the current state of the app or causing any **side-effects**.

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

### Stream class methods

I will refer to `Spliterator` characteristics as flags inside `StreamOpFlag` enum. That is because only the flags defined there are currently used in stream library.

| `void forEach(Consumer<? super T> action)`                                                                                                                                                                                                                                                                               |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| The _**forEach**_ method will run _action_ function on each element in the stream. The order of elements which sent to `action` is not defined in parallel execution. Also using this method in sequential execution, it is best practice to assume the order of elements which sent to `action` is not defined as well.|

| Terminal operation   |
| :------------------: |

| Stateless operation|
| :-----------------:|

```java
// example:

Arrays.asList(1,2,3)
  .parallelStream()
  .forEach(System.out::println);

// possible output:
// 3
// 1
// 2
```
| `void forEachOrdered(Consumer<? super T> action)`                                                                                                                                                                                                                                                                                                                                                                                                           |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The _**forEachOrdered**_ method will run _action_ function on each element in the stream and the order which elements from the calling stream are sending to `action` is defined by _encounting order_ if exist. Else the order of elements in the calling stream. Note: If element `n` (by order of the calling stream) did not reach yet to the `forEachOrdered` operation, any other element `m > n` which came to `forEachOrdered` will have to wait. |

| Terminal operation   |
| :------------------: |

| Statelful operation|
| :-----------------:|

```java
// example 1:

Arrays.asList(1,2,3)
  .parallelStream()
  // the encounting order is still 1,2,3
  .forEachOrdered(System.out::println);

// output:
// 1
// 2
// 3

// example 2:

Arrays.asList(3,1,2)
  .parallelStream()
  // encounting order is 3,1,2
  .sorted() 
  // encounting order is 1,2,3 because sorted operation determine new order.
  .forEachOrdered(System.out::println);

// output:
// 1
// 2
// 3
```
| `<R> Stream<R> map(Function<? super T,? extends R> mapper)`                                                                                                                                                                                                                                                                         |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The _**map**_ method is meant to create from each element in the stream a new element. The type of the new element can be different from the old element. |

| Modify flags:  | `DISTINCT` - `C` | `SORTED` - `C`   | 
| :------------: | :---------------:| :--------------: |

| Intermediate operation   |
| :----------------------: |

| Stateless operation|
| :-----------------:|

```java
// example 1:

Arrays.asList(1,2,3)
  .stream()
  .map((Integer x)->x+4)
  .forEach(System.out::println);

// output:
// 5
// 6
// 7

// example 2:

collection1.stream()
      .map((Integer x)->"hi"+x)
      .forEach(System.out::println);

// output:
// hi1
// hi2
// hi3
```

| `Stream<T> filter(Predicate<? super T> predicate)` |
| :--- |
| The _**filter**_ operation will determine for each element in the stream if that element will stay or not. The method will run on each element in the stream and check using a condition specified in the _predicate_ function if the element should be in the next stream. The next stream will contain all the elements from the last stream that _predicate_ return true for them. |

| Modify flags:  | `SIZED` - `C` |
| :------------: | :------------:|

| Intermediate operation  |
| :----------------------:|

| Stateless operation|
| :-----------------:|

```java
// example:

Arrays.asList(1,2,3).stream()
      .filter((Integer x)-> x > 1)
      .forEach(System.out::println);

// output:
// 2
// 3
```

| `Stream<T> peek(Consumer<? super T> action)` |
| :--- |
| _**Peek**_ will run an action on each element of the stream and will return a stream containing the same elements. This operation is mainly to support debugging. For example, we can print the elements after each operation in the pipeline to see how the stream is calculating the elements. Also, it can be used for _side effects_. |

| Intermediate operation  |
| :----------------------:|

| Stateless operation|
| :-----------------:|

```java
// example:

Arrays.asList("aaa","bbb")
        .stream()
        .filter((String str1)->str1.contains("a"))
        .peek(System.out::println)
        .map((String str1)->str1.concat("ccc"))
        .forEach(System.out::println);

// output:
// aaa
// aaaccc
```

| `Stream<T> sorted()`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The _**sorted**_ method is an intermediate operation and it will create a new stream in which all his elements are sorted according to by natural order. `T` must implement _Comparable_ interface so you can override `int Comparable.compareTo(T o)`. The `sorted` method can't send every element it gets to the next operation in the pipeline because the order of the elements is matters and there is a chance that `sorted` will receive element in the future with smaller value, so this element should be sent first to the next operation in the pipeline. `sorted` use a temporary collection to store all the elements it gets and he starts sorting them from the time it gets executed. |
Notes:
1. If the calling stream is sorted by any other order, then this method will clear the `SORTED` flag and re-inject it after it sorts the incoming elements by natural order. 
2. When `ORDERED` flag is on, For every two elements which are equal by natural order, the first element appears in the encounter order will enter to the output stream and the second will be filtered out.

| Modify flags: | `SORTED` - `CI`| `ORDERED` - `I` |
| :------------: | :------------: | :------------: |

| Intermediate operation  |
| :----------------------:|

| Statelful operation|
| :-----------------:|

```java
// example:

Arrays.asList(1,2,3,1,4).stream()
        .filter((Integer x)-> x>1)
        .sorted()
        .forEach(System.out::println);

// Out put:
// 2
// 3
// 4
```
![](http://imgur.com/NJI3wlo.jpg)

| `Stream<T> limit(long maxSize)`                                                                                                                                                                                                                                                                                                                                                                                                                                |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The _**limit**_ method is an intermediate operation that will create a new stream contains at most `n` elements where `n`=`maxSize`. When `n` elements passed him, or the last operation can't give more to `limit`, `limit` will end the execution of the **all** the stream. When `ORDERED` flag is on, `limit` will wait for the first `n` elements in the encounter order. It means that in case every element entered the `limit` operation except the first `n` elements, all those elements will wait there until the first `n` elements in the encounter order will come or `m` elements from the first `n` will be filtered out during the process until the `limit` operation. In this case, the first `m` elements after the first `n` elements in the encounter order will pass the `limit` operation. |

| Modify flags: | `SIZED` - `C`| `SHORT_CIRCUIT` - `I`|
| :------------: | :-----------:| :-------------------:|

| Intermediate operation  |
| :----------------------:|

| Statelful operation |
| :-----------------: |

```java
// example 1:

Arrays.asList(1,2,3,4,5).stream()
        .limit(2)
        .filter((Integer x)-> x > 1)
        .forEach(System.out::println);

// Out put:
// 2
```

![](http://imgur.com/ciubvZl.jpg)

```java
// example 2: (filter and limit swapped)

Arrays.asList(1,2,3,4,5).stream()
        .filter((Integer x)-> x > 1)
        .limit(2)
        .forEach(System.out::println);

// Out put:
// 2
```

![](http://imgur.com/oiDX6lN.jpg)

| `<A> A[] toArray(IntFunction<A[]> generator)`                                                                                                                                                                                                                     |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The _**toArray**_ method is a terminal operation that will return a new array of type `A` containing all the elements from the last operation in the pipeline. If `SIZED` flag is on, then the size of the generated array is known when `toArray` operation starts. In this case, it will initialize an array with the exact amount of cells needed. If `SIZED` is off then `toArray` will guess how many spaces it needs in the array and it may be a case when he initializes an array with fewer cells then he needs so multiple initializations will be done. Each time it will copy the elements from the last array to the new array.  `generator` receives Integer `n` and must return `A[]`. The array length must be at list equal to `n`. |

| Teminal operation|
| :---------------:|

| Statelful operation |
| :-----------------: |

```java
// example:

Integer[] array=Arrays.asList(3,2,1).stream()
            .sorted()
            // the following line is eqivalent to:
            //.toArray(Integer[]::new);
            .toArray((int n)->new Integer[n]); 
            
Arrays.stream(array).forEach(System.out::println);

// output:
// 1
// 2
// 3
```

| `IntStream mapToInt(ToIntFunction<? super T> mapper)`                                                                                                                                        |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Same as `map` operation, this method receive a function `mapper` whos return type is `Integer`. The return stream is a special stream with built-in functions to deal with Integer elements. |

| Modify flags: | `DISTINCT` - `C`| `SORTED` - `C`|
| :------------: | :--------------:| :------------:|

| Intermediate operation  |
| :----------------------:|

| Stateless operation |
| :-----------------: |

```java
// example:

Arrays.asList(3,2,1).stream()
        // notice that map get function with Integer
        // parameter because the parameter type is
        // generic and the return type must be
        // int because we convert the stream to IntStream.
        // the following line is equivalent to:
        // .mapToInt(Integer::intValue)
        .mapToInt((Integer x)-> x)
        // notice that map get function with int
        // parameter instead of Integer and the
        // return type must be int.
        .map((int x)->x*10)
        .forEach(System.out::println);

// output:
// 30
// 20
// 10
```

| `Optional<T> findAny()` |
| :--|
| The _**findAny**_ method is a terminal operation that will return one element from the caller stream inside an `Optional` object. This operation may return a different element for each execution without giving any consideration to the encounter order (if it exists). In case the caller stream is empty for any reason, return an empty `Optional<T>` object. This operation has much better performance than `findFirst` because it will return the first element came to it and it does not guarantee to return the first element in the calling stream.

| Modify flags: | `ORDERED` - `C`| `SHORT_CIRCUIT` - `I`|
| :------------: | :-------------:| :-------------------:|

| Terminal operation|
| :----------------:|

| Stateless operation |
| :-----------------: |

```java
// example:

Arrays.asList(14,13,12,11,10)
      .stream()
      .sorted()
      .findAny()
      .map((Integer element)-> element * element)
      .ifPresent(System.out::println);

// possible output:
// 100
```

| `T reduce(T identity, BinaryOperator<T> accumulator)` |
| :--- |
| In a synchronized execution, the **_reduce_** operation use the _identity_ value only once as an initial temporary result. The calculating is the same as the first overload we saw. In case of parallel running, it will work as described above. In case the calling stream doesn't have any elements, the `reduce` method return the _identity_ value as a result. |

| Terminal operation|
| :----------------:|

| Statelful operation |
| :-----------------: |

``` java
// example: finding max.

Integer result = Arrays.asList(2,3,4,5,6)
                        .stream()
                        .reduce(Integer.MIN_VALUE, (Integer resultUntilNow, Integer element)->
                        Math.max(resultUntilNow,element));
System.out.println(result);

// output:
// 6
```
![](http://i.imgur.com/2CUx0Cd.jpg)

| `<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)` |
| :--- |
| In a synchronized execution running, the _**reduce**_ operation uses the _identity_ value only once as an initial temporary result. The _identity_ type can be different from the elements in the calling stream. The calculating is the same as the first overload we saw. In case of parallel running, it will work as described above. In case the calling stream as no elements, return the _identity_ value as a result. |

| Terminal operation|
| :----------------:|

| Statelful operation |
| :-----------------: |

```java
// example: joining all elements

String result = Arrays.asList(2,3,4,5,6)
                      .stream()
                      .reduce("", (String resultUntilNow, Integer element)-> resultUntilNow+element,
                      (String partialResult1,String partialResult2)->partialResult1.concat(partialResult2));

System.out.println(result);

// output:
// 23456
```

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
Based on a work at https://gist.github.com/stavalfi/003cfe716051029eee983300895eb830.

