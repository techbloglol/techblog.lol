+++
categories = []
date = "2015-09-05T23:57:10-07:00"
description = ""
keywords = []
title = "How to be bleeding edge – C++ proposals that'll change the way you code"

+++

In this article, I'll go over a few of the most recently implemented C++ features and extensions in GCC and Clang. Some might just change the face of your code forever. At the very least, demonstrating your bleeding edge knowledge in interviews will be sure to get you some points.

Initialized lambda captures
===========================

This is a C++14 feature that as far as I've been concerned means "capture by move" support. As a stickler for optimization, I hate unnecessary copies. With initialized lambda captures, I can be even more of a stickler:

```
std::vector<Things> largeVector;
// fill largeVector with lots of stuff
auto result = std::async([largeVector{std::move(largeVector)}] {
    // process largeVector
});
```

The proposal: [N3648](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3648.html)

Digit separator
===============

Here's another C++14 feature that you may find nice every now and then. For numeric literals, you can now insert a separator:

```
namespace {
    constexpr int kSecondsInAWeek = 604'800;
}
```

Unfortunately, this'll probably break your syntax highligher (as you can see above).

The proposal: [N3781](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3781.pdf)

Literal operator templates
==========================

With user defined literals, we can do some pretty handy things with strings at compile time. But the standard's missing something: A template form of the literal operator for characters and strings. So with the current standard, we can't change the type of the operator based on the contents of the string.

One use case is for a type-safe formatter:

```
template<typename CharT, CharT... String>
auto operator""_format() { return Formatter<String...>(); }

void foo() {
    auto str = "Hello %s!\n"_format("world"); // okay!
    auto str = "Hello %s!\n"_format(1); // compile time error!
}
```

In this example, the `_format` suffix can create an object with a call operator that takes arguments of the correct type. So if you mix up your arguments or format specifiers, you get a compile time error.


The proposal: [N3599](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3599.html)

Nested namespace definitions
============================

If you've worked on well-organized libraries, you probably had to maintain code like this in nearly every source file:

```
namespace mycompany {
namespace mylib {
namespace somecomponent {

// stuff

}}} // namespace mycompany::mylib::somecomponent
```

But if you're bleeding edge, now you can write this instead:

```
namespace mycompany::mylib::somecomponent {

// stuff

} // namespace mycompany::mylib::somecomponent
```

The proposal: [N4230](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4230.html)

Resources
=========

I've only included features that have implementations in GCC and Clang, but you can find a lot of interesting proposals under consideration in the [C++ Standard Evolution Active Issues List](http://cplusplus.github.io/EWG/ewg-active.html).

You can probably find some things you didn't know you could do in GCC's [C Language Extensions](https://gcc.gnu.org/onlinedocs/gcc/C-Extensions.html). Most (if not all) of those are available in Clang as well.

And [C++ Support in Clang](http://clang.llvm.org/cxx_status.html) is good to keep up with if you use Clang.