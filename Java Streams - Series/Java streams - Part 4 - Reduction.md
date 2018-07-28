# Java streams - Part 4 - Reduction
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://github.com/stavalfi/lectures/tree/master/Java%20Streams%20-%20Series)

### Topics

1. [Introduction](#introduction)
2. [Reduction types](#chapter-1-reduction-types)
2. [Java reduction](#chapter-2-java-reduction)
4. [Conclusion](#conclusion)

### Introduction

`Reduce` in the context of functional programming is an operation to calculate a single value from a given collection. The reduce operation calculate it results from each element in the collection. The way he does that is by saving a temporary result and calculating a new temporary result from the last temporary result and an element he did not visit yet. The function who calculate the next temporary result is called `accumulator`. We must supply it to the reduce operation.

For example:
If we want to sum all the elements in a given array: `{10`,`11`,`12}` , reduce operation determine the first temporary result to be '10' and the accumulator function to be: `(Integer lastResult, Integer element) -> lastResult + element`.
In the first iteration, `10` which is the first temporary result and `11` which is an unvisited element will be sending to our accumulator for calculating the next temporary result: `21`.
In the second and last iteration, `21` and `12` will be sending to our accumulator for calculating the final result: `33`.

Notes:

1. The accumulator operation does not mutate any temporary result or any element in the collection. It creates new temporary result in each iteration.
2. The accumulator operation must be associative; For any three values `x`, `y` and `z`, the evaluation of `accumulator.apply(accumulator.apply(x,y),z)` equals to `accumulator.apply(x,accumulator.apply(y,z))`. Examples for non-associative accumulators: `%`, `/`: `2/(2/2)` isn't equal to (`2/2)/2`. In parallel execution (which we will examine in depth soon), the reducer operation will take advantage of this rule by expanding it recursively to four values.
3. The reduce operation can run in parallel, but more information on parallel will come later.

### Chapter 1: Reduction types

There are 2 types of Reduce operations:

> **Definition 5.1.** **_Reduce-left_** operation determine the first temporary result will be the first element in the collection from the left and the reduce operation will visit each element in the collection that he didn't visit yet from left to right.

The example we have seen above is a reduce-left operation.

> **Definition 5.2.** **_Reduce-right_** operation determine the first temporary result will be the first element in the collection from the right, and the reduce operation will visit each element in the collection that he didn't visit yet from right to left.

### Chapter 2: Java reduction

Java implements _reduce-left_ operation:

|`Optional<T> reduce(BinaryOperator<T> accumulator)`|
|:--|
| The **_reduce_** operation calculates the result from each element in the collection using an accumulator. It is a _reduce-left_ operation, so the first element from the left is the initial temporary result. This method return `Optional<T>` because the calling stream may be empty. |

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

Reduce works in parallel by splitting the work to be done in smaller portions.
Each thread takes a smaller task and calculating a temporary result using the accumulator and determining an initial temporary result for each portion of the collection. After finishing each task, the provided accumulator combine partial results into one. We can see that the reduction can run in parallel only with a given accumulator and without any additional synchronization.

The following image presents parallel computing of the reduce operation when the reduce split the work to 2 threads (can be more).

![](http://i.imgur.com/DkU20og.jpg)

As mentioned earlier, due to the fact that there are more then 3 elements in the collection, the reduce operation take advantage of the associativity because all the following forms of computations are **equal**:

1. `accumulator.apply(accumulator.apply(accumulator.apply(x,y),w),z)`
2. `accumulator.apply(accumulator.apply(x,y),accumulator.apply(w,z))`

_Note:_ The transformation from the first form to the second is immediate (without any intermediate steps) by the definition of associativity and the fact that `accumulator.apply` is associative. 

As we can see, in the second form, we can calculate `accumulator.apply(x,y)` and `accumulator.apply(w,z)` in parallel and combine the results later using `accumulator.apply`.

Also, As a challenge, I leave to the reader the task to activate the associativity rule on array of `5` elements so the reduce operation will support parallel invocations of `accumulator.apply` as much as possible.

----
The next overloading of the `reduce` operation in Java also receive an _identity_.

Instead of letting reduce operation determine the first left value to be the initial temporary result, we can declare it, and this value is called the _identity_ value.

This type of reduction is called `folding`. Java uses fold-left reduction by default.

| `T reduce(T identity, BinaryOperator<T> accumulator)` |
| :--- |
| In a synchronized execution, the **_reduce_** operation use the _identity_ value only once as an initial temporary result. The calculating is the same as the first overload we saw. In case of parallel running, it will work as described above. In case the calling stream doesn't have any elements, the `reduce` method returns the _identity_ value as a result. |

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

The third and last overloading of `reduce` operation in Java also receives a _combiner_.

After each thread finished his task, instead of using the provided accumulator function to combine the partial results, we can provide a _combiner_ function for that. The logic of the combiner can be different from the accumulator.
In case the run is sequential, there will be no use in the provided combiner function.

| `<U> U reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)` |
| :--- |
| In a synchronized execution running, the _**reduce**_ operation use the _identity_ value only once as an initial temporary result. The _identity_ type can be different from the elements in the calling stream. The calculating is the same as the first overload we saw. In case of parallel running, it will work as described above. In case the calling stream as no elements, return the _identity_ value as a result. |

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

### Conclusion

The reduce operation does not mutate it's state and create new one state in each iteration. In the last example, we saw that the reduce copy multiple strings multiple times. The complexity of that is too high. In the next chapter, we will see an alternative approach to this problem, but it will cost us something else.

All in all, we have seen that for each reduction we must provide an accumulator function and an optional identity or combiner function. Both the accumulator and the combiner must be stateless and pure functions to support parallel execution.