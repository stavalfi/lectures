# Java streams - Part 5 - Collectors
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://gist.github.com/stavalfi/969539b245fd71f18ecd14f48eed2a5d)

### Topics

1. [Introduction](#introduction)
2. [Reduce vs Collector](#chapter-1-reduce-vs-collector)
3. [toList() collector](#chapter-2-tolist-collector)
4. [Simple toList() collector implementation](#chapter-3-simple-tolist-collector-implementation)
5. [Collectors in the standard library](#chapter-4-collectors-in-the-standard-library)
4. [Conclusion](#conclusion)
5. [Legal](#legal)

### Introduction

The reduce operation we have seen above is powerful but come with a price - runtime complexity. Sometimes we prefer to mutate a state to receive a better complexity.

Java 8 created for us classes which implement the `Collector` interface. The concept of a collector and a reduce operation is similar - both operations calculate a value from multiple values.

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
The finisher is a function that can change the final result of the collector. The last method is the `characteristics` who return a `Set` of flags. Each flag indicates a specific behavior the collector support. All the flags are used by the `Stream.collect` 'terminal operation who run the collector, to gain better performance. We will discuss flags later.

### Chapter 1: Reduce vs Collector

The main differences between the `reduce` operation and a collector are:

1. Reduce operation forbid mutating any variable inside the accumulator function. Instead of mutating the temporary result, it creates and returns a new temporary result. On the other hand, collectors do not behave the same. We will see that instead of returning a new temporary result in each iteration, it will mutate it each time in the accumulator function so this accumulator does not return anything. Nevertheless, the collector still supports parallel execution.

2. Instead of supply an `identity` value to the reduce, we can provide a lambda (`supplier`) to the collector, so the logic of the `identity` value creation stays inside the collector scope only.

3. The reduce operation doesn't support changing its final result inside its scope, but the collector supports this kind of behavior.

4. Pre-defined flags can tell the stream operation which runs the collector how to execute. The following flags can be used:

| `Collector.Characteristics enum`|
| :--- |
| `CONCURRENT` - We can access the resulting container (collection or an object) from the accumulator function concurrently. In this case, the supplier should be called only once. |
| `IDENTITY_FINISH` - We doesn't change the resulting container, and the finisher function is: `(A container) -> container` |
| `UNORDERED` - The resulting container doesn't keep his elements in any order. Another way to think about that is, you can't use `for` loop to print the container elements. For example `HashMap`. This flag is meant to tell the operation `Stream.collect` that we do not care in what order the elements will be inserted into the final container. For example, if the flag `CONCURRENT` is off, that means we must split the work into smaller temporary containers. Meaning each thread will do a smaller task and will insert the elements in the different container. Because the container doesn't care about what order we insert elements, we don't need to wait for thread `n` to copy his result to a bigger container before thread `n-1` did that. We let any thread who finish his task to copy his result to a bigger container.|

### Chapter 2: toList() collector

Java 8 wrote for us some collectors we can use for free inside the `Collectors` class. We can use them by calling `Stream.collect` that accept a `Collector`.
The collect terminal operation will use the accumulator, supplier, combined, finisher and the characteristics function from the collector object to run.


| `public static <T> Collector<T,?,List<T>> toList()` |
| :--- |
| The __*toList*__ collector converts the calling stream to a `List<T>`. There is no guarantee to which class will be used to create the list. Also no guarantee on the thread-safety or the serializability of that class. |

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

### Chapter 3: Simple toList() collector implementation

Lets have a look at a possible implementation of the `toList` collector:

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

### Chapter 4: Collectors in the standard library

More collectors Java 8 provide us inside the `Collectors` class are:

| `public static <T> Collector<T,?,Map<Boolean,List<T>>> partitioningBy(Predicate<? super T> predicate)` |
| :--- |
| This collector partition the incoming elements into 2 groups: The elements which `predicate` return true for them and the second group is the rest. The result container implements `Map`.|

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
| This collector creates a `Map`. Each element is sent to the `classifier` function that generates a key. The value for each key is a collection of all the elements which produce the same key. The collection stored in each value implements `List<T>`. If you want to choose which collection to use, choose a different overload. API Note: In parallel running, each accumulator adds elements to different `Map` object and the combiner is merging them. You should use a different function to have better performance: `groupingByConcurrent` |

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
Some collectors who return `Map<T,?>` Accept collector as an argument. That is for transforming the elements inside each value to something else. They can transform the elements inside the value into an Integer that represent the amount of all the element who are inside the value or doing something else with those elements. Let's have a look at the overload and some examples:

| `public static <T,K,A,D> Collector<T,?,Map<K,D>> groupingBy(Function<? super T,? extends K> classifier, Collector<? super T,A,D> downstream)` |
| :--- |
|This overload has the same behavior as the last overload including some API notes. The difference here is that each element that produces the same key will be sent to the `downstream` collector. Each key will create different `downstream` collector. It means that if we have `n` keys and each key have `m` elements that produce that key, there will be `n` different `downstream` collectors for each `m` elements.|

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

The possibility of running the second collector inside a collector is very powerful.

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
---
### Conclusion

You will notice that there is a `reducing` collector. This operation can have `identity` value, `accumulator` and a `combiner` (Same as `Stream.reduce`). Why Java 8 created 2 reducers who operate the same? The answer is to let us use reduction inside _multi-level_ collectors. For example, we can use `reducing` collector inside a `groupingBy` collector. The research about _multi-level_ collectors is left to the reader.

---

### Legal

© Stav Alfi, 2018. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Java streams - Part 5 - Collectors" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/3ba04d7d114132d6f57413052bd8daa2.