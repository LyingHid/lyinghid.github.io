---
layout: post
title:  "Access C++ Private Member"
date:   2020-05-11 +0800
categories: C++ Private argument-dependent-lookup template-instantiation
---

Recently, I was seeking to bypass the access restrictions of private members and found two posts. Some of the solutions are straight forward, the others are mind blowing.

The first part are stratigies shown in [post 1](http://www.gotw.ca/gotw/076.htm) with the following example header file:

```C++
// File x.h

class X
{
public:
  X() : private_(1) { /*...*/ }

  template<class T>
  void f( const T& t ) { /*...*/ }

private:
  int private_;
};
```

# Copy class definition and add a friend to it

```C++
class X
{
  // instead of including x.h, manually duplicates X's
  // definition, and adds a line like:
  friend ::Hijack( X& );
};

void Hijack( X& x )
{
  x.private_ = 2; // evil laughter here
}
```

This is illegal because it violates the One Definition Rule, which says that if a type (here X) is defined more than once, the definitions must be identical.

# Define private public

```C++
#define private public // illegal
#include "x.h"

void Hijack( X &x )
{
  x.private_ = 2; // evil laughter here
}
```

Again:
1. It is illegal to define a reserved word;
2. It violates the One Definition Rule, as above.

# Create a twin class with all member public

```C++
class BaitAndSwitch {
// hopefully has the same data layout as X
// so we can pass him off as one
public:
  int notSoPrivate;
};

void f( X &x )
{
  // evil laughter here
  (reinterpret_cast<BaitAndSwitch &>(x)).notSoPrivate = 2;
}
```

However:
1. The object layouts of X and BaitAndSwitch are not guaranteed to be the same, although in practice they probably always will be;
2. The results of the reinterpret_cast are undefined, although most compilers will let you try to use the resulting reference in the way the hacker intended.

All above suffer from one problem as we access private members by defining a new class and the memory layout can be different between our new class and the original one.

But we do have legal methods granted by C++ standard.

# Template specialization

As the class has a template method, we can define the following method to bypass access restriction.

```C++
namespace
{
  struct Y {};
}

template<>
void X::f( const Y & )
{
  private_ = 2; // evil laughter here
}

void Test()
{
  X x;
  cout << x.Value() << endl; // prints 1
  x.f( Y() );
  cout << x.Value() << endl; // prints 2
}
```

# Class member pointer and template instantiation

It would be nice if we can have a class member pointer of the private member. That’s possible when we do template instantiation and pass the private member address to a template parameter as described [here](https://en.cppreference.com/w/cpp/language/class_template).

Here is the demo by [post 2](https://bloglitb.blogspot.com/)

```C++
template<typename Tag, typename Tag::type M>
struct Rob {
  friend typename Tag::type get(Tag) {
    return M;                                              (1)
  }
};

// use
struct A {
  A(int a):a(a) { }
private:
  int a;
};

// tag used to access A::a
struct A_f {
  typedef int A::*type;
  friend type get(A_f);                                    (2)
};

template struct Rob<A_f, &A::a>;                           (3)

int main() {
  A a(42);
  std::cout << "proof: " << a.*get(A_f()) << std::endl;    (4)
}
```

At line (3), the private member’s address is passed to the template and will be returned in the function at line (1). At line (2), the friend declaration matches the function defined at line (1), which will be found during argument dependent lookup when line (4) is executed.

At line (4), function `get` is invoked. The compiler will search it in two ways:
1. Normal lookup: from the scope it is called all the way to the global namespace;
2. [Argument Dependent Lookup](https://abseil.io/tips/49): the compiler searches namespace where `struct A_f` is used.

There is one and only one `get` function defined in `struct Rob`. By defining a friend function like line (1), it’s invisible to the normal lookup as explained [here](https://stackoverflow.com/a/8284809), but will be found by argument dependent lookup according to [this](https://stackoverflow.com/a/51304760), so that function `get` won't pollute the global namespace. After `get` is executed, it returned the member pointer of private member. By applying the pointer to class instance `a`, we finally got the value of its private member.
