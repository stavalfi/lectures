# Not Completed!
# Not In Progress!


Local classe by Stav Alfi

### Topics

1. [Introduction](#introduction)
2. [Local class](#local-class)
3. [Quiz](#quiz)
4. [Conclusion](#conclusion)
---
### Introduction


---

### Local class

Local classes and Lambdas have a lot in common.

The following code shows a local class inside `f1`. `f2` uses local variable `z` which is defined inside `f1`.

```java
class MyClass1
{
    int x;
    static MyClass1 y;
    public void f1()
    {
        String z;
        class MyLocalClass1
        {
            public void f2()
            {
                System.out.println(z+x+y.toString());
            }
        }
        MyLocalClass1 i1=new MyLocalClass1();
    }
}
```
How does `f2` get  access to`x`,`y` and `z`?  The compiler sends (as parameters) all the variables that are defined inside the enclosing method (in our example, only `z`) to all the constructors of the local class (in our case, only defualt constructor), so the local class has a copy of their values. That is because `f2` will be invoked in the future and maybe after `f1` ended during runtime. For example if `f1` run a thread that invoke `f2` then maby the thread will start excuting after `f1` already ended. So java gives us a copy of `z` as soon as possible. The local class must have a promise from the compiler which states that the variables it uses (which are not his) will still be alive later. `x` and `y` are guaranteed to be alive so they will be not sent as parameters to the default constructor of the local class.

As we said, the local class has a copy of the local varaibles, so the second reason to prevent mutation of local variables of the enclosing method is the way java is beacuse in case the lambda mutates them and finishes before the enclosing method finishes, a programmer may think that the variables in the enclosing method are up-to-date. That is wrong.

Until Java 7, If the local class wanted to read from those variables, they must have been defined as final. In Java 8 they can also be _effectivlly final_.

> **Definition 1.1.** A variable which wasn't defined as final, but did not change since his first initialization is  ___effectively final___.

> **Definition 1.2.** By referring  ___free variables___ of function `f1` we refer to each variable we can read ,which is inside `f1`, but is not defined in `f1` and nor is a parameter of `f1`.

We may conclude that any _free variables_ can be mutated inside `f2` except local variables of `f1`.

### Quiz

### Conclusion


------------------

### Legal

© Stav Alfi, 2017. Unauthorized use and/or duplication of this material without express and written permission from the owner is strictly prohibited. Excerpts and links may be used, provided that full and clear
credit is given to Stav Alfi with appropriate and specific direction to the original content.

Creative Commons License "Lambda Expressions In Depth By Stav Alfi" by Stav Alfi is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License.](http://creativecommons.org/licenses/by-nc/4.0/)
Based on a work at https://gist.github.com/stavalfi/e24178288973d104042d4162a02fd135.