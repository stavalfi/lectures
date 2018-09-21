# Functional Programming In JS - Composition And Transducers

### Topics

1. [Introduction](#introduction)
2. [Processing a single element](#processing-a-single-element)
3. [Processing a collection](#processing-a-collection)
4. [Transducers](#transducers)
5. [Conclusion](#conclusion)
6. [Sources](#sources)

### Additional

[Complete terms table](#complete-terms-table)

------

### Introduction
### Processing a single element
### Processing a collection
### Transducers
### Conclusion
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