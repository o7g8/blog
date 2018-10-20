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

### C++ Resources

<https://en.cppreference.com>

<http://www.cplusplus.com/reference/>

<http://eel.is/c++draft/> - draft of C++ standard.

<https://www.ideone.com/> - online C++ compiler.

### C++ features support by compilers

[C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)

[MSVC++: Support For C++11/14/17 Features (Modern C++)](https://msdn.microsoft.com/en-us/library/hh567368.aspx)

### GNU Compiler and Library References

[Using the GNU Compiler Collection (GCC)](https://gcc.gnu.org/onlinedocs/gcc/)

[The GNU C++ Library](https://gcc.gnu.org/onlinedocs/libstdc++/)

[GNU: Options Controlling C++ Dialect](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html)

### preprocessor and predefined compiler-specific macros

[MSVC/MSVC++ Predefined Macros](https://docs.microsoft.com/en-us/cpp/preprocessor/predefined-macros?view=vs-2017)

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

* Error `‘DBL_MAX’ was not declared in this scope`. See <https://stackoverflow.com/questions/23278930/what-is-dbl-max-in-c>

* Error `error: ‘x’ was not declared in this scope`. It can point to many things, but if you are in a template class, then look if the `x` is declared in the parent class. In this case the fix is `this->x`.

* Error `‘T’ does not name a type`. See <https://stackoverflow.com/questions/3608305/class-name-does-not-name-a-type-in-c>

* Error `declaration does not declare anything`:

```cpp

aa::bb::cc;

```

Surprisingly the line, which doesn't make any sense, is processed by VS2013 compiler.

* Error `invalid use of incomplete type 'T'`. See <https://stackoverflow.com/questions/20013901/im-getting-an-error-invalid-use-of-incomplete-type-class-map>

* Error: `invalid initialization of non-const reference of type ‘T&’ from an rvalue of type ‘T’`. Fixed with turning the reference into a `const` reference>.

```cpp

X(MyType& arg = MyType()) {
    ...
}
// to
X(const MyType& arg = MyType()) {
    ...
}

```

* Error: `invalid use of‘::’`. Obvious from the context.

* Error: `qualified name does not name a class before ‘{’ token`. The definitions like `class ns1::ns2::ClassName` are not accepted by GNU. And explicit namespace declaration is needed.

* Error: `‘stdext’ has not been declared`. Indicates usage of MS-specific `stdext::checked_array_iterator`.  See <https://stackoverflow.com/questions/25716841/checked-array-iteratort-in-c11> and <https://docs.microsoft.com/en-us/cpp/standard-library/checked-array-iterator-class?view=vs-2017>

* Error: `explicit specialization in non-namespace scope ‘class ns:T’`. A MS-specific issue. See <https://stackoverflow.com/questions/5777236/gcc-error-explicit-specialization-in-non-namespace-scope/5777264> and <https://stackoverflow.com/questions/1723537/template-specialization-of-a-single-method-from-a-templated-class>

* Error: `‘strcpy’ is nota member of ‘std’`. Include `<cstring>`.

* Error: `taking address of temporary`. See <https://stackoverflow.com/questions/16481490/error-taking-address-of-temporary-fpermissive>

* Error: `invalid initialization of non-const reference of type ‘T&’ from an rvalue of type ‘T’`. Use temp variable or make the function argument `const`.

* Error: `duplicate explicit instantiation of ‘class T’`.  The error occurs when `T1` is the same as `T2` in the instantiation below:

```cpp

template class A<T1>;
template class A<T2>;

```

Possible solutions:
use `#ifdef`, use `std::conditional` <https://stackoverflow.com/questions/13925730/conditional-explicit-template-instantiation>, or use C++17 `if constexpr()` (requires GCC7, VS2017 15.3) <https://tech.io/playgrounds/2205/7-features-of-c17-that-will-simplify-your-code/constexpr-if>

* Error: `no matching function for call to ‘ptr_fun(..)`. The best solution is to replace the obsolete `std::ptr_fun` with `std::function`.

* Error: `specialization of ``T`` in different namespace`. Move the specialization into the class' namespace. See <https://stackoverflow.com/questions/25594644/warning-specialization-of-template-in-different-namespace>.

* Error: `non-class, non-variable partial specialization ‘X’ is not allowed`. Needed to remove type parameters from the method definition:

```cpp

// from
template <T>
ReturnType<T> ns::method<T>(InputArg arg) {
    //...
}

// to
template <T>
ReturnType<T> ns::method(InputArg arg) {
    //...
}

```

* Error: `‘auto_ptr’ is not a member of ‘std’`. Convert to `unique_ptr`, see <https://www.bfilipek.com/2017/05/cpp17-details-fixes-deprecation.html> and <https://clang.llvm.org/extra/clang-tidy/checks/modernize-replace-auto-ptr.html>

* Change in zlib API, see <https://github.com/towardthesea/kh3ROS/blob/master/README~>

* Error: `‘strcpy_s’ was not declared in this scope`.

* Error: `‘memcpy_s’ was not declared in this scope`.

* Error: vectors with const elements, see <https://stackoverflow.com/questions/42273710/const-vector-reference-arguments-in-c>

## Linker error

Linking phase certifies the entire porting process.

* Linker error `multiple definition of ``T```. For specializations See <https://stackoverflow.com/questions/47544299/where-should-the-definition-of-an-explicit-specialization-of-a-class-template-be>

## SonarQube with C/C++

<https://github.com/SonarOpenCommunity/sonar-cxx/wiki/SonarQube-compatibility-matrix>

<https://github.com/SonarOpenCommunity/sonar-cxx/wiki/Installation>

<https://github.com/SonarOpenCommunity/sonar-cxx/wiki/Running-the-analysis>

<https://github.com/SonarOpenCommunity/sonar-cxx/tree/master/sonar-cxx-plugin/src/samples/SampleProject>