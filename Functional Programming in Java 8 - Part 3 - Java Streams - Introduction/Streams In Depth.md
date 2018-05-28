# Streams In Depth By Stav Alfi

### Topics

1. [Introduction](#chapter-1-introduction)
2. [What is a stream?](#chapter-2-what-is-a-stream)
3. [Primitive steams](#chapter-3-primitive-streams)
4. [Optional class](#chapter-4-optional-class)
5. [Reduction](#chapter-5-reduction)
6. [Collectors](#chapter-6-collectors)
7. [Spliterator](#chapter-7-spliterator)
8. [Parallel Streams](#chapter-8-parallel-streams)
9. [Quiz](#chapter-9-quiz)
10. [Conclusion](#chapter-10-conclusion)
11. [Legal](#legal)

### Additional

1. [Complete terms table](#complete-terms-table)
2. [Spliterator characteristics table](#spliterator-characteristics-table)
3. [Collections characteristics table](#collections-characteristics-table)
4. [Stream methods characteristics table](#stream-methods-characteristics-table)
5. [Stream class methods](#stream-class-methods)
6. [Collectors class methods](#collectors-class-methods)
7. [Spliterator interface methods](#spliterator-interface-methods)
8. [Optional class methods](#optional-class-methods)

---

### Chapter 1: Introduction

In this toturial we will cover the stream library from the basic operations and learn each relevant classes. The tutorial designates to developers with absolutliy no expirience with streams to more expirienced developered who want to clarify their knowledge. As we go deeper I will introduce new phrases and add more details to every method.

At the end of the tutorial, the reader will have all the necessary information about how each operation works and how to build a query that uses everything stream can give you.

**Prerequirements**

It is highly recommended that you will have basic knowledge about templates
Also a full understanding about lambda expressions is a must.

[Lambda Expressions In Depth By Stav Alfi](https://gist.github.com/stavalfi/e24178288973d104042d4162a02fd135)

---

### Chapter 2: What is a stream?

###### Basic operations

A stream is a collection of elements that can be queried in a diclerative way by a pipeline.

> **Definition 1.1.** _**Pipeline**_ is a couple of methods that will be excuted by a specific order.

By query we mean that some \(sometimes all\) operations in the pipeline will do something with each element in the source collection and calculate with all the elements ,that succesfully survived all the pipeline, a new value or a collection.

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
| The _**map**_ method is ment to create from each element in the stream a new element. The type of the new element can be different from the source element. This method will not cause the stream to do anything. In each pipeline we need a method that will make everything run, for example - the _forEach_ method as we saw. |

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
| _**Peek**_ will run an action on each element of the stream and will return astream containing the same elements. This operation is mainly to support debugging - for example we can print the elements after each operation in the pipeline to see how the stream is calculating the elements. Also it can be used for _side effects_. |

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
###### Operation types

We can see that in every pipeline there are 2 types of operations:

1. Operations that doesn't activate the stream so more operations can come after them in an aggrigation \(e.g. `map`,`filter`\).
2. Operations that activate the stream and doesn't allow more operations to be after them \(e.g. `forEach`\).

> **Definition 1.2.** _**Intermidiate operations**_ will be located everywhere in the stream expect at the end. They return a stream object and does not excute any operation in the pipeline.
>
> **Definition 1.3.** _**terminal operations**_ will be located only at the end of the stream. They excute the pipeline. They does not return stream object so no other_Intermidiate operations_ or _terminal operations_ can be added after them.

All the stream methods mentions so far: `map`,`filter`,`forEach`- In thier excution, they doesn't need to remmber old elements thet allready calculated. For example,`map` calculate new value from the value it get each time.

A stream will take each element in the source collection and will send him throughout all the operations in the pipeline before dealing with the next element in the source collection. Well thats a lie, it only behaves like that if all the operations in the pipeline are stateless. In the next chapters we will investigate statefull operations and when a stateless operation becomes statefull and vise versa. For now lets view the order of a calculation for each element in the source control incase all the operations in the pipeline are stateless:

```java
Arrays.asList(1,2,3).stream()
        .filter((Integer x)-> x>1)
        .map((Integer x)->x*10)
        .forEach(System.out::println);

source collection: 1, 2 ,3

filter(1) -> You are not OK. Element 1 will not pass to the next operation
in the pipeline. Now deal with element 2.
filter(2) -> You are OK. element 2 pass to the next operation.
map(2) -> create new element 20 and put it in the new stream.
forEach(20) -> print 20. End dealing with element 2 in the source collection.
Now deal with element 3.
filter(3) -> You are OK. element 3 pass to the next operation
map(3) -> create new element 30 and put it in the new stream.
forEach(20) -> print 30. No more elements in the source collection.
finish excuting the stream.

output:
20
30
```

![](http://imgur.com/O8pOesF.jpg)

For exmining the order of the oprations on elements with at list a single statefull operation, lets review the _sorted_ method:

| `Stream<T> sorted()` |
| :--- |
| The _**sorted**_ method is an intermidiate operation and it will create new stream that all his elements are sorted according by natural order. `T` must implement _Comparable_ interface so you can override `int Comparable.compareTo(T o)`. The `sorted` method can't send every element it gets to the next operation in the pipeline because the order of the elements is matters and there is a chance that `sorted` will receive element in the future with smaller value so this element should be sent first to the next operation in the pipeline. `sorted` use a temporery collection to store all the elements it gets and he start sorting them from the time it gets excuted. |

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

Another importent intermidiate operation is `limit`. As the name implies, we use it to limit the number of elements we send to the next operation in the pipeline.

| `Stream<T> limit(long maxSize)` |
| :--- |
| The _**limit**_ method is an intermidiate operation that will create new stream that contain at most `n` elements where `n`=`maxSize`. When `n` elements passed him or the last operation can't give more to `limit`, `limit` will end the excution of the **all** the stream. To answer the question if this operation is stateful or not, we need to understand who are those `n` elements that `limit` let them pass to the next operation in the pipeline. For now lets think that this operation is stateless and `limit` lets the first `n` elements that he get to pass at most. |

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
// example 2: (filter and limit swaped)

Arrays.asList(1,2,3,4,5).stream()
        .filter((Integer x)-> x>1)
        .limit(2)
        .forEach(System.out::println);

// Out put:
// 2
```

![](http://imgur.com/oiDX6lN.jpg)


As we will see later, we can construct a stream with infinite amout of elements. In that case does `sort` will wait forever until the next operation after him in the pipeline will excute? Yes.

> **Definition 1.4.** Intermidiate operations will be called _**short-circuiting operation**_ if that operation produce a new stream with finite amount of elements from `n` elements where `n` can be infinite number.
>
> **Definition 1.5.** Terminal operations will be called _**short-circuiting operation**_ if it can finish his excution after finite amount of time.

Using `limit` will turn each operation that comes after him in the pipeline into _circuiting operation_.

The last operation that we will examine for the first chapter is `toArray`.

| `<A> A[] toArray(IntFunction<A[]> generator)` |
| :--- |
| The _**toArray**_ method is a terminal operation that will return a new array of type `A` containing all the elements from the last operation in the pipeline. For now lets think that `toArray` know exactly how many elements he will receive throughout his excution. `generator` receives Integer `n` and must return `A[]`. The array length is at list equal to `n`. |

```java
// example:

// convert unsorted list of integers to sorted array of integers.
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

In the above example, `toArray` operation returned an array of type `Integer` istead of type `int`. That is beacuse `toArray` return array of generic type and java doesn't allow primitive to function as generic types. Then How can we receive an array of primitive type from stream? We must build a class or a method that will solve this problem and that method must know that we are interseted in array of a primitive type.

In this chapter we learned that every stream consist of:

1. Data source.
2. Chain of zero or more intermediate operations.
3. single terminal operation.

In the next chapter we will discover `IntStream` and other streams that deal with primitive types.

---

### Chapter 3: Primitive streams

The interfaces `IntStream`, `DoubleStream`, `LongStream` implement `BaseStream` interface. In addition to `Stream` interface methods, They expose additional methods to query their elements by knowing their type. As you can guess, now we **nicely** calculate the sum, average, min, max of the elements in the stream.

In this chapter we will examine some of the additional methods from `IntStream` interface is offering compired to `Stream` interface. The complete list of the methods is located in this chapter - [IntStream methods](#intstream-methods)

The first method we will learn is the one who convert`Stream<Integer>` to `IntStream`.

| `IntStream mapToInt(ToIntFunction<? super T> mapper)` |
| :--- |
| Same as `map` operation, this method receive a function `mapper` whos return type is `Integer`. |

```java
// example:

Arrays.asList(3,2,1).stream()
            // notice that map get function with Integer parameter because
            // the parameter type is generic and the return type must be
            // int because we convert the stream to IntStream.
            .mapToInt((Integer x)->x)
            // notice that map get function with int parameter instead of
            // Integer and the return type must be int.
            .map((int x)->x*10)
            .forEach(System.out::println);

// the above is equivalent:
// Arrays.asList(3,2,1).stream()
//          .mapToInt(Integer::intValue)
//          .map((int x)->x*10)
//          .forEach(System.out::println);

// output:
// 30
// 20
// 10
```

| `int sum()` |
| :--- |
| This is a terminal operation which return the sum of all the elements in the stream who called this method. |

```java
// example:

int sum = Arrays.asList(3,2,1).stream()
                .mapToInt(Integer::intValue)
                .sum();

System.out.println(sum);

// output:
// 6
```

Due to similarities, I will skip `min`,`max` and `summaryStatistics`.

| `<U> Stream<U> mapToObj(IntFunction<? extends U> mapper)` |
| :--- |
| This is an intermediate operation. `mapToObj` create from each integer an object from type `U`. This method only purpose is to convert `IntStream` to `Stream`. |

```java
// example:

Arrays.asList(1,2,3).stream()
                    // convert Stream<Integer> to IntStream.
                    .mapToInt(Integer::intValue)
                    // invoke some intermediate operations on IntStream.
                    // ...
                    // ...
                    // ...
                    // convert IntStream to Stream<String>.
                    .mapToObj((int x)-> "element value is: "+x)
                    .forEach(System.out::println);

// output:
// element value is: 1
// element value is: 2
// element value is: 3
```

Incase we want to convert `IntStream` to `Stream<Integer>` we can use `boxed`:

| `Stream<Integer> boxed()` |
| :--- |
| This is an intermediate operation. Same as `mapToObj` its main diffrence is the return type is completely known. It is a special case of `mapToObj`. |

```java
// example:

Arrays.asList(1,2,3).stream()
                    // convert Stream<Integer> to IntStream.
                    .mapToInt(Integer::intValue)
                    // convert IntStream to Stream<Integer>.
                    .boxed()
                    .forEach(System.out::println);

// output:
// 1
// 2
// 3
```

---

### Chapter 4: Optional class

Some terminal operations return an object that is made or equeal from/to one or more elements in the stream. What happends when the source collection is empty or non of the elements in the source collection survived all the pipeline until coming to the terminal operation?

Returning `null` is an option but a bad one. The following reasons may couse `NullPointerExcaption`:

1. We forgot to initilize an object.
2. We received a `null` pointer from a function and we did not check it.

Let's focus on the second reason - Do we know if the null we received is a bug of the programmer who wrote the function or the function should return us `null` for that call?

To avoid some of the reasons for `NullPointerExcaption`, Java 8 introduced us the `Optional<T>` type. It comes with helpeful intermidiate and terminal operations to deal with the absense or the present of a given value. Comparing to the intermidiate operations of streams, the optional's intermidiate operations will be invoked even when a terminal operation is present in the pipeline.

We will examine the basic operations of `Optional<T>` and then see some stream's terminal operations that return an `Optional<T>`:


| `public boolean isPresent()` |
| :--|
| The _**isPresent**_ terminal operation return true if it represent a existing value, else return false. |

| `public void ifPresent(Consumer<? super T> consumer)` |
| :--|
| If the object who call this method represent a existing value, the _**isPresent**_ terminal operation will invoke `consumer` function on the existing value. Else do nothing. |

| `public static <T> Optional<T> ofNullable(T value)' |
| :--|
| The _**ofNullable**_ function will return an optional representing a value. Use it **only** when you aren't sure if `value` is equal to `null`. if `value` is equal to `null`, this function will return an empty optional object (invoking `isPresent` terminal operation on an empty optional object return `false` and an empty optional object is not garentee to be singleton.) |

| `public static <T> Optional<T> of(T value)' |
| :--|
| The _**of**_ function will return an optional representing a non-null value. Use it **only** when you are absolutly sure that `value` is not equal to `null`. A `NullPointerExcaption` will be thrown incase `value` is equal to `null`. |

``` java
// example:

Optional.of(123)
.ifPresent((Integer existingValue)-> System.out.println(existingValue));

// the above is equivalent to:
// Optional.of(123)
//         .ifPresent(System.out::println);

// output:
// 123
```

| `public <U> Optional<U> map(Function<? super T,? extends U> mapper)` |
| :-- |
| If the object who call this method represent a existing value, the _**map**_ intermidiate operation will invoke `mapper` on existing value. `map` return type is an `Optional<U>`. When the value is absense, return an empty `Optional<U>`. `U` is the return type of `mapper`. |

| `public Optional<T> filter(Predicate<? super T> predicate)` |
| :-- |
| If the object who call this method represent a existing value and `predicate` return `true` then return an `Optional<T>` with the same value. Else return an empty `Optional<T>`. |

```java
// example 1:

Optional.of(1)
        .filter((Integer existingValue) -> existingValue > 0)
        .map((Integer existingValue) -> "optional value is: " + existingValue)
        .ifPresent(System.out::println);

// output:
// optional value is: 123

```

``` java
// example 2:

Optional.of(1)
        .filter((Integer existingValue) -> existingValue > 1)
        .map((Integer existingValue) -> "optional value is: " + existingValue)
        .ifPresent(System.out::println);

// output:
// (no output)
```

After examining the basic functions of `Optional<T>` lets go back to streams and look at a terminal opration who return `Optional<T>` object:

| `Optional<T> findAny()` |
| :--|
| The _**findAny**_ method is a terminal operation that will return one element from the caller stream. It may return a diffrent element for each excution. Later we will see why this behavior is better for performance. Incase the caller stream is empty from any reason, return an empty `Optional<T>` object. |

```java
// example:

Arrays.asList(14,13,12,11,10)
      .stream()
      .sorted()
      .findAny()
      .map((Integer element)-> element*element)
      .ifPresent(System.out::println);

//possible output:
// 100
```

We will cover much more powerful and intersting terminal opertion who return `Optional<T>` obejct in the next chapter.

__conclusion:__

The `Optional<T>` type help us to replace the old `if(value==null)` statement by building a pipeline to deal with the absense or the present of a given value. There are more operations we didn't cover. Go to [Optional class methods](#optional-class-methods) chapter to read more about `Optional<T>` operations.

___


### Chapter 5: Reduction

Reduce in the context of functional programming is an operation
that calculate a single value from a given collection. The reduce operation calculate it's result from each element in the collection. The way he does that is by saving a temporary result and calculating new temporary result from the last temporary result and an element he did not visit yet. The function who calculate the next temporary result is called `accumulator`. we must supply it to the reduce operation.

For example:
If we want to sum all the elements in a given array: `{10`,`11`,`12}` , reduce operation determine the first temporary result to be '10' and the accumolator function to be : `(Integer lastResult, Integer element) -> lastResult + element`.
In the first iteration, `10` which is the first temporary result and `11` which is an unvisited element will be sending to our accumulator for calculating the next temporary result: `21`.
In the second and last iteration, `21` and `12` will be sending to our accumulator for calculating the final result : `33`.

Notes:

1. The accumulator operation does not mutate any temporary result or any element in the collection. It creates new temporary result in each iteration.
2. The accumulator operation must be associative. Meaning for any given parameters, `accumulator.apply(x,y)` equals to `accumulator.apply(y,x)`.
This means that some accumulators can't be use by reduce operation, For example: `-`,`%`,`/`.
3. The reduce operation can run in parallel, but more information on parallel will come later.

There are 2 types of Reduce operations:

> **Definition 5.1.** **_Reduce-left_** operation determine the first temporary result will be the first element in the collection from the left and the reduce operation will visit each element in the collection that he didn't visit yet from left to right.

The example we seen above is a reduce-left operation.

> **Definition 5.2.** **_Reduce-right_** operation determine the first temporary result will be the first element in the collection from the right and the reduce operation will visit each element in the collection that he didn't visit yet from right to left.

Java implement _reduce-left_ operation:

|`Optional<T> reduce(BinaryOperator<T> accumulator)`|
|:--|
| The **_reduce_** operation calculate the result from each element in the collection using accumulator. It is _reduce-left_ operation so the firts element from the left is the initial temporary result. This method return `Optional<T>` because the calling stream may be empty. |

```java
// example: finding sum

Arrays.asList(2,3,4,5,6)
      .stream()
      .reduce((resultUntilNow,element)-> resultUntilNow + element)
      .ifPresent(System.out::println);

// output:
// 20
```
![](http://i.imgur.com/srRPrc3.jpg)

Recude works in parallel by spliting the work to be done to smaller portions.
Each thread take a smaller task and calculating a temporary result using the accumulator and determining an initial temporary result for each portion of the collection. After finishing each task, the provided accumulator combine partial results into one. We can see that the reduce can run in parallel only with a given accumulator and without any additional synchroniztion.

The following image present a parallel computing of the reduce operation when the reduce split the work to 2 threads (can be more).

![](http://i.imgur.com/DkU20og.jpg)

The next overloading of reduce operation in Java receive also an _identity_.

Instead of letting reduce operation determine the first left value to be the initial temporary result, we can declare it and this value is called the _identity_ value.

This type of reducion is called `folding`. Java use fold-left reduction by default but this behavior does not witten in the api so do not count on that in the future.

| `T reduce(T identity, BinaryOperator<T> accumulator)` |
| :--- |
| In a sequential running, the **_reduce_** operation use the _identity_ value only once as an initial temporary result. The calculating is the same as the first overload we saw. Incase of parallel running, it will work as described above. Incase the calling stream doesn't have any elements, reduce return the _identity_ value as a result. |

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

The third and last overloading of reduce operation in Java receive also a _combiner_.

After each thread finished his task, instead of using the provided accumulator function to combine the partial results, we can provide a _combiner_ function for that. The logic of the combiner can be different from the accumulator.
Incase the run is seqential, there will be no use in the provided combiner function.

| `<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)` |
| :--- |
| In a sequential running, the _**reduce**_ operation use the _identity_ value only once as an initial temporary result. The _identity_ type can be different from the elements in the calling stream. The calculating is the same as the first overload we saw. Incase of parallel running, it will work as described above. Incase the calling stream as no elements, return the _identity_ value as a result. |

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

__Conclution:__

The reduce operation does not mutate it's state and create new one state in each iteration. In the last example we saw that the reduce copy multiple strings multiple times. The complexity of that is too high. In the next chapter we will see an alternative approach for this problem but it will cost us something else.

All in all, we have seen that for each reduction we must provide an accumulator function and an optional identity or combiner function. Both the accumulator and the combiner must be stateless and pure functions to support parallel excution.

---

### Chapter 6: Collectors

The reduce operation we have seen above is powerful but come with a price - run time complixity. Sometimes we can efford mutating a state to receive a better complexity.

Java 8 created for us classes who implement the `Collector` interface. The concept of a collector and a reduce operation is similar - both operations calculate a value from multiple values.

| Collector<T,A,R> Interface - Abstract Methods|
| :--- |
| `BiConsumer<A,T> accumulator()` |
| `BinaryOperator<A> combiner()` |
| `Supplier<A> supplier()` |
| `Function<A,R> finisher()` |
| `Set<Collector.Characteristics> characteristics()` |

`T` - The input elements type of the calling stream.
`A` - The return type of the mutable accumulation operation.
`R` - The result type of the finisher operation.

The accumulator and the combiner are familiar to us from the reduce operation. The supplier is replacing the `identity` value we provided, meaning we can manage the creation logic of the `identity` value to each thread inside the scope of the collector.
The finisher is a function that can change the final result of the collector. The last method is the `characteristics` who return a `Set` of flags. Each flag indicates a specific behavior the collector support. All the flags are used by the `Stream.collect` 'terminal operation who run the collector, to gain better performance. We will discuss on flags later.

The main diffrences between reduce and collectors are:

1. Reduce operation forbid mutating any variable inside the accumulator function. Instead of mutating the temporary result, it create and return new temporary result. On the other hand, collectors does not behave the same. We will see that instead of returning new temporary result in each iteration, it will mutate it each time in the accumulator function so this accumulator does not return anything. Nevertheless the collector still support parallel excution.

2. Instead of supply an `identity` value to the reduce, we can provide a lambda (`supplier`) to the collector so the logic of the `identity` value creation stays inside the collector scope only.

3. The reduce operation doesn't support changing it's final result inside it's scope but the collector support this kind of behavior.

4. Pre-defined flags can tell the stream operation who runs the collector how to excute. The following flags can be used:

| `Collector.Characteristics enum`|
| :--- |
| `CONCURRENT` - We can access the result container (collection or an object) from the accumulator function concurrently. In this case, the supplier should be called only once. |
| `IDENTITY_FINISH` - We doesn't change the result container and the finisher function is: `(A container) -> container` |
| `UNORDERED` - The result container doesn't keep his elements in any order. Another way to think about that is, you can't use `for` loop to print the container elements. For example `HashMap`. This flag is ment to tell the `Stream.collect` operation that we do not care in what order the elements will be insert to the final container. For example, if the flag `CONCURRENT` is off, that means we must split the work to smaller temporary containers. meaning each thread will do a smaller task and will insert the elements in diffrent container. Because the container doesn't care about what order we insert elements , we don't need to wait for thread `n` to complete copy his result to a bigger container before thread `n-1` did that. We let any thread who finish his task to copy his result to a bigger container.|

Java 8 wrote for us some collectors we can use for free inside the `Collectors` class. We can use them by calling `Stream.collect` that accept a `Collector`.
The collect terminal operation will use the accumulator, supplier, cimbiner, finisher and the characteristics function from the collector object to run.


| `public static <T> Collector<T,?,List<T>> toList()` |
| :--- |
| The __*toList*__ collector convert the calling stream to a `List<T>`. There is no garentee to which class will be used to create the list. Also no garentee on the thread-safety or the serializability of that class. |

``` java
// example:

List<Integer> list1= Arrays.asList(2,3,4,5)
                            .stream()
                            .filter((Integer x)-> x >= 3)
                            // Collectors.toList() return a collector object!
                            .collect(Collectors.toList());
list1.forEach(System.out::println);

// output:
// 3
// 4
// 5
```

Lets have a look at a potential implementation of the `toList` collector:

``` java
class MyToListCollector<T> implements Collector<T,List<T>,List<T>> // Collector<T,A,R>
{
    // T = T - The input elements type of the calling stream.
    // A = List<T> - The return type of the mutable accumulation operation.
    // R = List<T> - The result type of the finisher operation.

    @Override
    public Supplier<List<T>> supplier() {
        return ()-> new LinkedList<T>();
    }

    @Override
    public BiConsumer<List<T>, T> accumulator() {
        return (List<T> temporaryContainer,T element)-> temporaryContainer.add(element);
    }

    @Override
    public BinaryOperator<List<T>> combiner() {
        return (List<T> temporaryContainer1,List<T> temporaryContainer2)-> {
            // if we report the Collector.Characteristics.CONCURRENT flag,
            // it means the container we are using to store the elements,
            // can be used for adding elements concurrently.
            // In this case, all the threads will use the same container and this
            // function, combiner, won't be called.
            // if we doesn't report the Collector.Characteristics.CONCURRENT flag,
            // it means the container we are using to store the elements,
            // can't be used for adding elements concurrently. In this case each
            // thread run the supplier function to create his temporary container
            // and when he is done adding his elements, he need to combine his
            // temporary container with other temporary containers.
            // In this case this function, combiner, will be called at list once.
            // Inside the combiner, we doesn't need to create new temporaryContainer
            // to hold both the given temporary containers because no other thread is
            // currently using temporaryContainer1 or temporaryContainer2.
            // Again, we didn't report the Collector.Characteristics.CONCURRENT flag
            // so 2 or more threads can't add to the same container.
            temporaryContainer2.forEach((T element)-> temporaryContainer1.add(element));
            return temporaryContainer1;
        };
    }

    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.identity(); // Function.identity() return: (List<T> container) -> container;
    }

    @Override
    public Set<Characteristics> characteristics() {
        Set<Characteristics> characteristics=new HashSet<Characteristics>();
        // For this implementation I choosed, I didn't change the final container
        // in the finisher function so it doesn't need to be called.
        characteristics.add(Characteristics.IDENTITY_FINISH);
        return characteristics;
    }
}
```

More collectors Java 8 provide us inside the `Collectors` class are:

| `public static <T> Collector<T,?,Map<Boolean,List<T>>> partitioningBy(Predicate<? super T> predicate)` |
| :--- |
| This collector partition the incoming elements into 2 groups: The elements who `predicate` return true for them and the second group are the rest. The result container implements `Map`.|

```java
// example:

Map<Boolean,List<Integer>> result= Arrays.asList(2,3,4,5)
              .stream()
              .collect(Collectors.partitioningBy(number-> number % 2 == 0));
              
result.entrySet().forEach(group -> System.out.println(group.getValue().size()));

// output:
// 2
// 2
```

| `public static <T,K> Collector<T,?,Map<K,List<T>>> groupingBy(Function<? super T,? extends K> classifier)` |
| :--- |
| This collector creates a `Map`. Each element is sent to the `classifier` function that generate a key. the value for each key is a collection of all the elements who produce the same key. The collection stored in each value implements `List<T>`. If you want to choose which collection to use, choose a different overload. API Note: In parallel running, each accumulator adds elements to different `Map` object and the combiner is merging them. You should use different function to have better performance: `groupingByConcurrent` |

```java
// example:

Map<Integer,List<Integer>> result= Arrays.asList(1,2,3,4,5,6,7,8)
.stream()
// the keys can be 0 or 1 or 2 or 3 only.
.collect(Collectors.groupingBy( x -> x%4));

result.entrySet().forEach(keyAndValue->
        System.out.println("Key: " + keyAndValue.getKey() +
        " , amount of elements:" + keyAndValue.getValue().size()));
        
// output:
// Key: 0 , amount of elements:2
// Key: 1 , amount of elements:2
// Key: 2 , amount of elements:2
// Key: 3 , amount of elements:2
```
Some collectors who return `Map<T,?>` accept collector as an argument. That is for transforming the elements inside each value to something else. They can transform the elements inside the value into an Integer that represent the amount of all the element who are inside the value or doing something else with those elements. Lets have a look at the overload and some examples:

| `public static <T,K,A,D> Collector<T,?,Map<K,D>> groupingBy(Function<? super T,? extends K> classifier, Collector<? super T,A,D> downstream)` |
| :--- |
| Same behavior as the last overload including the API notes. The differense here is that each element that produce the same key will be sent to the `downstream` collector. Each key will create diffrent `downstream` collector. It means that if we have `n` keys and each key have `m` elements that produce that key, there will be `n` different `downstream` collectors for each `m` elements.|

```java
// example 1:

Map<Integer,Long> result= Arrays.asList(1,2,3,4,5,6,7,8)
          .stream()
          .collect(Collectors.groupingBy((Integer x) -> x%4,
          // Collectors.counting() is a collector that return
          // the number of incoming elements.
          Collectors.counting()));

result.entrySet().forEach(keyAndValue->
          System.out.println("Key: " + keyAndValue.getKey() + 
          " , amount of elements:" + keyAndValue.getValue()));
// output:
// Key: 0 , amount of elements:2
// Key: 1 , amount of elements:2
// Key: 2 , amount of elements:2
// Key: 3 , amount of elements:2
```

The posibility of running second collector inside a collector is very powerful.

```java
// example 2:

Map<Integer,Map<Boolean,List<Integer>>> result= Arrays
              .asList(0,1,10,11,100,101,1000,1001,1002,1003)
              .stream()
              .collect(Collectors.groupingBy(
                    (Integer x) ->x.toString().length() ,
                     Collectors.groupingBy((Integer x) -> x%2 == 0)));

// Describing the result object will be less scary
// then showing what there is inside:
// the length of the numbers are the keys.
// meaning we have 4 different keys in this value:
// 1 (0,1), 2 (10,11), 3 (100,101), 4 (1000,1002,1001,1003).
// Instead of collecting each key's values in a List,
// I used a second group by to seperate them by even and odds.
// meaning the keys will be : true (even) and false (odd).
// All in all result looks like this: // <key,value> is
// an element in a map
// key: 1 , value: <true,0>, <false,1>
// key: 2 , value: <true,10>, <false,11>
// key: 3 , value: <true,100>, <false,101>
// key: 4 , value: <true,1000,1002>, <false,1001,1003>
```

I can give more examples for collectors inside a collector but the principle will be the same.

You will notice that there is a `reducing` collector. This operation can have `identity` value, `accumulator` and a `combiner` same as `Stream.reduce`. Why Java 8 created 2 reducers who operate the same? The answer is to let us use reduction inside a multi-level collectors. For example we can use `reducing` collector inside a `groupingBy` collector etc.

For the full list of all the collectors avidable in Java 8 go to: [Collectors class methods](#collectors-class-methods) chapter.

---

### Chapter 7: Spliterator

In the following paragraphs lets stop think about streams and return the good old collections and how to iterate them. You probably heared or used `Iterators`. they are greate but not sutiable for all the problems we are facing today. One of those is parallel calculating.

As we saw earlier, we can split the calculation on an array to calculate smaller chunks of the same array in parallel and after that, combining the temporary results to one final result. Spliting a collection can be done in diffrent types of collections and not only arrays. Which types of collection can we split? The ones we know their estimated `SIZE`. After spliting a binary tree, does the sub-trees are also spliteable; Do we know their estimated `SIZE`? The answer is yes only for balanced trees. If we do know their `SIZE` after the spliting, we can say that the `SIZE` of the collection is known and also the sub-trees; The collection is also `SUBSIZED`. 

  A `Spliterator` is a new interface in Java 8. Each collection in Java implement and return a spliterator same as returning an iterator. Both can be used to iterate over a given collection but spliterator can also be used to split the work to smaller tasks for the purpose of parallel computing. A spliterator can also represents a generator function or an IO channel. If we choose to split the collection which the spliterator is iterating over and succeed(we soon exemine the reasons for failer), the spliterator will now be responsible to only half+-(we will explain later why not precisely half) the elements he should have iterate befor the spliting and the new spliterator that will be created after the spliting will iterate on the rest of them elements; The other half. We can keep spliting the spliterator work recursively until there is nothing to split or we don't know the estimated `SIZE` of the spliterator (those are the reasons for failer of the spliting operation).

`SIZE` and `SUBSIZED` are only 2 characteristics a spliterator can have. We have 5 more:

| Characteristic|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :-------------| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CONCURRENT`  | It's possible to modify (delete/insert/replace elements) the source of the spliterator concurrently without any additional synchronization.
| `NONNULL`     | The source of the spliterator can't insert null elements so each of the elements is garentee to be non-null value. |
| `IMMUTABLE`   | The source of the spliterator prohibit modification (delete/insert/replace elements). |
| `DISTINCT`    | For each 2 different elements `x` and `y` in the source of the spliterator we can tell that `x.equals(y) == false`. |
| `ORDERED`     | If we will print the elements in the source of the spliterator mutiple times by iterating it, we will print the elements in the same order time after time. That means all the elements are stored in the source of the spliterator in a specific order. We can refer to that situation by specify that the source of the spliterator have _encounting order_. For example: `Array`, `List`,`Stuck` has _encounting order_ but `HashMap` doesn't. We can't iterate over `HashMap`. |
| `SORTED`      | The source of the spliterator is ordered by a comperator or in natural order. |
| `SIZED`       | We know the estimated amount of iterations left in the spliterator. |
| `SUBSIZED`  |  If we would split the task to 2 chunks we would know the estimated amount of iterations each spliterator would have. Note: If `SUBSIZED` is on, then `SIZED` also. The reason is if we know how many iterations will be needed after the split for each spliterator, we must know first how many iterations left without spliting in the first place. The claim doesn't follow for the opposite direction as mentioned above. For example - binary trees. |

Let's examine the basic abstract methods of `Spliterator` interface:

| `boolean tryAdvance(Consumer<? super T> action)`                                                                                                                                                                                                                                                                                                                  |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| If the spliterator did not iterate over all the elements yet, return `true` and perform `action` on the next element in the source of the spliterator. Then advance the spliterator to point to the next element. If there are no elements to iterate over, return false and don't invoke `action` because there is no element to send to the action anyway. If we used `Iterator` instead of `Spliterator` then in this case, we must have called 2 methods instead of one: `hasNext` and `next`. Now the interface is cleaner and less imperative. __Notes:__ __1.__ If `ORDERED` characteristic is off, there is no garentee which element will be visit by the spliterator using this method from the remaining elements. If the encounting order is important. __2.__ Remmber that if he split the work of a spliterator, each spliterator on the source collection will iterate over less elements now then using a single iterator. That means this method is checking how many elements this spliterator have left before finish his sub-task. 

```java
// example:

boolean didWeIterate = Arrays.asList(1,2,3)
        .spliterator()
        .tryAdvance((Integer x)-> System.out.println(x));

System.out.println(didWeIterate);

// output:
// 1
// true
```

| `long estimateSize()`                                                                                                                                                                                                                                                                     |
| :-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| If the `SIZED` characteristic is on and we did not iterate the source yet, return the exact number of elements left to iterate over. Else try to compute an estimated number of iterations left. If it is too expensive to compute or unknown or infinite then return `Integer.MAX_VALUE`.|

| `Spliterator<T> trySplit()`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Even if the `SUBSIZED` or `SIZED` charactersitics is off, this method will try to split the remaining iterations to 2 different spliterators. If it failed to do so or the spiterator that this method is invoked by was iterating befor calling this method or data structure constraints/ efficiency considerations, the method will return null. __Note:__ An ideal splitting will divides the elements exactly in half for parallel computing. For eample, in balanced binary tree, there should be no problem to do so. |

```java
// example:

public static void main(String[] args) {
    Spliterator<Integer> spliterator = Arrays.asList(1,2,3,4).spliterator();
    splitTheWork(spliterator);

    // output:
    // 1
    // 2
    // 3
    // 4
}

static void splitTheWork(Spliterator<Integer> spliterator1) {
  
    // spliterator2 will get the first half and spliterator1 will get the second half
    Spliterator<Integer> spliterator2=spliterator1.trySplit();
    if(spliterator2==null) {
        while (spliterator1.tryAdvance(System.out::println) == true);
        
        // the above line is equal to:
        // spliterator1.forEachRemaining(System.out::println);
        // For more information, see Spliterator interface methods chapter.
    }
    else {
        splitTheWork(spliterator2);
        splitTheWork(spliterator1);
    }
}
```

---
Let's go back to our world: _Streams_.

Streams iterate over the source collection using spliterators. When iterating over a stream in parallel we will use `trySplit` to split the work but the method may return null due to the reasons we mentioned earlier and the rest (or all) the calculation will be performing sequentally.

When we transform the source collection to a stream, the stream uses the spliterator, returned from that collection, to iterate over the elements and each _intermediate operation_ will return a new stream that uses the same spliterator.

Stream uses an internal enum to manage the relevant characteristics for him, `StreamOpFlag`. Each pipeline have it's own flags. The flags at the beggining are initilized to be the relavent characteristicts from the spliterator. An _intermediate operation_ and _terminal operation_ can modify the spliterator characteristicts or the _flags_ of the hole pipeline or exemine the spliterator's characteristics for choosing the best algorithm.
For example, the `filter`  _intermediate operation_ can reduce the number of elements which sent to him. The stream liberary can not guess how many elements left so if the `SIZED` flag and characteristic is on, now they are off.

The following table showes which operations types are allowed to modify charactersiticts:

|                             | `DISTICTS`    | `SORTED`    | `ORDERED`       | `SIZED`        | `SHORT_CIRCUIT`  |
| :-------------------------- | :-----------: | :---------: | :-------------: | :------------: | :---------------:|
| __Source stream__           | `Y`           | `Y`         | `Y`             | `Y`            | `N`              |
| __Intermediate operation__  | `PCI`         | `PCI`       | `PCI`           | `PC`           | `PI`             |
| __Terminal operation__      | `N`           | `N`         | `PC`            | `N`            | `PI`             |

* `Y` - Allowed to have
* `P` - May preserves
* `C` - May clears. 
* `I` - May injects. 
* `N` - Not valid; Irelevant for the operation. 

__Notes:__ 
1. `SHORT_CIRCUIT` means an operation is considerted to be _short circuit operation_ (Explained in chapter 1).
2. For now we can see that the Java stream liberary doesn't use `CONCURRECT`,`NONNULL`, `IMMUTABLE`. That doesn't mean they won't use them in the future. If `SIZED` is off then `SUBSIZED` will be too. For now there is no operation which clears or injects the `SUBSIZED` characteristic.
3. The existence of `SHORT_CIRCUIT` flag impleys that the above flags are not from the `Spliterator` Interface characterstics but from `StreamOpFlag` enum that uses the relevant flags he need for streams optimizations.

The following table showes which characteristics and flags each _intermediate operation_ may turns on and off: (`SHORT_CIRCUIT` is relevent only in the context of `StreamOpFlag` flags)

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

```java
// example:

Arrays.asList(1,2,3,4,5,6) // ORDERED SIZED SUBSIZED
        .stream() // no change -> ORDERED SIZED SUBSIZED
        .distinct() // ORDERED DISTINCT 
        .filter((Integer element) -> element > 4) // ORDERED DISTINCT 
        .sorted() // ORDERED DISTINCT SORTED 
        .forEach(System.out::println); // no change -> ORDERED DISTINCT SORTED 
```
---

### Chapter 8: Parallel Streams

Streams have the ability to split the source collection to multiple threads  by letting each thread work on different area in the source collection. We use `Stream.parallel` intermediate operation to get a stream that will __try to__ be parallel stream. Each stream operatrion will exemine the stream flags and the spliterator characteristics to see if a parallel execution is worth. If intermidtae operation `A` agreed to execture in parallel, all the stateless intermediate operations after `A` will do too until we meet a stateful operation `B` that has it's own rules. 

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

###### Concurrent vs Parallelism

There are alot of definitions to _concurrent_ and _parallel_ programming..I will introduce a set of definitions that will serve us best.

> **Definition 8.1.** **_Cuncurrent programming_** is the ability to solve a task using additional synchronization algorithms.

> **Definition 8.2.** **_Parallel programming_** is the ability to solve a task  without using additional synchronization algorithms.

Stream liberary is witten using additional synchronization algorithms but we have the ability to use it's methods without any. When we write _reducers_ or _collectors_ or make a use of the built-in _reducers_/_collectors_ stream gave us, we do not need to use any additional synchronization algorithms to solve our task. 
The only think we need to do is to understand how to split the big problem to smaller tasks.

The question we need to ask is when do we need to upgrade sequential solution to parallel one? 
* You have lot's of data to calculate so the parallel calculation will make a difference.
* Not every problem that can be solved in _Cuncurrent programming_ can be solved with Parallel programming; Spliting a task to independent smaller tasks is not always posible.
* Even if we succced to split the task to smaller independent tasks, we still need to store our temporary results. Keep in mind the space complexity needed.

Write each pipeline to support parallelizm although it won't be executing in parallel by your or the liberary choice.

---

###### Fork / Join Framework Introduction

  Stream liberary as other Java's standart liberaries using a common thread pool to assign tasks to threads. As it's name suggest, it's threads are used by everyone. Keep in mind that if you are using parallel stream on the common thread pool to calculate task that may stuck because they wait for a server response or they call to I/O blocking operation, then you will stack each part of your system that uses the common thread pool also at the same time.
  
With  Fork / Join Framework we can create new thread pools. The advantage is solving the problem I introduced above but the drawback is we may create too much threads that does not work.

The thread pool asign tasks to threads which are not bussy at the moment. After a thread finish it's task, it will wait for the next task and so on. Each thread pool manage a stuck of tasks (functions) for each of his threads - a thread will take a task from __his__ stuck and if it's empty he will take from other thread's stuck.

The common thread pool contain `N` threads. `N` is equal to the number of cores the computer have  `- 1` .  The `-1` is for letting the `main` run also without letting fight for exection time.

You can force the stream libarery to use different thread-pool by invoking the stream inside a task that will run in different thread-pool as we will see soon.

In this chapter we won't exemine each class and method this liberary proved but only the relevant for our need. I recommend any reader to exemine for him self the parts that are not covered here after reading this tutorial.

The following `Executors` functions let us create new different thread pools:

| `public static ExecutorService newSingleThreadExecutor()`                                                                                                                                                         |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Create and return a new thread pool with only one thread that will execute the tasks. The thread pool can not be configured to use more then one thread. If the thread is terminated, new thread will be created. |

| `public static ExecutorService newFixedThreadPool(int nThreads)`  |
| :---------------------------------------------------------------- |
| Create and return a new thread pool containing `nThreads` that take tasks from __one__ stuck. If any thread is terminated, new thread will be created. |

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
  ### Chapter 9: Quiz
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

Does the following code compiles? If yes, what does it print? If not, what is the reason and Java implementors didn't allow it?
```java
Stream.of(1, 2, 3)
        .filter((Integer number) -> number > 1)
        .findFirst()
        .sorted()
        .ifPresent((Integer number) -> System.out.println(number));
```

[Answer 3](#3-question-with-answer)

###### Question 4

What methods the `Person` class must implement to make the following code works? (I mean, what `sorted` operation invokes in `Person` class?)

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

Does this code compiles?

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

Can we improve the readability and the safty of the following code?

```java
List<Integer> list1=new ArrayList<>();

// list1 is used only after the following code is done.
// your oo-worker says that this pipeline has no
// reason/can't to run for some reason in parallel.
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
* Each list accept only 5 elements at most.

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

 `map` is only create a new stream with new content with each cell. so if `map` operation is running on a `Stream` with 0+ elements or an `Optional` with 0-1 elements, we will get the same result.

I would prefer the first one because for after the `findFirst` operation, we are not operating on stream any more so the operation `findFirst` is a delimiter between `Stream` pipeline and `optional` pipeline. My recommendation is to highlight this delimiter by locating it as low as we can because the amount of operation in `Optional` pipeline is small.

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

Both the pipelines calculate different result. The first will print `2` and the second won't print anything.

[Question 3](#question-3)

###### 3. Question with Answer

###### Question

Does the following code compiles? If yes, what does it print? If not, what is the reason and Java implementors didn't allow it?
```java
Stream.of(1, 2, 3)
        .filter((Integer number) -> number > 1)
        .findFirst()
        .sorted()
        .ifPresent((Integer number) -> System.out.println(number));
```

######  Answer

The code doesn't compile because `Optional` class doesn't have `sorted` method. The reason for that is because `Optional` instance represents amount of 0-1 elements so there is no point in sorting them.

[Question 4](#question-4)

###### 4. Question with Answer

###### Question

What methods the `Person` class must implement to make the following code works? (I mean, what `sorted` operation invokes in `Person` class?)

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

The above code does not print anything. Only elements which devide by 2 and 3 will pass the `filter` operation. That means that only elements which devided by 6 will. There any no such elements here so nothing is printed.

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

Incase we are not using `p1` and `p2` any where else in our code, we could write the following:

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
The reason is simiral to the last answer. But here we could not break it down to 2 `filter` operations because it will be equal to `and`.

Incase we are not using `p1` and `p2` any where else in our code, we could write the following:

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

It would print the following __if and only if__ Java would allow to use the same stream (`stream2`) more then once:
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
java.lang.IllegalStateException: stream has already been operated upon or closed...
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

Each node of l1 is a list. Inside the `flatMap` we return every inner list of l1 with a minor change on each element. All in all, all the elements in all the inner lists will be changed in the same way so we can do that change outside of the flat map.
  
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

The code doesn't compile. `mapToInt` receive a lambda which get an `Integer` and return `int`. `Integer` isn't `int` so we can't use `Function.identity()` because the parameter type does not equal to the return type.

[Question 11](#question-11)

###### 11. Question with Answer

###### Question

Can we improve the readability and the safty of the following code?

```java
List<Integer> list1=new ArrayList<>();

// list1 is used only after the following code is done.
// your oo-worker says that this pipeline has no
// reason/can't to run for some reason in parallel.
Stream.of(1,2,3,4,5)
        .filter(x->x>3)
        .peek(x-> callingHeavyFunction(x))
        .sorted()
        .forEach(x-> list1.add(x));
```

######  Answer

Yes. `list1` is used only after filling him so there is no reason to declare `list` before the stream. We should convert the stream to a list using `Collectors.toList()`. Also this code does not support parallelism. Even if `callingHeavyFunction` function does not support parallism now, it can in the future. We should write uniform pipelines.

###### 12. Question with Answer

###### Question

Write a `Map` collector in the following way (implement the `Collector` interface):
* The keys are the elements from the calling stream.
* The values are `List<T>`. `T` is the type of the elements from the calling stream.
* Each list accept only 5 elements at most.

######  Answer

  In the future... :)
  
---
  ### Chapter 10: Conclusion
  

By now you have all the information you need to write complicated pipelines.
Write each pipeline with the following guidelines:

* Readability
* Use as best as you can built-in classes and operations instead of creating new ones.
* Write each pipeline to support future parallelism.
* Understand when a stream will acually run in parallel and what are the consequences. 
* Keep learning and have fun :)
 
------------------

### Legal

© Stav Alfi, 2017. Unauthorized use and/or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear
credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Streams In Depth By Stav Alfi" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/969539b245fd71f18ecd14f48eed2a5d.

----

### Complete terms table

> **Definition 1.1.** _**Pipeline**_ is a couple of methods that will be excuted by a specific order.

> **Definition 1.2.** _**Intermidiate operations**_ will be located everywhere in the stream expect at the end. They return a stream object and does not excute any operation in the pipeline.

> **Definition 1.3.** _**terminal operations**_ will be located only at the end of the stream. They excute the pipeline. They does not return stream object so no other_Intermidiate operations_ or _terminal operations_ can be added after them.

> **Definition 1.4.** Intermidiate operations will be called _**short-circuiting operation**_ if that operation produce a new stream with finite amount of elements from `n` elements where `n` can be infinite number.

> **Definition 1.5.** Terminal operations will be called _**short-circuiting operation**_ if it can finish his excution after finite amount of time.

> **Definition 5.1.** **_Reduce-left_** operation determine the first temporary result will be the first element in the collection from the left and the reduce operation will visit each element in the collection that he didn't visit yet from left to right.

> **Definition 5.2.** **_Reduce-right_** operation determine the first temporary result will be the first element in the collection from the right and the reduce operation will visit each element in the collection that he didn't visit yet from right to left.

> **Definition 8.1.** **_Cuncurrent programming_** is the ability to solve a task using additional synchronization algorithms.

> **Definition 8.2.** **_Parallel programming_** is the ability to solve a task  without using additional synchronization algorithms.

###### Aditional definitions

> **Definition 1.** ___side-effect___  is a concept where a function mutates varaibles which are not defined in the function scope.

> **Definition 2.** A ___pure function___ is a function's return value that is **only** dependent on the input parameters. In addition, Evaluating the result does not mutate the current state of the app or causing any **side-effects**.

> **Definition 3.** ___Lambda Expression___ equals to object of a local class implements an  existing _Functional Interface_. The lambda is the implementation of the single instance method in that interface. _Lambda Expression_  is a function with zero or more parameters and a body that will be (or not) excuted at a later stage. Inside the body we have access to its _free variables_.

---

### Spliterator characteristics table

| Characteristic|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :-------------| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CONCURRENT`  | It's possible to modify (delete/insert/replace elements) the source of the spliterator concurrently without any additional synchronization.
| `NONNULL`     | The source of the spliterator can't insert null elements so each of the elements is garentee to be non-null value. |
| `IMMUTABLE`   | The source of the spliterator prohibit modification (delete/insert/replace elements). |
| `DISTINCT`    | For each 2 different elements `x` and `y` in the source of the spliterator we can tell that `x.equals(y) == false`. |
| `ORDERED`     | If we will print the elements in the source of the spliterator mutiple times by iterating it, we will print the elements in the same order time after time. That means all the elements are stored in the source of the spliterator in a specific order. We can refer to that situation by specify that the source of the spliterator have _encounting order_. For example: `Array`, `List`,`Stuck` has _encounting order_ but `HashMap` doesn't. We can't iterate over `HashMap`. |
| `SORTED`      | The source of the spliterator is ordered by a comperator or in natural order. |
| `SIZED`       | We know the estimated amount of iterations left in the spliterator. |
| `SUBSIZED`  |  If we would split the task to 2 chunks we would know the estimated amount of iterations each spliterator would have. Note: If `SUBSIZED` is on, then `SIZED` also. The reason is if we know how many iterations will be needed after the split for each spliterator, we must know first how many iterations left without spliting in the first place. The claim doesn't follow for the opposite direction as mentioned above. For example - binary trees. |

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

The following table showes which characteristics and flags each _intermediate operation_ may turns on and off: (`SHORT_CIRCUIT` is relevent only in the context of `StreamOpFlag` flags)

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

I will refer to `Spliterator` characteristics as flags inside `StreamOpFlag` enum. That is becouse only the flags defined there are currently used in stream liberary.

| `void forEach(Consumer<? super T> action)`                                                                                                                                                                                                                                                                               |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| The _**forEach**_ method will run _action_ function on each element in the stream. The order of elements which sent to `action` is not defined in parallel execution. Also using this method in sequentrial execution, it is best practice to ussume the order of elements which sent to `action` is not defined as well.|

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
| The _**forEachOrdered**_ method will run _action_ function on each element in the stream and the order which elements from the calling stream are sending to `action` is defined by _encounting order_ if exist. Else the order of elements in the calling stream. Note: If element `n` by the order of the calling stream did not reach yet to the `forEachOrdered` operation, any other element `m > n` which came to `forEachOrdered` will have to wait. |

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
| The _**map**_ method is ment to create from each element in the stream a new element. The type of the new element can be different from the old element. |

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
| _**Peek**_ will run an action on each element of the stream and will return astream containing the same elements. This operation is mainly to support debugging - for example we can print the elements after each operation in the pipeline to see how the stream is calculating the elements. Also it can be used for _side effects_. |

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
| The _**sorted**_ method is an intermidiate operation and it will create new stream which all his elements are sorted according by natural order. `T` must implement _Comparable_ interface so you can override `int Comparable.compareTo(T o)`. The `sorted` method can't send every element it gets to the next operation in the pipeline because the order of the elements is matters and there is a chance that `sorted` will receive element in the future with smaller value so this element should be sent first to the next operation in the pipeline. `sorted` use a temporery collection to store all the elements it gets and he start sorting them from the time it gets excuted. |
Notes:
1. If the calling stream is sorted by any other order, then this method will clear the `SORTED` flag and re-inject it after it will sort the incoming elements by natural order. 
2. When `ORDERED` flag is on, For each 2 elements which are equal by natural order, the first element appears in the encounter order will eneter to the output stream and the second will be filtered out.

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
| The _**limit**_ method is an intermidiate operation that will create new stream that contain at most `n` elements where `n`=`maxSize`. When `n` elements passed him or the last operation can't give more to `limit`, `limit` will end the excution of the **all** the stream. When `ORDERED` flag is on, `limit` will wait for the first `n` elements in the encounter order. It means that incase every element enetered the `limit` operation except the first `n` elements, all those elements will wait there until the first `n` elements in the encounter order will come or `m` elements from the first `n` will be filtered out during the procces until the `limit` operation. In this case the first `m` elements after the first `n` elements in the encounter order will pass the `limit` operation. |

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
// example 2: (filter and limit swaped)

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
| The _**toArray**_ method is a terminal operation that will return a new array of type `A` containing all the elements from the last operation in the pipeline. If `SIZED` flag is on then the size of the generated array is known when `toArray` operation starts. In this case it will initilize an array with the exact amount of cells needed. If `SIZED` is off then `toArray` will guess how many space it needs in the array and it may be a case when he initilize an array with less cells then he need so multiple initilization will be done. Each time it will copy the elements from the last array to the new array.  `generator` receives Integer `n` and must return `A[]`. The array length must be at list equal to `n`. |

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
| Same as `map` operation, this method receive a function `mapper` whos return type is `Integer`. The return stream is a special stream with build-in functions to deal with Integer elements. |

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
| The _**findAny**_ method is a terminal operation that will return one element from the caller stream inside an `Optional` object. This operation may return a diffrent element for each excution without giving any considiration on the encouter order, if exist. Incase the caller stream is empty from any reason, return an empty `Optional<T>` object. This operation has much better performance then `findFirst` because it will return the first element came to it and it does not garentee to return the first element in the calling stream.

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
| In a sequential running, the **_reduce_** operation use the _identity_ value only once as an initial temporary result. The calculating is the same as the first overload we saw. Incase of parallel running, it will work as described above. Incase the calling stream doesn't have any elements, reduce return the _identity_ value as a result. |

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
| In a sequential running, the _**reduce**_ operation use the _identity_ value only once as an initial temporary result. The _identity_ type can be different from the elements in the calling stream. The calculating is the same as the first overload we saw. Incase of parallel running, it will work as described above. Incase the calling stream as no elements, return the _identity_ value as a result. |

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
