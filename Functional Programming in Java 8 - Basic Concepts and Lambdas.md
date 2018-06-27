# Functional Programming in Java 8 - Basic Concepts and Lambdas By Stav Alfi

### Topics

1. [Introduction](#introduction)
2. [Functional Interface](#functional-interface)
3. [java.util.function package](#javautilfunction-package)
4. [Lambda Expression](#lambda-expression)
5. [Variable scope](#variable-scope)
6. [Anonymous class](#anonymous-class)
7. [Method reference types](#method-reference-types)
8. [Quiz](#quiz)
9. [Conclusion](#conclusion)

### Additional

1. [Complete terms table](#complete-terms-table)
2. [Variable scope table](#variable-scope-table)
------

### Introduction
This article is meant to build a common language with the reader which will be used to answer the following questions: What are _free variables_? Are we allowed to change them inside the lambda expression? Is it considered dangerous to mutate them?

__Prerequirements__

It is highly recommended that you will control Java Generics.

__Acknowledgements__

The people listed below have provided feedback and edit which has helped improve the quality of this article.

@damirbar - Special thanks to Damir Bar for acting as the provisional grammar consultant.

---

### Functional Interface

> **Definition 1.1.** ___Functional Interfaces___ are interfaces containing single abstract method and any number of default/static methods.

The compiler will define those interfaces as _functional Interfaces_. Also, we can add particular annotation above the interface, so the compiler will warn us if we declare more then one abstract function inside the interface. 

Also, the Javadoc will explicitly mention that this interface is a _Functional interface_. 

The compiler permits us to not explicitly add the `@FunctionalInterface` annotation to an interface with one abstract method so the interface will be _functional Interfaces_, it is optional and recommended.

```java
@FunctionalInterface
interface MyInterface
{
    void f1();
}
```

**An important note about exceptions** - In a case that the method which overrides `f1` can throw exceptions, the declaration of `f1` inside the _functional Interface_ must consist of the possible exceptions which `f1` may throw:
```java
@FunctionalInterface
interface MyInterface
{
    void f1() throws Exception, DataFormatException, IOException;
}
```
The following section contains common _functional interfaces_.

### java.util.function package

The most common _functional interfaces_ defined in `java.util.function` package. More specific _functional interfaces_ defined in other packages.

|Interface::Method                          |    Parameters Type| Return Type| More Info
|:----| :----:| :----:|:----|
|`Function<T,R>::apply`                       |    `T`              | `R`          |  |
|`Consumer<T>::accept`                        |    `T`              | `void`       | Expected to operate via side-effects | 
|`Predicate<T>::test`                         |    `T`              | `boolean`    | |
|`Supplier<T>::get`                           |    None             | `T`          | |
|`BiFunction<T,U,R>::apply`                   |    `T`,`U`          | `R`          | |
|`UnaryOperator<T> implements Function<T,T>`     |    `T`              | `T`          | |
|`BinaryOperator<T> implements BiFunction<T,T,T>`|    `T`,`T`          | `T`          | |
|`ToIntFunction<T>::applyAsInt`               |    `T`              | `int`        | |
|`DoubleConsumer::accept`                      |    `double`         | `void`       | Expected to operate via side-effects |
|`ObjIntConsumer<T>::apply`                   |    `Object`,`int`          | `void`          | Expected to operate via side-effects |

Note: Do not create your open _functional interfaces_., instead use what Java provides in `java.util.function` package or any other packages. You should also read and memorize the use case of each _functional interface_.  For example, by using `Predicate` interface, you should only check if an input parameter is fulfilled in some conditions. You should not do any side effects in the process because the caller will assume you are using this interface as java recommended you to use it. 

Understanding `side-effects` is not mandatory to get the idea and use lambdas.

> **Definition 1.2.** A function or expression is said to have a ___side effect___ if it modifies some state outside its scope or has an observable interaction with its calling functions or the outside world besides returning a value.

An essential concept in _functional programming_ is also _pure functions_.

> **Definition 1.3.** A function may be considered a ___pure function___ if both of the following statements about the function hold:
> 1. The function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change while a program execution proceeds or between different executions of the program, nor can it depend on any external input from I/O devices.
> 
>2. Evaluation of the result does not cause any semantically observable side effect or output, such as mutation of mutable objects or output to I/O devices.

### Lambda Expression

Java 8 introduced a new feature which takes us one step toward functional programming. It is the ability to create functions at runtime with lambda expressions.

> **Definition 2.1.** A variable which wasn't defined as final, but never changed is also   ___effectively final___.

> **Definition 2.2.** By referring  ___free variables___ of a function is all the variables didn't define inside the function nor parameters of that function.

> **Definition 2.3.** ___Lambda Expression___  is the method implementation  of a  _functional interface_. It has zero or more parameters and a body that will (or not) execute at a later stage. Inside the body, we have access to its _free variables_.

```java
Function<Integer,Integer> lambda1=(Integer a) -> a+1 ;
```

Let's break it down:

1. `Function` is the _Functional Interface_ that the lambda is implementing.
2.  The `Function<Integer, Integer>` interface contains, as expected, a single method which accepts an Integer as a parameter and returns Integer.
3.  `lambda1` is the name of the object from a class implementing `Function<Integer, Integer>` interface.
4.  `(.....)` contains all the parameters of the lambda. The number of parameters and their types must correspond to the declaration of the instance method inside `Function`.
5.  `->` separates the parameters and the body of the lambda.
6.  `a+1` is the body of the lambda. It returns the value of `a+1`. In case the body has more than one line, use `{`,  `}` :
```java
int x=10;
Function<Integer,Integer> lambda2=(Integer a) -> {
  a++;
  return a+x;
}
```
To invoke the lambda:
```java
int y=lambda2.applay(10);
```
`y` will be initilize with the value `21`.

### Variable scope

| Variable scope    |   Read  |  Write     |   More Info                              |
| :-----------------| :-----: | :--------: | :--------------------------------------: |
| Lambda expression |   `T`   |  `T`       |                                          |
| Enclosing method  |   `T`   |  `F`       |   Must be _final_ or _effectivly final_  |
| Enclosing class   |   `T`   |  `T`       |                                          |

__Why can't we write to a variable created at the enclosing method?__

1.  To avoid additional synchronizations while running the enclosing method and the lambda body in parallel.

2. The lifetime of a variable created at the enclosing method may have a shorter life than the lambda itself. In other words, the lambda may be invoked after the enclosing method returned. It means we will write to a variable that may not exist.

We conclude that the lambda must keep somehow a copy of all the variables it uses in the enclosing method. The current implementation of Java **may** add all the variables (in the enclosing method or class) the lambda read or write to the list of parameters the lambda get. It **may** also add only the variables from the enclosing method to the list of parameters because the rest can be reached in other ways. The lambda it private static method or private instance method inside the enclosing class. The decision of making it static method or instance method is decided by (can be changed in the future): If the lambda does not use any variable of the enclosing class, it **may** become a static method. Else it **may** become an instance method.

For example:

```java
public class A {
  
    public static void main(String[] args) {
        Consumer<Integer> f2 = (Integer x) -> System.out.print(x);
    }
}
```
After complining:
```java
javac A.java
javap -p A
```
Output:
```java
Compiled from "A.java"
public class A {
  public A();
  public static void main(java.lang.String[]);
  private static void lambda$main$0(java.lang.Integer);
}
```

Another example:

```java
public class A {

    int y=9;
    public void f1() {
        Consumer<Integer> f2 = (Integer x) -> System.out.print(y);
    }
}
```
After complining:
```java
javac A.java
javap -p A
```
Output:
```java
Compiled from "A.java"
public class A {
  int y;
  public A();
  public void f1();
  private void lambda$f1$0(java.lang.Integer);
}
```

### Anonymous class

It is a common mistake that lambdas translated to objects of anonymous classes that implement the functional interface. When creating an object of an anonymous class, first a class file is generated and then a new object is initialized. Creation of a file for each object of an anonymous class is a huge overhead to the VM in runtime when lambdas are widely used in every application.

### Method reference types

When your lambda has one line, calls a method with all the parameters of the lambda and return what that function returns then you better use method reference.


The following line
```java
Consumer<Integer> lambda1 = (Integer a) -> System.Out.println(a);
```
is recoomended to be written as:
```java
Consumer<Integer> lambda1 = System.Out::println;
```
The parameter of the lambda is used for calling to the method `println` without us needing to write them explicitly.

There are 4 types of Method references: Object::instanceMethod, Class::staticMethod, class::InstanceMethod and class::contructor.

__Object::instanceMethod__

as the name implies, the lambda calls a method of an object. The first example applies here too:
```java
Consumer<Integer> lambda1 = System.Out::println;
lambda1.accept("hi!!"); // will print: hi!!
```

__Class::staticMethod__

```java
public class App {
  
    private static int f1() {
        return 10;
    }
    
    public static void main( String[] args )
    {
        Supplier<Integer> lambda1=App::f1; // () -> App.f1();
        int x=lambda1.get(); // x = 10
    }
}
```

__class::InstanceMethod__

The first parameter will invoke one of his methods and use the second parameter as an argument:

```java
public class App {
  
    public int f1() {
        return 8;
    }
    
    public static void main( String[] args ) throws IOException
    {
        Function<App,Integer> lambda1 = App::f1; // (App a)-> a.f1();
        int x = lambda1.applay(new App()); // x = 8
    }
}
```
```java
public class App {
  
    public int f2(App a) {
        return 16;
    }
    
    public static void main( String[] args ) throws IOException {
        BiFunction<App,App,Integer> lambda1 = App::f2; // (App a,App b) -> a.f2(b);
        int y = lambda1.applay(new App()); // y = 16
    }
}
```
__class::contructor__

```java
Function<Integer,int[]> lambda1 = int[]::new; // (int x)-> new int[x];
int[] array = lambda1.apply(10);
```

```java
class A
{
    public static void main(String[] args)
    {
        Supplier<A> lambda1 = A::new; // (int x)-> new A();
        A a1 = lambda1.get();
    }
}
```

__Note:__ It is possible to use `super`/ `this` as an object inside methods:
```java
class A
{
    void f1(int x)
    {
        System.out.println("father is printing!");
    }
}
```
```java
class B extends A
{
    B()
    {
        Consumer<Integer> lambda1 = super::f1; // (int x) -> super.f1(x);
        lambda1.accept(10); // print: father is printing!
    }
    void f1(int x)
    {
        System.out.println("son is printing!");
    }
}
```
### Quiz
The Answers are at the end of the quiz.

###### Question 1

Search for `Runnable` funcation interface's method signature. Could we write inside the following code: `Runnable runnable1 = this::f1;`? Name a _readability_ disadvantage while sending `this::f1` as a parameter to a method.
```java
// Class MyClass1
public void f2() {
    Runnable runnable1 = () -> {
        System.out.println("hi");
    };
}

public void f1() {
    System.out.println("hi");
}
```
[Answer 1](#1-question-with-answer)

###### Question 2

What are the differences between the following _functional interfaces_:  `Runnable`, `Callable` and `Comparable`? Write lambdas using those _functional interfaces_ and test them out.

[Answer 2](#2-question-with-answer)

###### Question 3

What are the differences between _side effects_ and _pure function_?

[Answer 3](#3-question-with-answer)

###### Question 4

What are the four main new _funcational interfaces_ in `Java 8`? Which of them can be implemented with _side effects_ or _pure function_?

[Answer 4](#4-question-with-answer)

###### Question 5

What does the following code prints?
```java
Predicate<Integer> p1 = (Integer x) -> x % 2 == 0;
Predicate<Integer> p2 = (Integer x) -> x % 3 == 0;

System.out.println(p1.or(p2).test(1));
System.out.println(p1.and(p2).test(2));
System.out.println(p1.and(p2).and(p1).test(3));
System.out.println(p1.negate().test(4));
```

[Answer 5](#5-question-with-answer)

###### Question 6

Fill the types which are missing (`Type1`,`Type2`,`Type3`):
```java
Function<Type1,Type2> f1=(Integer x)->"hi"+x;
f1.andThen((Type3 something)->something.length());
```
What does this code prints?

[Answer 6](#6-question-with-answer)

###### Question 7

What does this code prints?
```java
Function<String, Integer> f1 = (String x) -> x.length();
Integer[] x = f1.andThen((Integer number) -> new Integer[number])
        .apply("123");
```

[Answer 7](#7-question-with-answer)

<br/><br/>
In the following questions, you need to determine if there is any compilation error, undefined behavior or there is nothing wrong. They are confusing and tricky. Take your time and explain your answer with as many details as possible.

###### Question 8

Assume all the threads don't fail to run.

```java
static int count=1;
public static void f1() {
  
  for(int i=1;i<=10;i++)
    new Thread(()->count++).start();
}
```

[Answer 8](#8-question-with-answer)

###### Question 9

Assume all the threads don't fail to run.

```java
public static void f2() {
  
  for(int i=1;i<=10;i++)
    new Thread(()->System.out.println(i)).start();
}
```

[Answer 9](#9-question-with-answer)

#### Answers: (With the questions)

###### 1. Question with Answer

###### Question

Search for `Runnable` funcation interface's method signature. Could we write inside the following code: `Runnable runnable1 = this::f1;`? Name a _readability_ disadvantage while sending `this::f1` as a parameter to a method.
```java
// Class MyClass1
public void f2() {
    Runnable runnable1 = () -> {
        System.out.println("hi");
    };
}

public void f1() {
    System.out.println("hi");
}
```
######  Answer

Yes, we could. The implementation of `Runnable` function interface is:
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

To understand the disadvantage, let us examine the following code:

```java
class Application{
    static void doSomeWork() {}
    
    static void f2(Runnable r1) {
        r1.run();
    }
    
    static void doSomeWork(String str) {}
    
    static void f3(Consumer<String> r1) {
        r1.accept("333333");
    }
    
    public static void main(String[] args) {
        // part A
        f2(() -> doSomeWork()); // line 1
        f2(Application::doSomeWork); // line 2
        // Part B
        f3((String str) -> doSomeWork(str)); // line 3
        f3(Application::doSomeWork); // line 4
    }
}
```
In line `1` we are sending to `f2` a lambda which calls `doSomeWork` with no params. We __explicitly see__ that we dont send any params and `doSomeWork` does not get any params. in line `2` we can't __explicitly see__ it; in line `2` we must go to the signature of `doSomeWork` (which we will find 2 of them) and understand if `doSomeWork` get params or not. The only way to understand which overload of `doSomeWork` we need to look into is by looking at the signature of `f2` and see that it gets `Runnable` which means that the overload of `doSomeWork` we are searching for, doesn't get any params.

We have the same problem of readability in line `4`. It means that by the only __looking with our eyes__ at lines `2` and `4`, we can't understand what the signature of `doSomeWork` is and we can't understand that there are two overloads to `doSomeWork` method. But by looking at lines `1` and `3`, we can easily see that there are two overloads to the method `doSomeWork`.

[Question 2](#question-2)

###### 2. Question with Answer

###### Question

What are the differences between the following _functional interfaces_:  `Runnable`, `Callable` and `Comparable`? Write lambdas using those _functional interfaces_ and test them out.

######  Answer

The functional interfaces' method signatures:

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}

// it is optional to specify @FunctionalInterface here.
public interface Comparable<T> {
    public int compareTo(T o);
}
```

They all have a different signature, so they not the same. Also, by the implementation details(java docs) we can see that they are intended for different use cases. The only way to know them is to read them. Misusing them will lead to bugs between different developers.

[Question 3](#question-3)

###### 3. Question with Answer

###### Question

What are the differences between _side effects_ and _pure function_?

######  Answer

The definitions from Wikipedia are:

> **Definition 1.2.** A function or expression is said to have a ___side effect___ if it modifies some state outside its scope or has an observable interaction with its calling functions or the outside world besides returning a value.

> **Definition 1.3.** A function may be considered a ___pure function___ if both of the following statements about the function hold:
> 1. The function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change while the execution proceeds or between different executions of the program, nor can it depend on any external input from I/O devices.
> 
>2. Evaluation of the result does not cause any semantically observable side effect or output, such as mutation of mutable objects or output to I/O devices.

[Question 4](#question-4)

###### 4. Question with Answer

###### Question

What are the four main new _funcational interfaces_ in `Java 8`? Which of them can be implemented with _side effects_ or _pure function_?

######  Answer

The four main new _functional interfaces_ in `Java 8` are: `Consumer`, `Supplier`, `Function` and `Predicate`.

1. `Consumer` - Get an object and do something with it. We are allowed to use _side effects here_.
2. `Supplier` - (from the java docs) Represents a supplier of results.
There is no requirement that a new or distinct result be returned each time the supplier is invoked.

It means we implement a `Supplier` not as a _pure function_.

1. `Function` - Get an object from type `A` and return object from type `B` (Type `A` can be equal to type `B`). No _side effects_ are allowed here. But sometimes we will implement it as a transformer from _id_ to an object we retrieved from a server. A server call has _side effects_.

2. `Predicate` - Get an Object a determine if it satisfies a condition.  No _side effects_ are allowed here. No _side effects_ are permitted here. 

[Question 5](#question-5)

###### 5. Question with Answer

###### Question

What does the following code prints?
```java
Predicate<Integer> p1 = (Integer x) -> x % 2 == 0;
Predicate<Integer> p2 = (Integer x) -> x % 3 == 0;

System.out.println(p1.or(p2).test(1));
System.out.println(p1.and(p2).test(2));
System.out.println(p1.and(p2).and(p1).test(3));
System.out.println(p1.negate().test(4));
```

######  Answer

It will print:
```java
false
false
false // In this case, there is no effect to the extra `and(p1)`.
false
```

[Question 6](#question-6)

###### 6. Question with Answer

###### Question

Fill the types which are missing (`Type1`,`Type2`,`Type3`):
```java
Function<Type1,Type2> f1=(Integer x)->"hi"+x;
f1.andThen((Type3 something)->something.length());
```
What does this code prints?

######  Answer

Solution:
```java
Function<Integer,String> f1=(Integer x)->"hi"+x;
f1.andThen((String something)->something.length());
```
The code doesn't print anything. We use `f1` as it was a normal object, which means we need to call one of it's methods to invoke something. In this case, we need to write `f1.apply(some value)`.

[Question 7](#question-7)

###### 7. Question with Answer

###### Question

What does this code prints?
```java
Function<String, Integer> f1 = (String x) -> x.length();
Integer[] x = f1.andThen((Integer number) -> new Integer[number])
        .apply("123");
```

######  Answer

This code prints nothing because we didn't send anything to the screen. But the call to `apply("123")` will return `integer[]{0,0,0}`.

[Question 8](#question-8)

###### 8. Question with Answer

###### Question

Assume all the threads don't fail to run.

```java
static int count=1;
public static void f1() {
  
  for(int i=1;i<=10;i++)
    new Thread(()->count++).start();
}
```

######  Answer

Answer: undefined.\
`count` is a static property so as a result, he will never be _effectivly final_. Also, we can read and write to a static variable from the lambda's body. Due to the absence of synchronization on `count`, we conclude that the result is undefined.

[Question 9](#question-9)

###### 9. Question with Answer

###### Question

Assume all the threads don't fail to run.

```java
public static void f2() {
  
  for(int i=1;i<=10;i++)
    new Thread(()->System.out.println(i)).start();
}
```

######  Answer

Answer: Compilation error.\
`i` is a _free variable_  and also is a local variable of the enclosing function. `i` will be mutated after its initialization so it can't be used inside the lambda. It must be defined as defined as _final_ or _effectively final_. So the lambda can read it's value. 

### Conclusion

The big difference between method reference and lambda expression is method reference references to the method directly it should invoke. The lambda is responsible for invoking a method (in case there is a call to a method inside the lambda body). It means we called two methods instead of one.

------------------

### Legal

© Stav Alfi, 2017. Unauthorized use or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear
credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Lambda Expressions In Depth By Stav Alfi" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/e24178288973d104042d4162a02fd135.


---

### Complete terms table

> **Definition 1.1.** ___Functional Interfaces___ are interfaces containing single abstract method and any number of default/static methods.

> **Definition 1.2.** A function or expression is said to have a ___side effect___ if it modifies some state outside its scope or has an observable interaction with its calling functions or the outside world besides returning a value.

> **Definition 1.3.** A function may be considered a ___pure function___ if both of the following statements about the function hold:
> 1. The function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change while the execution proceeds or between different executions of the program, nor can it depend on any external input from I/O devices.
> 
>2. Evaluation of the result does not cause any semantically observable side effect or output, such as mutation of mutable objects or output to I/O devices.

> **Definition 2.1.** A variable which wasn't defined as final, but never changed is also   ___effectively final___.

> **Definition 2.2.** By referring  ___free variables___ of a function is all the variables didn't define inside the function nor parameters of that function.

> **Definition 2.3.** ___Lambda Expression___  is the method implementation  of a  _functional interface_. It has zero or more parameters and a body that will (or not) execute at a later stage. Inside the body, we have access to its _free variables_.

### Variable scope table

| Variable scope    |   Read  |  Write     |   More Info                              |
| :-----------------| :-----: | :--------: | :--------------------------------------: |
| Lambda expression |   `T`   |  `T`       |                                          |
| Enclosing method  |   `T`   |  `F`       |   Must be _final_ or _effectivly final_  |
| Enclosing class   |   `T`   |  `T`       |                                          |