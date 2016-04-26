---
layout: post
title: "Java Generics vs C++ Templates"
excerpt: "Musing on the difference between Java Generics and C++ Templates"
tags: [machine learning, academic]
comments: true
published: true
---

In a former life, I was an Engineer at
[SwiftKey](http://www.swiftkey.com), where I spent a lot of time
writing C++. These days, as a Data Scientist at
[Stratified Medical](http://www.stratifiedmedical.com), I don't write
much C++ but I do write and review a fair bit of Java code, and often
find myself musing on the differences between the two languages. I was
recently surprised by the behaviour of some Java generic code, and
felt it worth a short blog post. Consider the following,
similar-looking, pieces of code.

```c++
#include <iostream>

class A {
public:
  static int foo;
};

class B: public A {
public:
  static int foo;
};

class C: public A {
public:
  static int foo;
};

int A::foo = 1;
int B::foo = 10;
int C::foo = 100;

template<typename T>
static void f(T t) {
  std::cout << t.foo << std::endl;
}

int main() {
  B b = B();
  C c = C();

  f(b);
  f(c);
}
```

```java
class A {
    public static int foo = 1;
}

class B extends A {
    public static int foo = 10;
}

class C extends A {
    public static int foo = 100;
}

class Test {
    static <T extends A> void f(T t) {
        System.out.println(T.foo);
    }

    public static void main(String[] args) {
        B b = new B();
        C c = new C();

        f(b);
        f(c);
    }
}
```

If we run the first piece of code we see the following output:

```bash
> g++ generic.cpp
> ./a.out
10
100
```

However, if we run the second we get:

```bash
> javac Generic.java
> java Generic
1
1
```

Why is this the case? It's down to the difference between what happens
at compile-time for the all-powerful _C++ templates_, and their
slightly diluted Java counterparts, _Generics_.

# Java Generics

In the old, pre-generic, days of Java the only way to do generic
programming was to pass things around as instances of ```Object``` or
some abstract super-class, casting things up or down as
appropriate. The classic example would be collections, like ```List```
or ```Set```. These were implemented to store elements of type
```Object```, which could be cast down to the appropriate time on
retrieval, e.g.

```java
List l = new List();
l.add(new Integer(1));
Integer i = (Integer) l.get(0);
```

Of course, this kind of programming is not very nice, as we have no
guarantees that all the objects in ```l``` will actually be of type
```Integer```, which can lead to run-time errors. A solution, added in
Java 1.5, was provided by Java Generics. Now, we can write the above
code as

```java
List<Integer> l = new List<Integer>();
l.add(10);
Integer i = l.get(0);
```

Not only do we do away with the need for casting, but we now have
compile-time checks that ensure every object in ```l``` is of type
```Integer```. We can also declare generic methods, like the following

```java

class MathFunctions {
    public static <T extends Comparable<? super T>> T max(T obj1, T obj2) {
        return obj1.compareTo(obj2) <= 0 ? obj2 : obj1;
    }
}
```

# C++ Templates

Templates have been a part of C++ since the beginning. Indeed, the
[Standard Template Library](https://www.sgi.com/tech/stl/) is at the
very core of the language. Just like Java Generics, they allow us to write generic data structures

```c++
vector<int> l;
l.append(1);
int i = l[0];
```

And generic functions

```c++
template <class T> const T& max(const T& a, const T& b) {
    return a < b ? b : a;
}
```

# So, what's the difference?

The key difference between Java Generics and C++ Templates is how the
parameterisation of the different types is treated by the compiler.

In C++, the compiler generates separate machine code for every type a
template function is defined for, e.g., the following code will result
in machine code for two versions of ```std::max``` being generated,
one of type ```int``` and one of type ```float```

```c++
std::max<int>(1,2);
std::max<float>(3.0f, 1.0f);
```

On the other hand, the Java compiler generates bytecode for a single
function and handles the generics by *type erasure*. According to the
Java documentation type erasure is achieved by the following:

- Replace all type parameters in generic types with their bounds or
  ```Object``` if the type parameters are unbounded. The produced bytecode,
  therefore, contains only ordinary classes, interfaces, and methods.
- Insert type casts if necessary to preserve type safety.
- Generate bridge methods to preserve polymorphism in extended generic
  types.

This means that the implementation of a generic max function given
above would be equivalent to

```java

class MathFunctions {
    public static Comparable max(Comparable obj1, Comparable obj2) {
        return obj1.compareTo(obj2) <= 0 ? obj2 : obj1;
    }

    public static Float max(Float f1, Float f2) {
        return (Float) max(f1,f2);
    }

    public static Integer max(Integer n1, Integer n2) {
        return (Integer) max(f1,f2);
    }

    ...
}

```

So, while the compiler is aware of the parameterised types, and
generates appropriate code to allow the programmer to write "generic"
functions, at run-time this information is no longer present and the
executing programs uses the leftmost bound of each type parameter, or
```Object``` if no bound exists.

Now, we can see the reason for the seemingly surprisingly divergent
behaviour exhibited by the programs at the start of this blog post. In
the C++ code, each call to the function ```f``` executed a different
piece of code, one specialised to ```B``` and one specialised to
```C```. On the other hand, in the java program type erasure meant
that both calls to ```f``` executed the same function, parameterised
by the left-most bound ```A```.
