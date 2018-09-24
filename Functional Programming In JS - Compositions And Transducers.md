# Functional Programming In JS - Compositions And Transducers By Stav Alfi

### Topics

1. [Introduction](#introduction)
2. [Processing a single element](#processing-a-single-element)
3. [Processing a collection](#processing-a-collection)
4. [Transducers](#transducers)
5. [Conclusion](#conclusion)

### Additional

1. [Sources](#sources)
2. [Quiz](#quiz)
3. [Complete terms table](#complete-terms-table) (Wikipedia)

---
_General note:_ This tutorial is heavily influenced by the book [Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS) by [Kyle Simpson](https://github.com/getify). I _highly_ recommend you to read his excellent book.

------

### Introduction

__Prerequirements__

* `ES6/ES2015` Syntax: Arrow functions and spread operator.
* Closures.
* reduce, reduceRight, map, filter functions.
* Basic terminology of functional programming: Side effects and pure functions by Wikipedia definitions.
* Imperative vs. declarative code.

----

Throughout this tutorial, we will cover a series of basic technics for manipulating a collection of functions. We will build a utility that processes a single element using composition of functions. Then we will process a collection using multiple implementations such that each of them can do more than the last.

<img src="https://i.imgur.com/xBVMq9d.png" width="150" height="98"/>

Functional programming is all about functions: the logic. Using special technics, we will remove most of the unnecessary information from the implementation to emphasize our actual logic. The fundamental of programming is to divide our logic to multiple procedures such that each area will do something different. More procedures, the more comfortable for us to test and understand the overall picture. Functional programming is written using declarative code so the implementation details, which are harder to understand, will be less visible. 

To avoid situations where a function calls directly other function, we need support from our programming language or using a third function for defining a relationship between those two functions. Why is that important? We would like to be able to invoke more than one unique function after a specific function. Otherwise, we will have to implement the same function multiple times such that each implementation will call a different function. One technique for doing so is called _function composition_ by sending the result of the first function to the second function as a parameter (a utility does the functions invocations). Javascript doesn't have (yet) the syntax for composing functions, so we will have to build or use a utility for using function compositions.

> **Definition 1.1.** In mathematics, ___function composition___ is the pointwise application of one function to the result of another to produce a third function. For instance, the functions f : X → Y and g : Y → Z can be composed to yield a function which maps x in X to g(f(x)) in Z. The resulting composite function is denoted g ∘ f : X → Z, defined by (g ∘ f )(x) = g(f(x)) for all x in X.

The utilities we are building during this tutorial will call `f` and `g` in a pre-defined order (determined by the user). The positive outcome will keep any function like `f` and `g` free from directly calling other functions.

If you notice, I also used 2 different ways to describe code: _procedure_ and _function_. Let's define them also:

> **Definition 1.2.** The ___subroutine/procedure___ may accept input parameters and may return a computed value to its caller (its return value).

> **Definition 1.3.** A ___function___ is a procedure and a process that associates to each element of a set X a unique element of a set Y. Functions consider being _pure fucntions_.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Function_machine2.svg/1200px-Function_machine2.svg.png" width="200" height="200">

The only difference between them is that a procedure sometimes does not accept input parameters or a return value, so a composition between procedures which are not functions is impossible. Also, procedures which do not accept parameters or doesn't return anything won't be a pure function. Functional programming is all about deterministic functions.

To conclude, we will talk about pure functions which accept a single argument. Any other functions won't fit the definition of composition.

> **Definition 1.4.** A ___unary function___ is a function that takes one argument.

----

### Processing a single element

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Function_machine5.svg/2000px-Function_machine5.svg.png" width="300" height="400">

At the beginning of our journey we will implement a utility to process a single element by sending the element to a function `f1` and then the result will be sent to function `f2` and so on...

```javascript
const result = compose(...,f3,f2,f1)(element);
```
Our job is to implement `compose` function. It will send `element` to `f1` and the result will be sent to `f2` and so on until the last function is invoked its result will be the `result`.

Why I split the params into two groups: functions and the input element such that each group is given to us in a different function? Then we can use the same function multiple times and not worry about supplying all the parameters at the same moments (sometimes we can't because we currently don't know all of them):

```javascript
const composition = compose(...,f3,f2,f1)
const result1 = composition(element1);
....
const result2 = composition(element2);
```

This splitting technique is called curring.

> **Definition 2.1.** In mathematics and computer science, ___currying___ is the technique of translating the evaluation of a function that takes multiple arguments into evaluating a sequence of functions, each with a single argument.

Thanks to `ES6` spread syntax, we can cheat a little bit by accepting multiple functions as a single array.

Let's implement compose function:

```javascript
const compose = (...funcs) =>
    initialElement =>
        funcs.reduceRight((result, func) => func(result), initialElement);
```

As I implicitly specified it before, there is a repeated pattern here; we use function and the last result to produce a new result: `(result, func) => func(result)`. We don't need any additional values to produce the final result except the first element. In situations like that, `reduce` is the first thing that should come up in your mind. But why `reduceRight`? Remember that the goal is to implement something that is similar to mathematical composition and not something like _pipe_ in Unix. The first function we invoke is the last the user provided to `compose`.

Usage examples:

```javascript
const multiply = number => number * 10;
const toString = number => number + "!";

const composition = compose(toString,multiply);

composition(1); // 1 -> 10 -> 10!
composition(10); // 10 -> 100 -> 100!
```
----

### Processing a collection

###### V1

`compose` return a unary function which accepts a parameter and transform it. Why not using it inside `map` which accept a function with the same signature!

```javascript
[1,2,3].map(compose(toString,multiply));

// output:
// ["10!", "20!", "30!", "40!"]
```

That's nice, but we want to use other operators except `map`.

###### V2

Our next generation of composition will accept functions such that each of them will process a collection instead of a single element. We will create something simiral to:

```javascript
const add = (number1, number2) => number1 + number2;
const multiply = number => number * 10;

compose(
    operator(Array.prototype.reduce, add),
    operator(Array.prototype.map, multiply)
)([1, 2, 3]);

// output: 
// [1,2,3] -> [10,20,30] -> 60
```

The following implementation will give us the advantage that we can use more operators except for what `Array.prototype` provide us but with one condition: Every operator we choose to use must use `this` to access the actual array.

As I just mentions, to implement this version, we need to manually bind `this`:

```javascript
1. this of Array.prototype.map === [1,2,3]
2. this of Array.prototype.reduce === [10,20,30]
```

First let's implement `opertor` function:

```javascript
const operator = (func, ...params) =>
            function () {
                return func.apply(this, params);
            };
```

We can't use arrow function inside the return of the `operator` function because we need to change `this` at the function call site while the arrow function will make it's `this` to be the `this` of the `operator` function which is the global `this` and not the actual array we will pass later as a parameter.

Now we need to implement `compose`. It will be almost identical to its last version because, again, compose accept unary functions. The only difference is that now those unary functions's `this` must be the collection they process:

```javascript
const compose = (...operators) =>
            initialThis =>
                operators.reduceRight(
                    (currentThis, operator) => operator.apply(currentThis),
                    initialThis
                );
```

If you look closely at the `operator` implementation, you will see that we sent to `compose` a closure and not an actual `operator` function. The closure already knows what to do thanks to the parameters we sent to the `operator` function. The only thing the closure need is someone who will change it's `this` - the `compose` function.

The `compose` function will send the result of the first closure to the second closure (of the second operator) as it's `this` and so on...

----
### Transducers

We have a performance issue in the last two versions. Do you see it? 

Like Javascript Array.prototype methods, each one of our operators will iterate over the whole array, and only then the next operator will be called. That means if we added another operator (which will not be covered by this tutorial) `take` that determines how many elements at most will be passed to the next operator:

```javascript
const add = (number1, number2) => number1 + number2;
const multiply = number => number * 10;

compose(
    reduce(add),
    take(2),
    map(multiply),
    map(multiply)
)([1, 2, 3, 4]);

//  note: I used the solution of the second question in the quiz to be 
//        able to write the above code. The operator and compose 
//        implementations didn't change.
```

The flow as we implemented `compose` and `operator` until now: 
`[1,2,3,4] -> [10,20,30,40] -> [100,200,300,400] -> 300`.

Instead of what we actually wanted for increasing preformance: `[1,2,3,4] -> [10,20] -> [100,200] -> 30`.

Our current implementation considered to be eager processing while the implementation needed to have the better performance we call lazy processing. Consider the following visualizations:

Eager - iterate over the source collection multiple times:
![](https://cdn-images-1.medium.com/max/1600/1*mJicJiOZT4M9jwv6kMkwRg.gif)

Lazy - Iterate over the source collection only once at most:
![](https://cdn-images-1.medium.com/max/1600/1*rEOyWd0MTPv_NZvzDaFbkA.gif)

> Gifs source: The gifs are taken from the tutorial: [Understanding Transducers in JavaScript](https://medium.com/@roman01la/understanding-transducers-in-javascript-3500d3bd9624) by [Roman Liutikov](https://medium.com/@roman01la) which is also _highly_ recommended to understand this subject.

In this and the last chapter, we will implement a lazy implementation of `compose` and `operator` functions and also support `filter`! In the end, I will introduce you a library which took this concept one step further and wrote a protocol which allows multiple functional libraries' operators works together and let us the user use all of them concurrently.

Transducers are the `operators` without the performance problem we mentioned earlier. We will need to implement them again in a smarter way, and it will force us to redesign and rewrite our `compose` function in such a way it will support our transducers.

Let's begin.

The operators `map` and `filter` doesn't behave the same. The former transforms each element without changing the source array size while the latter may decrease the size of the output array.

The plan is to somehow loop through the source array such that all the transducers will process each element. If all the transducers `filters` kept that element, the last transducers will add that element to the output array.
Else, one of the transducers `filters` kicked this element for some reason, and our implementation won't add him to the result array.

The idea is very similar to our first implementation of compose:

```javascript
[1,2,3].map(compose(toString,multiply));

// output:
// ["10!", "20!", "30!", "40!"]
```

The only problem above is the implementation; we can't do much except elements transformations.

Let's continue.

As I mentioned before, all the elements will pass through sometimes all our transducers. If a transducer approves this element, then it will send the element (or the element's transformation) to the next transducers, and if not, the transducer won't call the next transducers. That means, if all the transducers approved the element, then we need to add him to the output array. If one didn't approve the element, then that element won't be in the result array.

Did you figure out which operator will replace the `map` operator who calls the `compose` for every element? I will give you another clue; We sometimes add an element to the result array and sometimes not:

```javascript
const question = "did this element passed all the transducers?";
(resultArray, element) => question(element) ?
    [...resultArray, "transformed element"] :
    resultArray;
``` 
In this kind of thinking, we could give the output array of iteration `i` (when processing element `i`) to the calculation of iteration `i+1`. Each iteration will produce a new array, and the last iteration will be our final result array.

The best candidate for such a behavior is the `reduce` function:

```javascript
[1,2,3].reduce(compose(transducer1,transducer2,transducer3),[]);
```

The output function of `compose` must accept as a first argument the array _until now_ and as a second argument the _current_ element we are processing. If a transducer approves the element, he will pass those two params to the next transducer, and if not, he won't call the next transducer1 and return the array _until now_ same as he got it. If the last transducer approved the element, then he will return a new array with the current element in it; otherwise, he will return the array he received as a parameter.

It means that each transducer signature equals to the output function of `compose`.

Because we want to let the user write the actual logic, we need to hide the implementation details from him. We should make the following code works:

```javascript
[1, 2, 3, 4].reduce(compose(filter(isEven), map(increase)), []);
```

Our first attempt to implement filter and map transducers:

```javascript
const map = transformer =>
    (arrayUntilNow, element) => [...arrayUntilNow, transformer(element)];
const filter = predicate =>
    (arrayUntilNow, element) => predicate(element) ?
        [...arrayUntilNow, element] :
        arrayUntilNow;
```

Since only the last transducer can (if it approve) add the element to the array, then this attempt fails.

To understand the solution to this little problem, we need to remember:

1. Each transducer sends the array and the element to the next transducer.
2. The reduce function sends the initial (empty) array and the first element.
3. This is the trick -> we can create an "adding element to an array function" which accepts an array and an element. 

Each transducer will call a function which accepts an array and an element. All the transducers except the last will call the next transducer, and the last transducer will call the add "adding an element to an array function".

(The add function trick is so simple and brilliant).

Second attemp to implement filter and map transducers:

```javascript
const map = mapper =>
    (nextTreducing, arrayUntilNow, currentElement) =>
        nextTreducing(arrayUntilNow, mapper(currentElement));

const filter = predicate =>
    (nextTreducing, arrayUntilNow, currentElement) =>
        predicate(currentElement) ?
            nextTreducing(arrayUntilNow, currentElement) :
            arrayUntilNow;
```

_Remmember:_ `nextTreducing` will be the actual next transducer for each of the transducers except the last one which will be the "adding element to an array function".

This attempt has a minor problem of function signature:  if `nextTreducing` is the next transducer, it should accept two parameters and not three. We can fix it easily by:

```javascript
const map = mapper =>
    nextTreducing =>
        (arrayUntilNow, currentElement) =>
            nextTreducing(arrayUntilNow, mapper(currentElement));

const filter = predicate =>
    nextTreducing =>
        (arrayUntilNow, currentElement) =>
            predicate(currentElement) ?
                nextTreducing(arrayUntilNow, currentElement) :
                arrayUntilNow;
```

But how do we use those transducers?

How to use`map` transducer directly:
```javascript
const map = mapper =>
    nextTreducing =>
        (arrayUntilNow, currentElement) =>
            nextTreducing(arrayUntilNow, mapper(currentElement));

// the user will provide them:
const multiply = number => number * 10;
const add = number => number + 1;

// the user won't provide this because it's implementation detail:
const addElementToArray = (arrayUntilNow, currentElement) => [...arrayUntilNow, currentElement];

const result1 = map(number => number * 10)(addElementToArray)(
    // the last transducer will give this parameters to this transducer:
    [], 10
);
// ([],10) -> [100]
const result2 = map(number => number * 10)(addElementToArray)([1], 8);
// ([1],8) -> [1,80]
const result3 = map(number => number * 10)(map(number => number + 1)(addElementToArray))([2], 5);
// ([2],5) -> ([2],50) -> [2,50]
```
How to use `filter` transducer directly:

```javascript
const filter = predicate =>
    nextTreducing =>
        (arrayUntilNow, currentElement) =>
            predicate(currentElement) ?
                nextTreducing(arrayUntilNow, currentElement) :
                arrayUntilNow;

// the user will provide them:
const multiply = number => number * 10;
const isEven = number => number % 2 === 0;

// the user won't provide this because it's implementation detail:
const addElementToArray = (arrayUntilNow, currentElement) => [...arrayUntilNow, currentElement];

const result1 = filter(isEven)(addElementToArray)(
    // the last transducer will give this parameters to this transducer:
    [], 10
);
// ([],10) -> [10]
const result2 = filter(isEven)(addElementToArray)([1], 7);
// ([1],7) -> [1]
const result3 = filter(isEven)(map(multiply)(addElementToArray))([2], 6);
// ([2],6) -> ([2],6) -> ([2],60) -> [2,60]
```

Take the time to understand the third example in each of the above.

Until now we entirely built our transducers and understood we need to use `reduce` to make it all work. The only task left for us is building the `compose` function. 

The `compose` accepts multiple transducers which each of them accepts __nextTreducing__. The compose needs to chain the second from last transducer to the last transducers and so on except the first transducer which should get the "adding element to an array function".

The initial `compose` does it prefectly well:

```javascript
const compose = (...operators) =>
    initialThis =>
        operators.reduceRight(
            (currentThis, operator) => operator.apply(currentThis),
            initialThis
        );
```

The only problem in the above implementation is the order of the functions he process. To understand the problem, let's see the final result:

```javascript
const isEven = number => number % 2 === 0;
const multiply = number => number * 10;

[1, 2].reduce(
            compose(
                filter(isEven),
                map(multiply)
            )(expandArray),
            []
        );

// expected output:
// reduce send ([],1) ->
// map transform and send ([],10) ->
// filter approve and send ([],10) ->
// expandArray add it to the array and reuturn [10]
// reduce send ([10],2) ->
// map transform and send ([10],20) ->
// filter ignore and return [10] ->
// reduce return [10]

// actual output:
// reduce send ([],1) ->
// filter ignore
// reduce send ([].2) ->
// filter approve and send ([],2) ->
// map transform and send ([],20) ->
// expandArray add it to the array and reuturn [20] ->
// reduce return [20]
```

Take the time to understand what was the problem before reading the following explanation. 


`compose` will send `expandArray` to `map(multiply)` as a parameter and we will receive a function back (which is the next transducer) which `compose` will send to `filter(isEven)` as a parameter and we will get a function back which the `reduce` will use directly in each iteration such that the `reduce` will send him the array and the current element: `element -> filter -> map`. 

Take the time to understand what was the problem.

To reverse the order, we will use `pipe` which can be implemented in two different ways:

```javascript
// v1:
const pipe = (...funcs) =>
            initialParam =>
                funcs.reduce((currentParam, func) => func(currentParam), initialParam);
// v2:                
const pipe = (...funcs) => compose(...(funcs.reverse()));
```

The final version:

```javascript
const isEven = number => number % 2 === 0;
const multiply = number => number * 10;

[1, 2, 3, 4].reduce(
            pipe(
                filter(isEven),
                map(multiply)
            )(expandArray),
            []
        );
        
// actual output: (copy-paste from expected output from the last example)
// reduce send ([],1) ->
// map transform and send ([],10) ->
// filter approve and send ([],10) ->
// expandArray add it to the array and reuturn [10]
// reduce send ([10],2) ->
// map transform and send ([10],20) ->
// filter ignore and return [10] ->
// reduce return [10]
```

Let's make it more readable:

```javascript
const transduce = (...funcs) =>
            addElementToArray =>
                elements =>
                    elements.reduce(pipe(...funcs)(addElementToArray), []);
```

Usage of the final product:

```javascript
const addElementToArray = (arrayUntilNow, currentElement) => [...arrayUntilNow, currentElement];

const result = transduce(
            filter(isEven),
            map(increase)
        )
        (addElementToArray)
        ([1, 2]);
```

Other solution to all this tutorial:

```javascript
const source = [1, 2]
const result = [];

for (let i = 0; i < source.length; i++)
    if (source[i] * 10 % 2 === 0)
        result.push(source[i] * 10);
        
console.log(result);
```

Bad joke ;)

----
### Conclusion

There is a project on GitHub: [transducers-js](https://github.com/cognitect-labs/transducers-js). Its last commit was around 2016, but the critical fact is that they wrote a protocol about transducers in js. The core code is the implementation of our transducers. They added support for cancellation of a process so operators like `some`, `indexOf`, `take`, `limit` and so on can be implemented by the same API. It means that every functional or reactive library which will implement their operators as transducers by this protocol will lead to a new area such that the user can use multiple transducers from multiple libraries at the same code. 

By now you should have a solid understanding of what transducers are, how to implement them and how to extend the implementation for more operators.

----
### Sources

As mentioned before, this tutorial is heavily influenced by the book [Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS) by [Kyle Simpson](https://github.com/getify). I _highly_ recommend you to read his excellent book.

He covers all the topics I explained in great detail.

Also, [Understanding Transducers in JavaScript](https://medium.com/@roman01la/understanding-transducers-in-javascript-3500d3bd9624) by [Roman Liutikov](https://medium.com/@roman01la) which is also _highly_ recommended understanding transducers.

----

### Quiz

###### Question 1

We implemented the compose function which accept multiple functions and an intitial value:

```javascript
const compose = (...funcs) =>
    initialElement =>
        funcs.reduceRight((result, func) => func(result), initialElement);
```

Re-write compose such that it will accept multiple functions and multiple values such that all those values will be sent to the last function provided by the user. The rest of the functions accept a single argument. 

Also, you may not receive functions, so your implementation must support this behavior too (by doing nothing and returning `undefined`). 

###### Question 2

The second version of `compose` accept operators which accept methods from `Array.prototype`. Use the `operator` function to implement the actual operators: `reduce` and `map`. Let the user use them like this:

```javascript
const add = (number1, number2) => number1 + number2;
const multiply = number => number * 10;

compose(
    reduce(add),
    map(multiply)
)([1, 2, 3]);

// output: 
// [1,2,3] -> [10,20,30] -> 60
```

The `Array.prototype` methods were implementation details. Now we completely removed any link to them because the user may choose to use other operators which are not implemented by `Array.prototype`.

###### Question 3

Implement the following transducer:

* `tap` - accept an element, invoke a function which doesn't return a value and then call the next transducer with the arguments he received (mainly for side effects).

###### Question 4

```javascript
const toString = number => number + "-> ";
const isEven = number => number % 2 === 0;

const result = transduce(
                  map(toString),
                  filter(isEven)
              )
              (/* fill_this_argument */)
              ([1, 2, 3, 4]);
```

Fill the second parameter to transduce so `result` will be equal to the string `'2-> 4-> '`.

### Complete terms table

> **Definition 1.1.** In mathematics, ___function composition___ is the pointwise application of one function to the result of another to produce a third function. For instance, the functions f : X → Y and g : Y → Z can be composed to yield a function which maps x in X to g(f(x)) in Z. The resulting composite function is denoted g ∘ f : X → Z, defined by (g ∘ f )(x) = g(f(x)) for all x in X.

> **Definition 1.2.** The ___subroutine/procedure___ may accept input parameters and may return a computed value to its caller (its return value).

> **Definition 1.3.** A ___function___ is a procedure and a process that associates to each element of a set X a unique element of a set Y. Functions consider being _pure fucntions_.

> **Definition 1.4.** A ___unary function___ is a function that takes one argument.

> **Definition 2.1.** In mathematics and computer science, ___currying___ is the technique of translating the evaluation of a function that takes multiple arguments into evaluating a sequence of functions, each with a single argument.

###### Extra terms:

> **Definition 1.** A function or expression is said to have a ___side effect___ if it modifies some state outside its scope or has an observable interaction with its calling functions or the outside world besides returning a value.

> **Definition 2.** A function may be considered a ___pure function___ if both of the following statements about the function hold:
> 1. The function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change while the execution proceeds or between different executions of the program, nor can it depend on any external input from I/O devices.
> 
>2. Evaluation of the result does not cause any semantically observable side effect or output, such as mutation of mutable objects or output to I/O devices.
