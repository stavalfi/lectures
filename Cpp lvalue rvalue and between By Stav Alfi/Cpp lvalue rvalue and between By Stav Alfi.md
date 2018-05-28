# C++ lvalue, rvalue and between By Stav Alfi

### Topics

1. [Introduction](#introduction)
2. [Object definitions](#object-definitions)
3. [Common misunderstanding](#common-misunderstanding)
4. [Expressions vs Types](#expressions-vs-types)
5. [Move consturctor and opertator](#move-constructor-and-operator)
6. [Source](#source)
7. [Complete terms table](#complete-terms-table)
8. [Legal](#legal)

---

### Introduction

This tutorial attempts to clarify the common mistakes about the multiple invalid definitions for `lvalue` and `rvalue` expressions.

### Object definitions

I will start by introducing the common, valid and different definitions for the term `obejct`. 
Throughout this tutorial, I will implicitly use the first definition.

> **Definition 1.1.** An ___object in c programing language___  is an area in the memory which contain an address we can reach which _may_ contain data.
 
For example, function or an instance of a struct.
 
> **Definition 1.2.** An ___object in OPP___ is an instance of a class.

Also I will define one more common term, I will use alot:

> **Definition 1.3.** An ___temporary object___ is an object of class type. it does not have a _name_ (we can't reach this object after it's evaluation). It has an address on the stack and it will be destoryed __after__ evaluating the full expression which which has a "some link" to the expression who created that temporary object, unless we _named_ that temporary object. By naming I mean, creating a "way" to reach that temporary object which the compiler approve (This tutorial will teach you how to do it).

###### Examples:
From definition 1.3: "it will be destoryed after evaluating the expression which created him...":
```java
struct A {
    A(){
        cout<<"A::A()"<<endl;
    }
    A(const A& a){
        cout<<"A::A(const A& a)"<<endl;
    }
    ~A(){
        cout<<"A::~A()"<<endl;
    }
};

A f1(){
    return A(); // return a temporary object
}
A f2(A a) {
    cout<<"f2"<<endl;
    return a;
}
int main() {
    f2(f2(f1()));
    return 0;
}
```
Stack trace:
```java
A::A()
A::A(const A& a)
A::~A()
A::A(const A& a)
f2
A::A(const A& a)
A::A(const A& a)
f2
A::A(const A& a)
A::~A()
A::~A()
A::~A()
A::~A()
A::~A()
```

As you can see, all the temporary objects are destoryed after evaluating the full expression. Also I will add to the denition above that the order they are detoryed is opposite to the order they were created.

Note: to compile the above code, please add `-fno-elide-constructors` flag to the g++ compilation command to avoid comipler's consturctor optimizations.
###### End of examples

### Common misunderstanding

The [standart](http://eel.is/c++draft/basic.lval#3) explicitly say:

>  Historically, lvalues and rvalues were so-called because they could appear on the left- and right-hand side of an assignment (although this is no longer generally true); ... Despite their names, these terms classify expressions, not values.

### Expressions vs Types

![](https://i.imgur.com/hMZPzj4.png)

To better understand what are `lvalue` and `rvalue` expressions, lets define the following expressions:

> **Definition 2.1.** A ___glvalue___ is an expression which it's evaluation is an _object, function or bitfield_ which we can choose _(we can also choose not)__ to refer later (_later_ is equal to - after the evaluation of the given expression).
  
By definition, __any__ expressions which it's evaluation is a reference to object type, are also glvalue expression.

###### Examples:

```java
int f(); // glvalue - this expression is a function. by defenition, this expression is a glvalue.
&f;      // glvalue - this expression evaluates a pointer which we can refer to later.
           // by defenition, this expression is a glvalue.
f();      // not glvalue - this expression evaluates a call to function f which return a temporary value.
           // we can't refer to that specific temporary value because after the function will finish executiong,
           // the temporary object will be destoryed.
```

```java
class A{
    int k;
    const A& f1() { return A();} // compiler warning: returning reference to temporary.
    const int& f2() { return k;}  // note: k is glvalue.
};

A a;
a.f1(); // it doesn't matter if this expression is a glvalue or not.
           // it's undefined - can't use the returned object. it already destoryed.
a.f2(); // glvalue - this expression return a const reference to k. k can be accessed (refer to) after evaluating this expression.
```

```java
const int x=4;
x; // glvalue - x can be accessed after executing this expression.
```

```java
int i;
int* p = &i;
int& f();
int&& g(); // note: T&& is eventually a reference to object of type T. * We will talk about how is it different from T& soon. *

int h();

h(); // not glvalue
g(); // glvalue
f();  // glvalue
i;     // glvalue
*p;  // glvalue
```

###### end of examples

We noticed that a `glvalue` expression evaluate object or reference which can __sometimes__ be assigned too and __always__ can be read from.

`glvalue` expression is a `lvalue` or a `xvalue` expression only.

To better understand what is a `xvalue` expression, first we need to understand what are `prvalue` expression and `rvalue reference` type.

> __Definition 2.2.__ A ___prvalue___ (pure rvalue) is an expression which it's evaluation is a _temporary object_.

###### Examples:

```java
int f1() { return 6 };

6;      // prvalue
f1();   // prvalue
```

###### end of examples

> __Definition 2.3.__ A ___rvalue reference___ is a type (not an expression) of a special reference: `T&&` or `const T&&`. An _rvalue reference_ is a reference to temporary object.

__Important note:__ As soon as there are no active references to that temporary object (it can happen after the end of scope of all `rvalue references` to this temporary object), this temporary object will be destoryed. We will use this note later on.
    
###### Examples:

```java
const int&& f1() { return 6; } // f1 return type is rvalue reference.
A&& f2() { return A(); }       // f2 return a reference to prvalue.

int&& x=f1(); // x is a reference. to be more precise - it is a rvalue reference 
		       // (soon we will be able to answer what excatly is the expression f() ).
```

###### end of examples

###### Common mistake:

__False claim:__ if expression `e` is a `rvalue reference` expression,
 then `e` is a `lvalue` expression. 
 <br/>
__Contradiction:__ `rvalue reference` is a type and not an expression.

###### End of common mistake

There are 4 ways to create an `rvalue reference` type:

1. Return from a function a temporary object - `int&& f1() { return 6; }`
2. Assigning a temporary object directrly to `rvalue reference`varaible.
3. `static_cast<T&&>(....)` - `int&& x = static_cast<int&&>(6);`
4. `std::move<T>(....)` - `int&& x = std::move<int&&>(6);`

`std::move` source code:

```java
  /**
   *  @brief  Convert a value to an rvalue.
   *  @param  __t  A thing of arbitrary type.
   *  @return The parameter cast to an rvalue-reference to allow moving it.
  */
  template<typename _Tp>
    constexpr typename std::remove_reference<_Tp>::type&&
    move(_Tp&& __t) noexcept
    { return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }
```

> Definition 3.3. A _recursive_ definition to ___xvalue___ is a _glvalue_ expression which is one of the following forms:
> 1. A call to a function which return a _rvalue reference_ to an object of type _T_ (not function).
> 2. a convert _to_ type of _rvalue reference_. (for example, `static_cast<T&&>(...)`).
> 3. an access to _xvalue_ expression's class member `c`. The evaluation of a _xvalue_ expression must be an object of type _T_. 


###### Examples:

```java
struct A{ int data; };

int&& i;
int& f();
int&& g();
A&& g1();

int h();

h();       	// not glvalue - rvalue
g();       	// glvalue - xvalue
g().data;  // glvalue - xvalue
f();       	// glvalue - lvalue
i;          	// glvalue - lvalue
```

```java
const int&& f1() { return 6; } // f1 return rvalue reference - a reference to rvalue expression.

int&& x=f1(); 
x;      // glvalue - lvalue
f1();   // glvalue - xvalue
```

###### end of examples

> __Definition 4.1.__ A ___rvalue___ is an expression which is not a _lvalue_.

Note: A _xvalue_ expression can be considered to be `glvlue` and `rvalue` expressions.

There are 2 ways to create `rvalue` expression: expression which it's evaluation is `xvalue` or `prvalue`. We will see now that there is an implicit cast from `prvalue` expression's evaluation (which is an object) to oject of type `rvalue reference`.

### Move constructor and operator:

##### Move constructor

As we saw earlier, we can't modify a temporary objects  or even modify a non-static member of a class though `prvalue` expressions. Trying to do so, will lead to compilation error: `using temporary as lvalue`. 

###### Examples:
```java
struct A {
    int* bigArray;
    A():bigArray(new int[2^64]) {
        cout<<"A::A()"<<endl;
    }
    A(const A& a) {
        cout<<"A::A(const A& a)"<<endl;
        // ...deep copy bigArray member....
    }
    ~A() {
        cout<<"A::~A()"<<endl;
        delete this->bigArray;
    }
};

A f1() {
    cout<<"f1()"<<endl;
    return A();
}

void f2(A a) {
    cout<<"f2(A a)"<<endl;
}

int main() {
    f1().bigArray=new int[4];  // compilation error: using temporary as lvalue
    cout<<"finsihed f1().bigArray=new int[4]"<<endl;
    ...
}
```

Stack trance (incase there was no error):

```java
f1()
A::A()
A::A(const A& a)
A::~A()
finsihed f1().bigArray=new int[4]
A::~A()
```

###### End of examples

What if we want to send the result of `f1` to `f2`: `f2(f1())`? Then The stack, in this example, will looks like:

###### Examples
```java
int main() {
    f2(f1());
    cout<<"end of f2(f1());"<<endl;
    ...
}
```
Stack trance:
```java
f1()
A::A()
A::A(const A& a)
A::~A()
A::A(const A& a)
f2(A a)
A::~A()
A::~A()
end of f2(f1())
```
###### End of examples

Well, calling twise to A::copy constructor is unacceptable for us. Why? Because we are deep-copy 2 times the member `bigArray` insead of zero times. 

Why we would want to avoid, _ in this case_, deep-copy (2 times) the member `bigArray`?  Because `f1()` is a `prvalue` expression so it means we can't modify it's evaluation (which is an object). so I would like to, magicaly, send this object, which we can'y modify, directly to the parameter of `f2` so `f2` will use this object, which we can't modify, and let `f2` use it as it want and even modify it by dark-megic! By doing so, we avoided creating 2 extra objects, we don't need. One of them is a temporary object it self which will be created in the `main`: `f2( -->>here<<--)`.

How do we accomplish it? By creating additional consturctor:  _move consturctor_ in `A` class:

```java
A(A&& a) {
     cout<<"A::A(A&& a)"<<endl;
     this->bigArray=a.bigArray;
      // when the destructor of the temporary object which `a` is
      // referencing to is called, our bigArray won't be deleted.
     a.bigArray=nullptr;
}
```

Thats it! 

```java
f1()
A::A()
A::A(A&& a)
A::~A()
A::A(A&& a)
f2(A a)
A::~A()
A::~A()
end of f2(f1());
```
Same as for operator move.

### Source:
* [C++ standard](http://eel.is/c++draft/basic.lval)
* [stackoverflow.com - user2079303's answer](https://stackoverflow.com/a/47956898/806963)
* [stackoverflow.com](https://stackoverflow.com/questions/27364787/glvalue-real-examples-and-explanation/27364969#27364969)
* [thbecker.net](http://thbecker.net/articles/rvalue_references/section_03.html)

### Complete terms table

> **Definition 1.1.** An ___object in c programing language___ 
 is an area in the memory which contain an address we can reach which _may_ contain data.
 
> **Definition 1.2.** An ___object in OPP___ is an instance of a class.

> **Definition 1.3.** An ___temporary object___ is an object of class type. it does not have a _name_ (we can't reach this object after it's evaluation). It has an address on the stack and it will be destoryed __after__ evaluating the full expression which which has a "some link" to the expression who created that temporary object, unless we _named_ that temporary object. By naming I mean, creating a "way" to reach that temporary object which the compiler approve (This tutorial will teach you how to do it).

> **Definition 2.1.** A ___glvalue___ is an expression which it's evaluation __(not the actual value it evaluates)__
 is an _object, function or bitfield_ which we can choose __(we can also choose not)__ to refer later
  (_later_ is equal to - after the evaluation of the given expression).
  
> __Definition 2.2.__ A ___prvalue___ (pure rvalue) is an expression which it's evaluation is a temporary object.
 A temporary object can't be refer to after we evaluate it.
  
> __Definition 2.3.__ A ___rvalue reference___ is a type (not an expression) of a special reference: `T&&` or `const T&&`. An _rvalue reference_ is a reference to temporary object.
    
> __Definition 2.4.__ A ___rvalue___ is an expression which is not a _lvalue_.
    
### Legal

© Stav Alfi, 2017. Unauthorized use and/or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear
credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "C++ lvalue, rvalue and between By Stav Alfi" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/52560f2b0d57d97b34ecae21f0bc9fa9.