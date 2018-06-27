# Java streams - Part 1 - What is a Java stream? 
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://gist.github.com/stavalfi/969539b245fd71f18ecd14f48eed2a5d)

### Topics

1. [Introduction](#introduction)
2. [Basic operations](#chapter-1-basic-operations)
3. [Operation types](#chapter-2-operation-types)
4. [Conclusion](#conclusion)
5. [Legal](#legal)

### Introduction

A Java stream is a collection of elements that can be queried in a declarative way by a pipeline.

> **Definition 1.1.** _**Pipeline**_ is a couple of methods that will be executed by a specific order.

By query, we mean that some \(sometimes all\) operations in the pipeline will do something with each element in the source collection and calculate with all the elements, that successfully survived all the pipeline, a new value or a collection.

### Chapter 1: Basic operations

Some of the basic operations that a pipeline can have are the following:

---

| `void forEach(Consumer<? super T> action)` |
| :--- |
| The _**forEach**_ method will run _action_ function on each element in the stream. |

```java
// example:

// create the collection
List<Integer> collection1= Arrays.asList(1,2,3);

// create new stream from collection1
Stream<Integer> stream1= collection1.stream();

// print each element in stream1
stream1.forEach((Integer x)-> System.out.println(x));

// the above is equivalent to:
// collection1.stream().forEach(System.out::println);

// output:
// 1
// 2
// 3
```

---

| `<R> Stream<R> map(Function<? super T,? extends R> mapper)` |
| :--- |
| The _**map**_ method is meant to create from each element in the stream a new element. The type of the new element can be different from the source element. This method will not cause the stream to do anything. In each pipeline, we need a method that will make everything run, for example - the _forEach_ method as we saw. |

```java
// example:

// create the collection
List<Integer> collection1= Arrays.asList(1,2,3);

// create new stream from collection1
Stream<Integer> stream1= collection1.stream();

// create new stream with the integers: 5,6,7
// Note: we created new values instead of changing collection1's values.
Stream<Integer> stream2= stream1.map((Integer x)->x+4);

// each stream object has the same methods
stream2.forEach(System.out::println);

// the above is equivalent to:
// collection1.stream().forEach(System.out::println);

// output:
// 1
// 2
// 3

collection1.stream()
      .map((Integer x)->"hi"+x)
      .forEach(System.out::println);

// output:
// hi1
// hi2
// hi3
```

---

| `Stream<T> filter(Predicate<? super T> predicate)` |
| :--- |
| The _**filter**_ operation will determine for each element in the stream if that element will stay or not. The method will run on each element in the stream and check using a condition specified in the _predicate_ function if the element should be in the next stream. The next stream will contain all the elements from the last stream that _predicate_ return true for them. |

```java
// example:

// create the collection
List<Integer> collection1= Arrays.asList(1,2,3);

// create new stream from the collection
collection1.stream()
      .filter((Integer x)-> x>1)
      .forEach(System.out::println);

// output:
// 2
// 3
```

| `Stream<T> peek(Consumer<? super T> action)` |
| :--- |
| _**Peek**_ will run an action on each element of the stream and will return a stream containing the same elements. This operation is mainly to support debugging. For example, we can print the elements after each operation in the pipeline to see how the stream is calculating the elements. Also, it can be used for _side effects_. |

```java
// example:

Arrays.asList("aaa","bbb")
        .stream()
        .filter((String str1)->str1.contains("a"))
        .peek((String str1)-> System.out.println(str1))
        .map((String str1)->str1.concat("ccc"))
        .forEach((String str1)-> System.out.println(str1));

// the above is equivalent to:
//Arrays.asList("aaa","bbb")
//         .stream()
//         .filter((String str1)->str1.contains("a"))
//         .peek(System.out::println)
//         .map((String str1)->str1.concat("ccc"))
//         .forEach(System.out::println);

// output:
// aaa <- printed by peek
// aaaccc <- printed by forEach
```
### Chapter 2: Operation types

We can see that in every pipeline there are 2 types of operations:

1. Operations that don't activate the stream so more operations can come after them in an aggregation \(e.g. `map`, `filter`\).
2. Operations that activate the stream and doesn't allow more operations to be after them \(e.g. `forEach`\).

> **Definition 1.2.** _**Intermidiate operations**_ will be located everywhere in the stream expect at the end. They return a stream object and does not execute any operation in the pipeline.
>
> **Definition 1.3.** _**terminal operations**_ will be located only at the end of the stream. They execute the pipeline. They do not return stream object so no other_Intermidiate operations_ or _terminal operations_ can be added after them.

All the stream methods mention so far: `map`, `filter`, `forEach`- In their execution, they don't need to remember old elements that already calculated. For example,`map` calculate a new value from the value they receive each time.

A stream will take each element in the source collection and will send him throughout all the operations in the pipeline before dealing with the next element in the source collection. Well, that's a lie, it only behaves like that if all the operations in the pipeline are stateless. In the next chapters, we will investigate stateful operations and when a stateless operation becomes stateful and vise versa. For now, let's view the order of a calculation for each element in the source control in case all the operations in the pipeline are stateless:

```java
Arrays.asList(1,2,3).stream()
        .filter((Integer x)-> x>1)
        .map((Integer x)->x*10)
        .forEach(System.out::println);

source collection: 1, 2 ,3

filter(1) -> You are not OK. Element 1 will not pass to the next operation
in the pipeline. Now deal with element 2.
filter(2) -> You are OK. element: 2 passes to the next operation.
map(2) -> create new element: 20 and put it in the new stream.
forEach(20) -> print 20. Then, end dealing with the element: 2 in the source collection.
Now deal with the element: 3.
filter(3) -> You are OK. element 3 pass to the next operation
map(3) -> create new element 30 and put it in the new stream.
forEach(20) -> print 30. No more elements in the source collection.
Finish executing the stream.

output:
20
30
```

![](http://imgur.com/O8pOesF.jpg)

For examining the order of the operations on elements with a list a single stateful operation, let's review the _sorted_ method:

| `Stream<T> sorted()` |
| :--- |
| The _**sorted**_ method is an intermediate operation and it will create a new stream that all his elements are sorted according to by natural order. `T` must implement _Comparable_ interface so you can override `int Comparable.compareTo(T o)`. The `sorted` method can't send every element it gets to the next operation in the pipeline because the order of the elements is matters and there is a chance that `sorted` will receive element in the future with smaller value, so this element should be sent first to the next operation in the pipeline. `sorted` use a temporary collection to store all the elements it gets and he starts sorting them from the time it gets executed. |

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

Another essential intermediate operation is `limit`. As the name implies, we use it to limit the number of elements we send to the next operation in the pipeline.

| `Stream<T> limit(long maxSize)` |
| :--- |
| The _**limit**_ method is an intermediate operation that will create a new stream that contains at most `n` elements where `n`=`maxSize`. When `n` elements passed him or the last operation can't give more to `limit`, `limit` will end the execution of the **all** the stream. To answer the question if this operation is stateful or not, we need to understand who are those `n` elements that `limit` let them pass to the next operation in the pipeline. For now, let's think that this operation is stateless and `limit` lets the first `n` elements that he gets to pass at most. |

```java
// example 1:

Arrays.asList(1,2,3,4,5).stream()
        .limit(2)
        .filter((Integer x)-> x>1)
        .forEach(System.out::println);

// Out put:
// 2
```

![](http://imgur.com/ciubvZl.jpg)

```java
// example 2: (filter and limit swapped)

Arrays.asList(1,2,3,4,5).stream()
        .filter((Integer x)-> x>1)
        .limit(2)
        .forEach(System.out::println);

// Out put:
// 2
```

![](http://imgur.com/oiDX6lN.jpg)


As we will see later, we can construct a stream with an infinite amount of elements. In that case, does `sort` will wait forever until the next operation after him in the pipeline will execute? Yes.

> **Definition 1.4.** Intermediate operations will be called _**short-circuiting operation**_ if that operation produces a new stream with a finite amount of elements from `n` elements where `n` can be an infinite number.
>
> **Definition 1.5.** Terminal operations will be called _**short-circuiting operation**_ if it can finish his execution after a finite amount of time.

Using `limit` will turn each operation that comes after him in the pipeline into _circuiting operation_.

The last operation that we will examine in the first chapter is `toArray`.

| `<A> A[] toArray(IntFunction<A[]> generator)` |
| :--- |
| The _**toArray**_ method is a terminal operation that will return a new array of type `A` containing all the elements from the last operation in the pipeline. For now, let's think that `toArray` know exactly how many elements he will receive throughout his execution. `generator` receives Integer `n` and must return `A[]`. The array length is at list equal to `n`. |

```java
// example:

// convert an unsorted list of integers to a sorted array of integers.
Integer[] array=Arrays.asList(3,2,1).stream()
            .sorted()
            .toArray((int n)->new Integer[n]);

// the above is equivalent to:
// Integer[] array=Arrays.asList(3,2,1).stream()
//         .sorted()
//         .toArray(Integer[]::new);

// print the array
Arrays.stream(array).forEach(System.out::println);

// output:
// 1
// 2
// 3
```

In the above example, `toArray` operation returned an array of type `Integer` instead of type `int`. That is because `toArray` return array of generic type and Java doesn't allow primitive to function as generic types. Then how can we receive an array of primitive type from the stream? We must build a class or a method that will solve this problem, and that method must know we are interested in the array of a primitive type.

### Conclusion

In this chapter we learned that every stream consists of:

1. Data source.
2. Chain of zero or more intermediate operations.
3. single terminal operation.

In the next chapter, we will discover `IntStream` and other streams that deal with primitive types.

---

### Legal

© Stav Alfi, 2018. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Java streams - Part 1 - What is a Java stream?" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/6c385f8360091cc33b6701bd4ee5c242.
