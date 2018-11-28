+++
title = "Lab notebook: Porting 550K LoC of C/C++ from Windows to Linux"
date = 2018-11-28T00:17:11Z
draft = false
tags = []
categories = ["C++", "C", "Porting", "MSVC", "GCC", "Linux"]
+++

As a part of Native Cloud transformation at my work I've ported a financial library from Windows to Linux. The library consists of 250K LoC of C and 300K LoC of C++. Some of the code is 20 years old. During the last 10 years it has been developed with MSVC++ exclusively for Windows. I wanted to port the code to GCC/Linux and keep the MSVC++/Windows build error-free in the same time.

About 70% of the porting efforts were spent on adaption of the legacy C++ code to C++14 standard. The porting took roughly 2 months.

Here is the summary of my learning from the project:

* Keep the log of faced errors and their solutions like this one. It's a huge time saver and helps answering code review questions as well.

* Patience and determination are the keys. Don't be discouraged by megabytes of compilation error log at the start. Progress will be very slow in the beginning, but then things start to speedup almost exponentially: in the first 3 weeks I made only 3% of progress, then it took only 2 days to go from 60% to 100%. The "exponential law" applies to both compilation and linking errors.

* Implement CI for the existing platform and for the new platform/compiler from very beginning. Keep your changes small, so it will be easy to revert the code to the last "good" state in case something goes wrong.

* Use CMake, it's quite convenient and it allows to track progress as well.

* Porting of Unicode-related code can be tricky.

* Don't cut corners with `-fpermissive`.

What I would do differently:

* Establish a build with clang to get more understandable build error messages for C++ code.   

## Docs and References

### C++ Resources

<https://en.cppreference.com>

<http://www.cplusplus.com/reference/>

