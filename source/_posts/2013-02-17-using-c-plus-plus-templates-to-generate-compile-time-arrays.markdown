---
layout: post
title: "Using C++ templates to generate compile-time arrays"
date: 2013-02-17 09:22
comments: true
categories: [cpp,templates,trick,code]
---

There was huge amount of attention payed to template meta-programming
in C++ within recent history.  The idea was to let the compiler do
some additional work upfront so that the runtime environment would be
then forever relived of the need to fill the same role.

The goto example in Computer Science for recursion is to list the
first *n* [Fibonacci](http://en.wikipedia.org/wiki/Fibonacci_number)
numbers. It's easy to see why this is done, the code is very simple
and highly readable:

``` cpp
int fib(int n)
{
  if (n <= 0) {
    return 0;
  } else if (n == 1 || n == 2) {
    return 1;
  } else {
    return fib(n-1) + fib(n-2);
  }
}
```

Of course, we would never actually do this in practice--since it
inefficient--but it does nicely illustrate the beauty of simplicity of
recursion.  It's fitting then, that we try using the same example to
use the same function to illustrate compile-time array generation.

Note that `gcc`/`clang` users will not be able to take advantage of
this, as it seems to only be a "feature" of the Microsoft C++
compiler.  It's also non-standard, so it's unlikely to work at any
point in the future.


``` cpp
template<class T, int n>
struct fibonacci_numbers { 
    static const T end = fibonacci_numbers<T, n-1>::end
        + fibonacci_numbers<T, n-2>::end;
};

template<class T> struct fibonacci_numbers<T, 2> { static const T end = 1; };
template<class T> struct fibonacci_numbers<T, 1> { static const T end = 1; };
template<class T> struct fibonacci_numbers<T, 0> { static const T end = 0; };
```

The code uses a few tricks to accomplish it's job: We use partial
specialization to help define the base case of the recursion.  This
can be seen in the last three lines, where the numbers listed
coincide with the conditions in the original code.

The more surprising property we take advantage of is that the MS C++
compiler allocates storage in the stack for the `end` member of
`fibonacci_numbers`.  Thus, since we are recursively defining
instances of the `fibonacci_numbers` structure, the compiler allocates
stack storage for each definition.

Once compiled, accessing the data must be done in reverse, due to the
nature of the template recursion.  The following loop prints the first
12 Fibonacci numbers:

```
int 
main () 
{ 
  fibonacci_numbers<int, 12> fibonacci;
  for (int i = n; i >= 0; i--) {
    cout << (n-i) << (*((&fibonacci)-i)).end << "\n";
  }
  return 0;
}
```

It's not pretty, but it is effective.  It should be easy to write an
iterator wrapper for the above code, but I'll leave that for a future
post or update.
