# Java streams - Part 6 - Spliterator
#### [Stav Alfi](https://github.com/stavalfi) | [Lectures](https://github.com/stavalfi/lectures) | [Java streams tutorial series](https://gist.github.com/stavalfi/969539b245fd71f18ecd14f48eed2a5d)

### Topics

1. [Introduction](#introduction)
2. [The spliterator interface](chapter-1-the-spliterator-interface)
3. [Spliterator methods](chapter-2-the-spliterator-methods)
4. [Spliterator in Java streams](chapter-3-spliterator-in-java-streams)
6. [Legal](#legal)

### Introduction

In the following paragraphs, let's stop thinking about streams and return the good old collections and how to iterate them. You probably heard or used `Iterators`. they are great but not suitable for all the problems we are facing today. One of those is parallel calculating.

As we saw earlier, we can split the calculation on an array to calculate smaller chunks of the same array in parallel and after that, combining the temporary results to one final result. Splitting a collection can be done in different types of collections and not only arrays. Which types of a collection can we split? The ones we know their estimated `SIZE`. After splitting a binary tree, does the sub-trees are also splittable; Do we know their estimated `SIZE`? The answer is yes only for balanced trees. If we do know their `SIZE` after the splitting, we can say that the `SIZE` of the collection is known and also the sub-trees; The collection is also `SUBSIZED`.


### Chapter 1: The spliterator interface

A `Spliterator` is a new interface in Java 8. Each collection in Java implements and return a spliterator same as returning an iterator. Both can be used to iterate over a given collection, but spliterator can also be used to split the work to smaller tasks for parallel computing. A spliterator can also represent a generator function or an IO channel. If we choose to split the collection which the spliterator is iterating over and succeed(we soon examine the reasons for failed), the spliterator will now be responsible to only half+-(we will explain later why not precisely half) the elements he should have iterate before the splitting and the new spliterator that will be created after the splitting will iterate on the rest of the elements; The other half. We can keep splitting the spliterator work recursively until there is nothing to split or we don't know the estimated `SIZE` of the spliterator (those are the reasons for failer of the splitting operation).

`SIZE` and `SUBSIZED` are only 2 characteristics a spliterator can have. We have 5 more:

| Characteristic|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :-------------| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CONCURRENT`  | It's possible to modify (delete/insert/replace elements) the source of the spliterator concurrently without any additional synchronization.
| `NONNULL`     | The source of the spliterator can't insert null elements, so each of the elements is guaranteed to be a non-null value. |
| `IMMUTABLE`   | The source of the spliterator prohibit modification (delete/insert/replace elements). |
| `DISTINCT`    | For each 2 different elements `x` and `y` in the source of the spliterator we can tell that `x.equals(y) == false`. |
| `ORDERED`     | If we print the elements in the source of the spliterator multiple times by iterating it, we will print the elements in the same order time after time. That means all the elements are stored in the source of the spliterator in a specific order. We can refer to that situation by specifying that the source of the spliterator has _encounting order_. For example: `Array`, `List`,`Stuck` has _encounting order_ but `HashMap` doesn't. We can't iterate over `HashMap`. |
| `SORTED`      | The source of the spliterator is ordered by a comparator or in the _natural order_. |
| `SIZED`       | We know the estimated amount of iterations left in the spliterator. |
| `SUBSIZED`  |  If we split the task into 2 chunks, we would know the estimated amount of iterations each spliterator would have. Note: If `SUBSIZED` is on, then `SIZED` also. The reason is if we know how many iterations will be needed after the split for each spliterator, we must know first how many iterations left without splitting in the first place. The claim doesn't follow for the opposite direction as mentioned above. For example - binary trees. |

### Chapter 2: Spliterator methods

Let's examine the basic abstract methods of `Spliterator` interface:

| `boolean tryAdvance(Consumer<? super T> action)`                                                                                                                                                                                                                                                                                                                  |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| If the spliterator did not iterate over all the elements yet, return `true` and perform `action` on the next element in the source of the spliterator. Then advance the spliterator to point to the next element. If there are no elements to iterate over, return false and don't invoke `action` because there is no element to send to the action anyway. If we used `Iterator` instead of `Spliterator`, then we must have called two methods instead of one: `hasNext` and `next`. Now the interface is cleaner and less imperative. __Notes:__ __1.__ If `ORDERED` characteristic is off, then there is no guarantee which element will be visited by the spliterator using this method from the remaining elements. __2.__ After the split, each spliterator on the source collection will iterate over fewer elements now than using a single iterator. That means this method is checking how many elements this spliterator have left before finish his sub-task. 

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
| If the `SIZED` characteristic is on and we did not iterate the source yet, return the exact number of elements left to iterate over. Else try to compute an estimated number of iterations left. If it is too expensive to compute or unknown or infinite, then return `Integer.MAX_VALUE`.|

| `Spliterator<T> trySplit()`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Even if the `SUBSIZED` or `SIZED` characteristics is off, this method will try to split the remaining iterations to 2 different spliterators. If it failed to do so or the spiterator that this method is invoked by was iterating before calling this method or data structure constraints/ efficiency considerations, the method will return null. __Note:__ An ideal splitting will divide the elements exactly in half for parallel computing. For example, in a balanced binary tree, there should be no problem to do so. |

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

### Chapter 3: Spliterator in Java streams

Streams iterate over the source collection using spliterators. When iterating over a stream in parallel, we will use `trySplit` to split the work, but the method may return null due to the reasons we mentioned earlier and the rest (or all) the calculation will be performed sequentially.

When we transform the source collection to a stream, the stream uses the spliterator, returned from that collection, to iterate over the elements and each _intermediate operation_ will return a new stream that uses the same spliterator.

Stream uses an internal enum to manage the relevant characteristics for him, `StreamOpFlag`. Each pipeline has its flags. The flags at the beginning are initialized to be the relevant characteristics from the spliterator. A _intermediate operation_ and _terminal operation_ can modify the spliterator characteristics or the _flags_ of the whole pipeline or examine the spliterator's characteristics for choosing the best algorithm.
For example, the `filter`  _intermediate operation_ can reduce the number of elements which sent to him. The stream library can not guess how many elements left so if the `SIZED` flag and characteristic is on, now they are off.

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
1. `SHORT_CIRCUIT` means an operation is considered to be _short circuit operation_ (Explained in chapter 1).
2. For now, we can see that the Java stream library doesn't use `CONCURRENT`,`NONNULL`, `IMMUTABLE`. That doesn't mean they won't use them in the future. If `SIZED` is off, then `SUBSIZED` will be too. For now, there is no operation which clears or injects the `SUBSIZED` characteristic.
3. The existence of `SHORT_CIRCUIT` flag implies that the above flags are not from the `Spliterator` Interface characteristics but from `StreamOpFlag` enum that uses the relevant flags he needs for streams optimizations.

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

### Legal

© Stav Alfi, 2018. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Java streams - Part 6 - Spliterator" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/5f357213c3275fe923727a6221eafecd.