[Draft of C++ standard](http://eel.is/c++draft/) 

[Online C++ compiler](https://www.ideone.com/)

### C++ features support by compilers

[C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support)

[MSVC++: Support For C++11/14/17 Features (Modern C++)](https://msdn.microsoft.com/en-us/library/hh567368.aspx)

### GNU Compiler and Library References

[Using the GNU Compiler Collection (GCC)](https://gcc.gnu.org/onlinedocs/gcc/)

[The GNU C++ Library](https://gcc.gnu.org/onlinedocs/libstdc++/)

[GNU: Options Controlling C++ Dialect](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html)

### Preprocessor and predefined compiler-specific macros

[Guide to predefined macros in C++ compilers (gcc, clang, msvc etc.)](https://blog.kowalczyk.info/article/j/guide-to-predefined-macros-in-c-compilers-gcc-clang-msvc-etc..html)

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

### Windows-specific types

[Windows Data Types](https://docs.microsoft.com/en-us/windows/desktop/winprog/windows-data-types)

[UNIX Compatibility](https://docs.microsoft.com/en-us/cpp/c-runtime-library/unix?view=vs-2017)

## Faced issues

* Error: `Unrecognized 'string_char_traits' as type argument of 'basic_string'`. See <https://gcc.gnu.org/onlinedocs/libstdc++/manual/strings.html>:

```cpp

// replace
typedef std::basic_string<char, string_char_traits<char>> mystring;
// with
typedef std::basic_string<char> mystring;

```

* Missing `min` and `max` can be declared as:

```c
#ifndef max
#define max(a, b) (((a) > (b)) ? (a) : (b))
#endif

#ifndef min
#define min(a, b) (((a) < (b)) ? (a) : (b))
#endif
```

* Error: `extra qualification 'xxx::' on member 'yyy...'`. See <https://stackoverflow.com/questions/11692806/error-extra-qualification-student-on-member-student-fpermissive>

* Error: `need ‘typename’ before ‘xxx::..iterator’ because ‘xxx’ is a dependent scope`. See <https://stackoverflow.com/questions/20866892/c-iterator-with-template>

* Locales: replace usages Windows-specific `_locale_t` with the standard facets of `std::locale`. More specifically if you do any kind of locale-dependent formatting or parsing, use `std::num_get<char>`, `std::num_put<char>`, `std::time_get<char>`, `std::time_put<char>`, etc. See more in [std::locale](https://en.cppreference.com/w/cpp/locale/locale).

* Default parameters in function definitions instead function declarations.

* Error: `there are no arguments to ‘X’ that depend on a template parameter, so a declaration of ‘X’ must be available`. See [Common Misunderstandings with GNU C++](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Using_the_GNU_Compiler_Collection/c---misunderstandings.html), also [No arguments that depend on a template parameter](http://www.agapow.net/programming/cplusplus/no-arguments-that-depend-on-a-template-parameter/)

* Missing `typename` here and there. Use `auto` where applicable, otherwise add the `typename`.

* Error `expected primary-expression before ‘T’`.

* Error `invalid use of template-name ‘X’ without an argument list`.

* Error `‘DBL_MAX’ was not declared in this scope`. See <https://stackoverflow.com/questions/23278930/what-is-dbl-max-in-c>

* Error `error: ‘x’ was not declared in this scope`. It can point to many things, but if you are in a template class, then look if the `x` is declared in the parent class. In this case the fix is `this->x`.

* Error `‘T’ does not name a type`. See <https://stackoverflow.com/questions/3608305/class-name-does-not-name-a-type-in-c>

* Error `declaration does not declare anything`:

```cpp

aa::bb::cc;

```

Surprisingly the line, which doesn't make any sense, is processed by VS2013 compiler. Delete it.

* Error `invalid use of incomplete type 'T'`. See <https://stackoverflow.com/questions/20013901/im-getting-an-error-invalid-use-of-incomplete-type-class-map>

* Error: `invalid initialization of non-const reference of type ‘T&’ from an rvalue of type ‘T’`. Turn the reference into a `const` reference:

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

* If your code uses outdated `zlib` API, see <https://github.com/towardthesea/kh3ROS/blob/master/README~>

* Error: `‘strcpy_s’ was not declared in this scope`. See <https://en.cppreference.com/w/c/string/byte/strcpy> and <https://msdn.microsoft.com/en-us/library/td1esda9.aspx>.

* Error: `‘memcpy_s’ was not declared in this scope`. See <https://en.cppreference.com/w/c/string/byte/memcpy> and <https://msdn.microsoft.com/en-us/library/wes2t00f.aspx>.

* Error: `vectors with const elements`. See <https://stackoverflow.com/questions/42273710/const-vector-reference-arguments-in-c>

* Error: `unknown type name ‘BOOL’`. See <https://docs.microsoft.com/en-us/windows/desktop/winprog/windows-data-types> and <https://stackoverflow.com/questions/8133074/error-unknown-type-name-bool/20743457>

* Warning: `implicit declaration of function ‘_getcwd’`. Not really a warning, but an error. See <https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/getcwd-wgetcwd?view=vs-2017> and <http://pubs.opengroup.org/onlinepubs/009695399/functions/getcwd.html>

* Error: `‘_MAX_PATH’ undeclared here (not in a function)` or the same error for `MAX_PATH`. See <https://stackoverflow.com/questions/9449241/where-is-path-max-defined-in-linux>.

```c
#include <linux/limits.h>
#define MAX_PATH PATH_MAX
#define _MAX_PATH PATH_MAX
```

* Warning (an error): `implicit declaration of function ‘_fpreset’`. See <https://msdn.microsoft.com/en-us/library/kfy34skx.aspx>, <https://stackoverflow.com/questions/2231504/why-and-when-should-one-call-fpreset>. Not portable, don't call it on Linux.

* Warning (an error): `implicit declaration of function ‘GetEnvironmentVariableA’`. A Windows-specific function <https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-getenvironmentvariable>. Use `getenv` <http://man7.org/linux/man-pages/man3/getenv.3.html>

* Warning (an error): `implicit declaration of function ‘GetCurrentProcessId’`. A Windows-specific function <https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-getcurrentprocessid>. Use `getpid` <http://man7.org/linux/man-pages/man2/getpid.2.html>

* Warning (an error): `implicit declaration of function ‘SetLastError’`. A Windows-specific function <https://msdn.microsoft.com/en-us/library/windows/desktop/ms680627(v=vs.85).aspx>. Use `errno` instead <http://man7.org/linux/man-pages/man3/errno.3.html>

* Warning (an error): `implicit declaration of function ‘GetLastError’`. A Windows-specific function <https://msdn.microsoft.com/en-us/library/windows/desktop/ms679360(v=vs.85).aspx>. Use `errno` instead <http://man7.org/linux/man-pages/man3/errno.3.html>

* Warning (an error): `implicit declaration of function ‘_fpclass’`. See <https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/fpclass-fpclassf?view=vs-2017> and <https://www.systutorials.com/docs/linux/man/3-fpclassify/>, <https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/fpclassify?view=vs-2017>

* Error: `‘INT_MAX’ undeclared (first use in this function)`. Add `#include <limits.h>`.

* Error: `‘errno’ undeclared(first use in this function)`. Add `#include <errno.h>`.

* Warning (an error): `implicit declaration of function ‘isspace’`. Add `#include <ctype.h>`.

### Unicode portability issues

If the code uses Windows-specific `tchar.h`, then look into <http://www.rensselaer.org/dept/cis/software/g77-mingw32/include/tchar.h> also read <https://www.tldp.org/HOWTO/Unicode-HOWTO-6.html>, <http://pubs.opengroup.org/onlinepubs/7908799/xsh/wchar.h.html>.

* Missing `TCHAR`:

```c

 typedef wchar_t TCHAR;

 ```

* Warning (an error): `implicit declaration of function ‘_T’`. It's Windows-specific macro, see <https://msdn.microsoft.com/en-us/library/dybsewaf.aspx> <https://social.msdn.microsoft.com/Forums/vstudio/en-US/8ce6ddef-3f1a-4033-a28b-54af91766e9f/teach-me-what-is-t?forum=vcgeneral>.

```c

#define _T(x) L##x

```

* Warning (an error): `implicit declaration of function ‘_tctime’`. See <https://msdn.microsoft.com/en-us/library/59w5xcdy.aspx>. Use `wcsftime` instead <http://pubs.opengroup.org/onlinepubs/7908799/xsh/wcsftime.html>. Exists also for Windows <https://msdn.microsoft.com/en-us/library/fe06s4ak.aspx>

* Warning (an error): `implicit declaration of function ‘_ftprintf’`. Use `fwprintf` instead. See <https://msdn.microsoft.com/en-us/library/xkh07fe2.aspx>.

* Error: `unknown type name ‘_TUCHAR’`. See <https://msdn.microsoft.com/en-us/library/c426s321.aspx>, use `wchar_t`.

## Linker errors

Linking phase certifies the entire porting process.

* Link order of static libraries is important. See <http://www.network-theory.co.uk/docs/gccintro/gccintro_18.html> and <https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking>

* Linker error `multiple definition of ``T```. For specializations See <https://stackoverflow.com/questions/47544299/where-should-the-definition-of-an-explicit-specialization-of-a-class-template-be>
