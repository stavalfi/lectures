# C++ Extercises By Stav Alfi -updated at 15/8/2017

##### Questions 1
```cpp
#include <iostream>

using namespace std;

void f1()
{
    cout<< "f1()" <<endl;
}
void f1(int x = 5)
{
    cout<< "f1(int x = 5)" <<endl;
}

int main() {
    f1();
    return 0;
}
```
What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
---
##### Questions 2
``` cpp
//
// main.cpp:
//

#include <iostream>

// c1.h have the following functions:
// 1. void f1(int x); (f1 implem' is : int f1(int x){ return x; })
#include <c1.h>

int main() {
    
    f1(4);

    return 0;
}
```
What is a potential output after running the following commands:
```
gcc -c c.c
g++ -std=c++11 -c main.cpp
g++ main.o c.o
./a.out
```
---
##### Questions 3
```cpp
#include <iostream>
using namespace std;

struct A
{
    int x = 5;
};
class B: A
{
    int y = 10;
};

int main() {
    struct A a1;
    A a2;
    class B b1;
    B b2;
    
    cout << a1.x << endl;
    cout << a2.x << endl;
    cout << b1.y << endl;
    cout << b2.y << endl;
    cout << b1.x << endl;
    cout << b2.x << endl;
    
    return 0;
}
```
* What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
* When we should write: `MyStruct s1` instead of `struct MyStruct s1`?
* When we should use: `hpp` instead of `h`?
---
##### Questions 4

``` cpp
#include <iostream>
using namespace std;

struct A
{
    A(int x)
    {
        
    }
};

int main() {
    
    A a1;
    A a2= a1;
    a2 = a1;
    return 0;
}
```
What is the consequences of writing the constructor as above?
* Is there still a defualt constructor?
* Is there a copy constructor?
* Is there a operator= ?

* What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
---
##### Questions 5

```cpp
#include <iostream>
using namespace std;

struct A
{
    A* a1;
    A()
    {
        cout << "A::A()" << endl; // 1
    }
    A(const A& a1)
    {
        cout << "A::A(const A a1)" << endl; // 2
    }
    A operator= (const A a1)
    {
        cout << "A::operator=(const A a1)" << endl; // 3
        return a1;
    }
    ~A()
    {
        cout << "A::~A()" << endl; // 4
    }
};

int main() {
    A array[1];
    array[0] = A();
    cout << "----" << endl;
    return 0;
}
```
What is the output after running the following commands:
```
g++ -std=c++11 -fno-elide-constructors -c main.cpp
g++ main.o
./a.out
```
---
##### Questions 6

```cpp
#include <iostream>
using namespace std;

struct A
{
    int x;
    A():x(5)
    {
        cout << "A::A()" << endl; // 1
    }
    A(int y):x(y)
    {
        cout << "A::A(int y)" << endl; // 2
    }
    void operator()(int y)
    {
        cout << "A::operator()" << endl; // 3
    }
};

struct B
{
    A a1;
    B()
    {
        a1(10);
    }
};

int main() {
    
    B b1;
    return 0;
}
```
What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
---
##### Questions 7

```cpp
#include <iostream>
using namespace std;

struct A
{
    int x = 5;
    int& operator()()
    {
        cout << "A::operator()" << endl;
        return this->x;
    }
    const int& operator()() const
    {
        cout << "A::operator() const" << endl;
        return this->x;
    }
};

struct B
{
    B(int x) {}
    A operator()()
    {
        cout << "B::operator()" << endl;
        return A();
    }
    const A operator()() const
    {
        cout << "B::operator() const" << endl;
        return A();
    }
};

int main() {
    
    B b1();
    b1();
    //
    B b2{};
    const B b3{};
    b2()() = 10;
    b3()() = 11;
    return 0;
}
```
What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
---
##### Questions 8

a. Why can't we write: `int& x;` ? Hint:
```cpp
int& x;
...
...
x = y;
```
b. Why can't we write:
```cpp
int z; 
int& const y = z;
```
c. Why this code works:
```cpp
int x = 4 , y = 5;
const int& z = x + y;
// Why can't we write int& z = x + y ?
```
---
##### Questions 9

```cpp
#include <iostream>
using namespace std;

struct A
{
  A(int x)
  {
      cout << "A::A(int x)" << endl; // 1
  }
  A(const A& a1)
  {
      cout << "A::A(const A& a1)" << endl; // 2
  }
  operator int()
  {
      cout << "A::operator int()" << endl; // 3
      return 5;
  }
};
int main() {
    
    A a1(10);
    a1 = 4;
    int y = a1 + 5;
    int z = 5 + a1;
    int w = 5 + z;
    return 0;
}
```
What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
---
##### Questions 10

