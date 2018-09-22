# Functional Programming In JS - Composition And Transducers By Stav Alfi

### Topics

1. [Introduction](#introduction)
2. [Processing a single element](#processing-a-single-element)
3. [Processing a collection](#processing-a-collection)
4. [Transducers](#transducers)
5. [Quiz](#quiz)
6. [Conclusion](#conclusion)
7. [Sources](#sources)

### Additional

[Complete terms table](#complete-terms-table) (Wikipedia)

---
_General note:_ This tutorial is heavily influenced from the book [Functional-Light JavaScript](https://github.com/getify/Functional-Light-JS) by [Kyle Simpson](https://github.com/getify). I _highly_ recommend you to read his great book.

_Java note:_ Due to ansense of composition syntax support, there is no way to implement the code without reflections and wildcards (`?`).

------

### Introduction

__Prerequirements__

* `ES6/ES2015` Syntax: Arrow functions and spreed operator.
* Clusures.
* reduce, reduceRight, map, filter functions.
* Basic terminolegy of functional programming: Side effects and pure functions by wikipedia definitions.
* Imperative vs declerative code.

----

Throughout this tutorial we will cover a series of basic teqenics for manipulating collection of functions. 

Functional programming is all about functions; the logic. Using special teqenics, we will remove most of the unnessesery informaition from the implementation to empesize our actual logic. The basic fundamentals of programming is deviding our logic to multiple procedures such that each area will do something different. More procedures, the easier for us to test and understand the overall picture. Functional programming is written using declerative code so the implementation details, which are harder to understand, will be less visible. 

To avoid a function calling directly to other function we need a support from our programming language or using a third function for defining a relationship between those two functions. Why is that important? We would like to have the opretonity to invoke more then one unique function after a specific function or we will have to implement the same function multiple times such that each implementation will call a different function. One teqnique for doing so is called _function composition_ by sending the result of the first function to the second function as a parameter (the functions invocations are done by a utility). Javascript doesn't have (yet) the syntax for compose functions, so we will have to build or use a utility for using function compositions.

> **Definition 1.1.** In mathematics, ___function composition___ is the pointwise application of one function to the result of another to produce a third function. For instance, the functions f : X → Y and g : Y → Z can be composed to yield a function which maps x in X to g(f(x)) in Z. The resulting composite function is denoted g ∘ f : X → Z, defined by (g ∘ f )(x) = g(f(x)) for all x in X.

The utilities we will build during this tutorial is the `Z` function and the positive outcome will make keep any function like `f` and `g` free from directly calling other functions.

If you notices, I also used 2 different ways to describe code: _procedure_ and _function_. Let's define them also:

> **Definition 1.2.** The ___subroutine/procedure___ may accept input parameters and may return a computed value to its caller (its return value).

> **Definition 1.3.** A ___function___ is a procedure and a process that associates to each element of a set X a unique element of a set Y. functions consider to be _pure fucntions_.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Function_machine2.svg/1200px-Function_machine2.svg.png" width="200" height="200" style="display: block;margin-left: auto;margin-right: auto;">

The only difference between them is that a procedure sometimes does not accept input paramters or a return value so a composition between procedures which are not functions is impossible. Also procedures which does not accept parameters or doesn't return anything won't defentely be pure function. Functional programming is all about deterministic functions.

To conclude, we will talk about pure functions which accept a single argument. Any other functions won't fit to the definition of composition.

> **Definition 1.4.** A ___unary function___ is a function that takes one argument.

----

### Processing a single element

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Function_machine5.svg/2000px-Function_machine5.svg.png" width="300" height="400" style="display: block;margin-left: auto;margin-right: auto;">

In the beggining of our jeorny we will implement a utility to process a single element by sending the element to a function `f1` and then the result will be sent to function `f2` and so on...

```javascript
const result = compose(...,f3,f2,f1)(element);
```
Our job is to implement `compose` function. It will send `element` to `f1` and the result will be sent to `f2` and so on until the last function is invoked it's result will be the `result`.

Why I splitted the params to two groups: functions and the input element such that each group is given to us in different function? Then we can use the same function multiple times and not worry about supplying allt he parameters at the same moments (sometimes we can't because we currently don't know all of them):

```javascript
const composition = compose(...,f3,f2,f1)
const result1 = composition(element1);
....
const result2 = composition(element2);
```

This tequenic is called curring.

> **Definition 1.4.** In mathematics and computer science, ___currying___ is the technique of translating the evaluation of a function that takes multiple arguments into evaluating a sequence of functions, each with a single argument.

Thanks to `ES6` spread operator syntax, we can cheat a little bit by accepting multiple functions as a single array.

Let's implement compose function:

```javascript
const compose = (...funcs) =>
    initialElement =>
        funcs.reduceRight((result, func) => func(result), initialElement);
```

As implicitly specifing it before, there is a repeated pattern here; we use function and the last result to produce a new result: `(result, func) => func(result)`. We don't need any additional values to produce the final result except the intitial element. In situations like that, `reduce` is the first thing that should come up in your mind. But why `reduceRight`? Remmber that the goal is to implement something that is similar to mathematical composition and not something like _pipe_ in unix. The first function we invoke is the last the user provided to `compose`.

Usage examples:

```javascript
const composition = compose(number => number + "!", number => number * 10);
composition(1); // 1 -> 10 -> 10!
composition(10); // 10 -> 100 -> 100!
```
----

### Processing a collection



----
### Transducers



----
### Quiz

###### Question 1

We implemented the compose function which accept multiple functions and an intitial value:

```javascript
const compose = (...funcs) =>
    initialElement =>
        funcs.reduceRight((result, func) => func(result), initialElement);
```

Rewrite compose such that it will accept multiple functions and multiple values such that all those values will be sent to the last function provided by the user. The rest of the functions accept a single argument. 

Also, you may not receive functions so your implementation must support this behavior too (by doing nothing and returning `undefined`). 

----
### Conclusion


----
### Sources





----
### Complete terms table




// intro:
// a problem in oop?
// solution: function compositions (global functions)
// function compositions - advantages
// tutorial:
// arity, unary functions, procedure, function
// high order functions, curring
// write compose function and show how one element is processed by it:
// const compose = (...functions) => functions.reduceRight((composedFunction, currentFunction) =>
//     (...params) => currentFunction(composedFunction(...params)));
// process array using functions composition (f1)
// improve it by removing the need to know Array.prototype directly. (f2)
// talk about significant performance issue
// solution: Transducers: (f3)
// implement operator using reduce operator (demonstrating on filter and map functions as they are stateless and fully functional)
//      because then I can rebuild the array and remove elements from the result array if I want.
// show how to use the final My-Map,My-Filter functions directly.
// show how they can be composed directly without compose function:
// const result3 = map(increase)(
//     filter(isEven)(expandArray)
// )([1, 2, 3], 5);
// const result = [1, 2, 3, 4].reduce(map(increase)(filter(isEven)(expandArray)), []);
// improve the code by using compose function - for user readability.
// why compose is a bug here.
// pipe
// use pipe
// build Transducers function.
// conclusion
// terms
// sources