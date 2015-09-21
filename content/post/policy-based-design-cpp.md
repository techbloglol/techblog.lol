+++

title = "Policy based design in C++"
author = "vmrob"
draft = false
date = "2015-09-14T01:22:06-06:00"
tags = ["c++", "code", "design patterns"]
description = "Policy-based design is a compile-time variant of the strategy pattern using C++ templates. Learn what policies are, how they're used, and how they can decouple classes to make complex code modular and testable."

+++

[Policy-based design](https://en.wikipedia.org/wiki/Policy-based_design) makes use of template parameters to change the inheritance hierarchy of a class for the sake of changing specific runtime behaviors. While similar to the strategy pattern, C++ templates are compile-time constructs and benefit from all of the optimizations the compiler back-end provides.

As an example, the program below, compiled with full optimizations, will result in what is essentially a single call to `printf` with the arguments `%s` and `foo`.

```C++
struct foo {
    static const char* print() { return "foo"; }
};

template <typename policy>
struct printer : private policy {
    using policy::print;
};

#include <cstdio>

int main() {
    printer<foo> p1;
    printf("%s\n", p1.print()); // foo
}
```

Additionally, policy classes that have no data members (sometimes referred to as stateless classes), can benefit from the [empty base class](http://en.cppreference.com/w/cpp/language/ebo) optimization which eliminates the memory overhead of your stateless policies.

```C++
template <typename policy>
struct printer : private policy {
    using policy::print;
};

struct stateless {
    static const char* print() { return "stateless"; }
};

struct stateful {
    const char* s = "stateful";
    const char* print() { return s; }
};

#include <cstdio>

int main() {
    printer<stateless> p1;
    printer<stateful> p2;

    printf("%lu\n", sizeof(p1)); // 1
    printf("%lu\n", sizeof(p2)); // 8
}
```

What might be the strongest benefit to this design is that it greatly simplifies unit tests. The ability to swap in mock-policies and test subclasses in the absence of the parent class complexities is really what this is all about.

```C++
#include <cassert>

struct complex_algorithm {
    static size_t calculate() { /* complex logic */ return 0; }
};

template <typename policy>
struct foo : private policy {
    using policy::calculate;
};

// Later in the test framework

struct mock_algorithm {
    static size_t expected;
    static size_t calculate() { return expected; }
};

size_t mock_algorithm::expected = 0;

void foo_test() {
    mock_algorithm::expected = rand();

    foo<mock_algorithm> f;

    // verify that calculate returns the policy's calculated result
    assert(f.calculate() == mock_algorithm::expected);
}

void complex_algorithm_test() {
    // testing for the complex algorithm
}

int main() {
    foo_test();
    complex_algorithm_test();
}
```

Even if you have already been doing something similar in C++, hopefully you'll find this useful next time you need to stitch together complex objects and still need to be able to test each component in isolation.
