# Java streams - Additional - Quiz
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://github.com/stavalfi/lectures/tree/master/Java%20Streams%20-%20Series)

  The Answers are at the end of the quiz.
  
  ----

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