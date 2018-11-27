+++
title = "Porting of a C++ library from MS VC++ to GNU"
date = 2018-09-25T11:27:38+02:00
draft = true
tags = []
categories = ["C++", "Porting", "MSVC++", "GCC"]
+++

> Assembly of Japanese bicycle require great peace of mind. (c) Robert Pirsig

Here I will describe things which I have learned during my work on porting a financial (non-modern)C++ library from MSVC++ to GNU.

## Docs and References

### Standard C++ Library References

<https://en.cppreference.com>

<http://www.cplusplus.com/reference/>

<http://eel.is/c++draft/>

### C++ features support by compilers

[C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)

[MSVC++: Support For C++11/14/17 Features (Modern C++)](https://msdn.microsoft.com/en-us/library/hh567368.aspx)

### GNU Compiler and Library References

[Using the GNU Compiler Collection (GCC)](https://gcc.gnu.org/onlinedocs/gcc/)

[The GNU C++ Library](https://gcc.gnu.org/onlinedocs/libstdc++/)

[GNU: Options Controlling C++ Dialect](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html)

[Guide to predefined macros in C++ compilers (gcc, clang, msvc etc.)](https://blog.kowalczyk.info/article/j/guide-to-predefined-macros-in-c-compilers-gcc-clang-msvc-etc..html)

[GNU: Common Predefined Macros](https://gcc.gnu.org/onlinedocs/cpp/Common-Predefined-Macros.html)

[C/C++ tip: How to list compiler predefined macros](http://nadeausoftware.com/articles/2011/12/c_c_tip_how_list_compiler_predefined_macros)

```bash

g++ -std=c++14 -dM -E -x c++ /dev/null | sort

```

Save temp files:

```bash

g++ -save-temps -c file.cc

```

See: <https://stackoverflow.com/questions/35537350/gcc-optional-preprocessor-output-and-compilation-in-one-pass>

[GNU: Preprocessor Output](https://gcc.gnu.org/onlinedocs/cpp/Preprocessor-Output.html)

## Issues

Ambition is to get everything compiled without using `-fpermissive`.

* Unrecognized `string_char_traits` as type argument of `basic_string`:

```cpp

// convert
typedef std::basic_string<char, string_char_traits<char>> scstring;
// to
typedef std::basic_string<char> scstring;

```

See <https://gcc.gnu.org/onlinedocs/libstdc++/manual/strings.html>

* Error: `extra qualification 'xxx::' on member 'yyy...'`. See <https://stackoverflow.com/questions/11692806/error-extra-qualification-student-on-member-student-fpermissive>

* Error: `need ‘typename’ before ‘xxx::..iterator’ because ‘xxx’ is a dependent scope`. See <https://stackoverflow.com/questions/20866892/c-iterator-with-template>

* Locales: replace usages Windows-specific `_locale_t` with the standard facets of `std::locale`. More specifically if you do any kind of locale-dependent formatting or parsing, use `std::num_get<char>`, `std::num_put<char>`, `std::time_get<char>`, `std::time_put<char>`, etc. See more in [std::locale](https://en.cppreference.com/w/cpp/locale/locale).

* Default parameters in function definitions instead function declarations.

* Override of `<cmath>` definitions required:

```cpp

#ifndef __GNUC__
#define _CMATH_
#else
#define _GLIBCXX_CMATH 1
#endif

```

*It's an extremely bad idea, because it relies on inner workings of the header!* And it collapses when you compile it with `c++14`.

* Global namespace pollution and override of standard functions:

<https://stackoverflow.com/questions/11085916/why-are-some-functions-in-cmath-not-in-the-std-namespace>

Thing which doesn't work (breaks once we get `#include <cmath>`):

```cpp

//#define _GLIBCXX_CMATH 1 // a hack which doesn't help either!!

// #include <cmath> // pollutes global namespace and doesn't work
#include <functional>
#include <iostream>

namespace principia {
    namespace shadow {
        #include <math.h>
    }

    namespace basutil {
        inline double SafeWrap(std::function<double()> const &f) {
            std::cout << "Hello\n";
            return f();
        }

        inline double const acos(double const x) {
			return SafeWrap([x](){return shadow::acos(x);});
		}
    }
}

using principia::basutil::acos;

int main(int argc, char** argv) {
    auto arg = 0.4;
    std::cout << principia::basutil::acos(arg) << std::endl;
    std::cout << acos(arg) << std::endl;
}

```

Thing which works only in case if there are no other redefinitions of `Acos()` :

```cpp

#include <cmath> // pollutes global namespace
#include <functional>
#include <iostream>

namespace principia {
    namespace basutil {
        inline double SafeWrap(std::function<double()> const &f) {
            std::cout << "Hello\n";
            return f();
        } 

        inline double const acos(double const x) {
			return SafeWrap([x](){return std::acos(x);});
		}
    }
}

const auto& Acos = principia::basutil::acos;
// using namespace principia::basutil; // won't work

int main(int argc, char** argv) {
    auto arg = 0.4;
    std::cout << principia::basutil::acos(arg) << std::endl;
    std::cout << Acos(arg) << std::endl;
    //std::cout << acos(arg) << std::endl; // won't work
}

```

It seems like the only solution is to be explicit about namespace:

```cpp

namespace safe = principia::basutil;
// then use safe::cos(..)

```

There are possible issues with macro which use the namespace-less invocations.

Another potential issue is that C++ math is used instead of C.

* Error: `there are no arguments to ‘X’ that depend on a template parameter, so a declaration of ‘X’ must be available`. See [Common Misunderstandings with GNU C++](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Using_the_GNU_Compiler_Collection/c---misunderstandings.html), also [No arguments that depend on a template parameter](http://www.agapow.net/programming/cplusplus/no-arguments-that-depend-on-a-template-parameter/)

* Missing `typename` here and there.

* Error `expected primary-expression before ‘T’`.

* Error `invalid use of template-name ‘X’ without an argument list`.
