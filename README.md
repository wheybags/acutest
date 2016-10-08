# CUTest Readme

Home: http://github.com/mity/cutest


## What Is CUTest

CUTest means "C Unit Testing" and it is intended to do exactly that, while
being as simple as possible to use it, not to stand in the developer's way
and minimize any external dependencies.

To achieve that, the complete implementation resides in a single C header file,
`"cutest.h"`, and its core depends only on few standard C library functions.

CUTest supports C as well as C++, and it can deal with unit tests which throw
C++ exceptions.


## Word of Disambiguation

Originally, I was so proud to find out the name Cutest for this little project.
However, as I have recently learned, there are so many more projects of the
same name, doing the same way:

 * http://cutest.sourceforge.net/
   (with many mirrors of it on github, likely the 1st one having the name)
 * https://github.com/rafael-santiago/cutest
 * https://github.com/yubako/cutest
 * https://github.com/m-pilia/cUTest
 * https://github.com/djanowski/cutest (unit tests for Ruby)
 * https://github.com/InfernalLogic/cutest
 * https://ccpforge.cse.rl.ac.uk/gf/project/cutest/wiki
 * https://github.com/aiobofh/cutest

(and likely even more.)

To deal with it, the [issue #6](https://github.com/mity/cutest/issues/6) has
been created.

In the mean time, please at least be careful what particular Cutest is meant
when it is being talked about.


## Overview

**Main features:**
* Unit tests in C or C++ are supported.
* No need to install/setup/configure testing framework. CUTest is just single
  C/C++ header, `"cutest.h"`.
* The header provides program entry point (function `main()`).
* Minimal dependencies: Core features only depend on few standard C headers,
  optional features may use more if available on the particular system.
* Trivial interface for writing unit tests: Few preprocessor macros described
  further below.

**Windows specific features:**
* By default, every unit test is executed as a child process.
* By default, if the output is directed to a terminal, the output is colorized.
* CUTest installs a SEH filter to print out uncaught SEH exceptions.

**Unix specific features:**
* By default, every unit test is executed as a child process.
* By default, if the output is directed to a terminal, the output is colorized.

**C++ specific features:**
* CUTest catches C++ exceptions thrown from unit test functions. Such unit
  tests are considered to fail.
* If the exception is derived from `std::exception`, `what()` is written out
  in the error message.

Any C/C++ module implementing one or more unit tests and including `"cutest.h"`,
can be built as a standalone program. We call the resulted binary as a "test
suite" for purposes of this document. The suite is then executed to run the
tests, as specified with its command line options.

By default, all unit tests in the program are run and (on Windows and Unix)
every unit test is executed in a context of its own subprocess. Both can be
overridden on the command line.

We say any unit test succeeds if all conditions (preprocessor macros `TEST_CHECK`
or `TEST_CHECK_`) called throughout its execution pass, the test does not throw
an exception (C++ only), and (on Windows/Unix) the unit test subprocess is not
interrupted/terminated (e.g. by a signal on Unix or SEH on Windows).

Exit code of the test suite is 0 if all unit tests pass, 1 if any of them fails,
or other number if an internal error occurs.


## Writing Unit Tests

To use CUTest, simply include the header file `"cutest.h"` on the beginning of
the C/C++ source file implementing one or more unit tests. Note the header
provides implementation of the `main()` function.

```C
#include "cutest.h"
```

Every test is supposed to be implemented as a function with the following
prototype:

```C
void test_example(void);
```

The tests can use preprocessor macro `TEST_CHECK` or `TEST_CHECK_` to validate the
test conditions. They can be used multiple times, and if any of those conditions
fails, the particular test is considered to fail.

(The macro `TEST_CHECK_` can be used only if your C preprocessor supports variadic
macros: it takes printf-like extra arguments to provide a custom error message
if the condition fails.)

For example:

```C
void test_example(void)
{
    void* mem;
    int a, b;

    mem = malloc(10);
    TEST_CHECK(mem != NULL);

    mem = realloc(mem, 20);
    TEST_CHECK(mem != NULL);

    a = 1;
    b = 2;
    TEST_CHECK_(a + b == 3, "Expected %d, got %d", 3, a + b);
}
```

Note that the tests should be independent on each other. Whenever the test
suite is invoked, the user may run any number of tests in the suite, in any
order. Furthermore by default, on platforms where supported, each unit test
is executed as a standalone (sub)process.

Finally, the test suite source file has to list the unit tests, using the
macro `TEST_LIST`. The list specifies name of each test (it has to be unique)
and pointer to a function implementing the test. I recommend names which are
easy to use on command line, i.e. especially avoid space and other special
characters in them. Also avoid using dash as a first character, as it would
be then interpreted as a command line option, not a test name.

```C
TEST_LIST = {
   { "example", test_example },
   ...
   { 0 }
};
```

Note the test list has to be ended with zeroed record.

Finally you just compile the C/C++ test suite source file as a simple program.
For example, assuming `cc` is your C compiler:

```sh
$ cc test_example.c -o test_example
```

More comprehensive description of API can be found in comments in the header
`"cutest.h"`.


## Running Unit Tests

When the C file with the tests is compiled, the resulted testing binary can be
used to run the tests.

By default (without any command line options), it runs all implemented unit
tests. It can also run only subset of the unit tests as specified on the
command line:

```sh
$ ./test_example                # Run all tests in the suite
$ ./test_example test1 test2    # Run only tests "test1" and "test2"
$ ./test_example --skip test3   # Run all tests with the exception of "test3"
```

To see all the options, simply run the binary with the option --help.

```sh
$ ./test_example --help
```


## License

CUTest is covered with MIT license, see the file `LICENSE.md` or beginning of
`cutest.h` for its full text.


## More Information

The project resides on github:

* http://github.com/mity/cutest

You can find the latest version of CUTest.h there, contribute with enhancements
or report bugs.
