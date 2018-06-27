# Java streams - Part 2 - Primitive streams
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://gist.github.com/stavalfi/969539b245fd71f18ecd14f48eed2a5d)

### Topics

1. [Introduction](#introduction)
2. [Types and examples](#chapter-1-types-and-examples)
3. [Legal](#legal)

### Introduction

The interfaces `IntStream`, `DoubleStream`, `LongStream` implement `BaseStream` interface. In addition to `Stream` interface methods, They expose additional methods to query their elements by knowing their type. As you can guess, now we **nicely** calculate the sum, average, min, max of the elements in the stream.

In this chapter we will examine some of the additional methods from `IntStream` interface is offering compared to `Stream` interface.

### Chapter 1: Types and examples

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

### Legal

© Stav Alfi, 2018. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Java streams - Part 2 - Primitive streams" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/98c170f8f13c302c76f8bf92cda62557.