``` cpp
#include <iostream>
using namespace std;

template <typename T>
struct A
{
    void f1();
};

template <typename T>
void A<T>::f1()
{
    cout << "A<T>::f1()" << endl;
}

template <>
void A<A<int>>::f1()
{
    cout << "A<A<int>>::f1()" << endl;
}

template <>
void A<int>::f1()
{
    cout << "A<int>::f1()" << endl;
}

int main() {
    
    A<A<int>> a1;
    a1.f1();
    return 0;
}
```
* What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
* When do we need to use _template specification_?
-----
##### Questions 11
```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v1(1);
    v1.push_back(0);
    v1.push_back(1);
    v1.push_back(2);
    v1[3]=3;
    return 0;
}
```
* What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
* How can we make this code safer?
* What is the difference between `vector<T>::push_back` , `vector<T>::emplace_back` , `vector<T>::insert` ?
-----
##### Questions 12
```cpp
vector<int> v1(4);
cout << *v1.data() << endl;

bool is_address_equal = ((void*)&v1 == (void*)v1.data());
cout << is_address_equal << endl;
```
What is the output after running the code? (The code compiles)

---
##### Questions 13
```cpp
vector<int> v1={1,2,3,4,5};
for(vector<int>::iterator start=v1.begin();start<v1.end();start++)
    if(*start % 2 == 0)
    {
        v1.erase(start);
    }
```
What is the output after running the code (The code compiles)?

-----------------------------------------------------------------
##### Questions 14
```cpp
list<int> l1;
//
// filling l1 with elements...
// ...
//
for (std::list<int>::iterator start=l1.begin(); start != l1.end(); start++)
    if(*start % 2 == 0)
    {
        l1.remove(*start);
    }
```
What is the output after running the code (The code compiles)?

-----------------------------------------------------------------
##### Questions 15
```cpp
int number;
bool b = false;
while(!b)
{
    cin.clear();
    b = (cin >> number); 
}
```
What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
a
```
---
##### Questions 16
```cpp
#include <iostream>
#include <vector>
using namespace std;

class A
{
    vector<int> v1(100);
    public:
        virtual void f1()
        {
            cout << "A::f1" << endl;
        }
};
class B : public A
{
    vector<int> v2(200);
    
    public:
        void f1()
        {
            cout << "B::f1" << endl;
        }
};

int main() {
    A* a1=new B;
    a1->f1();
    delete a1;
    
    return 0;
}
```
* What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
* Is there a memory leak? Explain.
----

##### Questions 17
```cpp
class A
{
    public:
        virtual void f1()
        {
            cout << "A::f1" << endl;
        }
        void f2()
        {
            this->f1();
        }
        virtual ~A()
        {
            cout << "A::~A" << endl;
        }
};
class B : public A
{
    public:
        virtual void f1()
        {
            cout << "B::f1" << endl;
        }
};

int main() {
    A* a1=new B;
    a1->f2()
    delete a1; 
       
    return 0;
}
```
What is the output after running the following commands:
```
g++ -std=c++11 -c main.cpp
g++ main.o
./a.out
```
---

##### Questions 18
```cpp
struct A
{
    int x;
    virtual void f1() {}
    void g1() {}
    void h1() {}
    ~A() {}
};
struct B : public A
{
    virtual void f1(int x) {}
    void f1() {}
    void g1() {}
    virtual ~B() {}
};
struct C : public B
{
    void f1() {}
    virtual void g1() {}
    virtual void h1() {}
    virtual ~C() {}
};
struct D : public C
{
    virtual void h1() {}
    virtual ~D() {}
};
```
* Build virtual tables for `A` , `B` , `C` , `D`.
* Calculate the size of each class.

____________

<br><br><br><br><br><br><br>

# Additional material

### Iterators

`i` - iterator , `x` - variable , `index` - integer variable

|             |    output|      input    |       forward  |   bi-directional|    random                     |
| ----------- | ---------| --------------| ---------------| ----------------| ------------------------------|
| Read        |          |      `x=*i`   |       `x=*i`   |   `x=*i`        |    `x=*i`,`x=i[index]`        |
| Write       |    `*i=x`|               |       `*i=x`   |   `*i=x`        |    `*i=x`,`i[index]=x`        |
| direction   |    `++`  |      `++`     |       `++`     |   `++`,`--`     |    `++`,`--`,`+`,`-`,`+=`,`-=`|
| Comparsion  |          |      `==`,`!=`|       `==`,`!=`|   `==`,`!=`     |    `==`,`!=`,`<`,`>`,`<=`,`>=`|
