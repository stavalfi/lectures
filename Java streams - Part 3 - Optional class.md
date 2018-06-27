# Java streams - Part 3 - Optional class
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://gist.github.com/stavalfi/969539b245fd71f18ecd14f48eed2a5d)

### Topics

1. [Introduction](#introduction)
2. [Operations](#chapter-1-operations)
3. [Conclusion](#conclusion)
4. [Legal](#legal)

### Introduction


Some terminal operations return an object that is made or equal from/to one or more elements in the stream. What happens when the source collection is empty, or none of the elements in the source collection survived all the pipeline until coming to the terminal operation?

Returning `null` is an option but a bad one. The following reasons may cause `NullPointerExcaption`:

1. We forgot to initialize an object.
2. We received a `null` pointer from a function and we did not check it.

Let's focus on the second reason - Do we know if the null we received is a bug of the programmer who wrote the function or the function should return us `null` for that call?

To avoid some of the reasons for `NullPointerExcaption`, Java 8 introduced us the `Optional<T>` type. It comes with intermediate and terminal operations to deal with the absence/ presence of a value. 

Note: While the _intermediate operations_ of streams run when there is a call to a terminal operation, the optional's _intermediate operations_ executed immediately.

### Chapter 1: Operations

We will examine the basic operations of `Optional<T>` and then see some stream's terminal operations that return an `Optional<T>`:


| `public boolean isPresent()` |
| :--|
| The _**isPresent**_ terminal operation returns true if it represents an existing value, else returns false. |

| `public void ifPresent(Consumer<? super T> consumer)` |
| :--|
| If the object who call this method represents an existing value, the _**isPresent**_ terminal operation will invoke `consumer` function on the existing value. Else do nothing. |

| `public static <T> Optional<T> ofNullable(T value)' |
| :--|
| The _**ofNullable**_ function will return an optional representing a value. Use it **only** when you aren't sure if `value` is equal to `null`. If `value` is equal to `null`, this function will return an empty `optional` object (invoking `isPresent` terminal operation on an empty optional object return `false` and an empty `optional` object is not guaranteed to be a singleton.) |

| `public static <T> Optional<T> of(T value)' |
| :--|
| The _**of**_ function will return an optional representing a non-null value. Use it **only** when you are sure that `value` is not equal to `null`. A `NullPointerExcaption` will be thrown in case `value` is equal to `null`. |

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
| If the object who call this method represents an existing value, the _**map**_ intermediate operation will invoke `mapper` on the existing value. `map` return type is an `Optional<U>`. When the value is absent, return an empty `Optional<U>`. `U` is the return type of `mapper`. |

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

After examining the basic functions of `Optional<T>`let's go back to streams and look at a terminal operation who return `Optional<T>` object:

| `Optional<T> findAny()` |
| :--|
| The _**findAny**_ method is a terminal operation that will return one element from the caller stream. It may return a different element for each execution. Later we will see why this behavior is better for performance. In case the caller stream is empty for any reason, return an empty `Optional<T>` object. |

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

We will cover much more powerful terminal operation which returns `Optional<T>` object in the next chapter.

---

### Conclusion

The `Optional<T>` type help us to replace the old `if(value==null)` statement by building a pipeline to deal with the absence/presence of a given value. There are more operations we didn't cover. Go to [Optional class methods](#optional-class-methods) chapter to read more about `Optional<T>` operations.

---

### Legal

© Stav Alfi, 2018. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Java streams - Part 3 - Optional class" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/0e80e3df73e7c2ab3c3310ddef573bba.