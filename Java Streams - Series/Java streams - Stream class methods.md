# Java streams - Additional - Stream class methods 
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://github.com/stavalfi/lectures/tree/master/Java%20Streams%20-%20Series)

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