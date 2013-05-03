---
layout: post
title: "Mac OS Gets Cilk++ Support via Clang and LLVM"
date: 2013-03-24 17:35
comments: true
categories: [cpp, osx, llvm, clang, cilk, cilkplus, programming]
---

Part of my yet un-finished graduate work involved using a combination
of threading and cache-oblivious algorithms to improve the performance
of various linear algebra computations on sparse data sets.  During
the course of my work, I used a few concurrency models, including
Intel's [Cilk Plus](http://cilkplus.org) data and task parallelism
techniques.

Recently, the [Cilk Plus language extensions were introduced to the Clang frontend for LLVM](http://cilkplus.github.com).  Which mean it
was available on Darwin.  Though the [Cilk Plus extensions and runtime have been available in GCC](http://gcc.gnu.org/svn/gcc/branches/cilkplus/) for some time,
building the GCC code on Darwin proved to be non-trivial. (For me, at
least--- the Linux build was far easier.)  The introduction of Cilk
Plus in Clang allows for easy compilation and use on the Darwin
platform.

I used the instructions given on the new [Cilk Plus/LLVM GitHub site](http://cilkplus.github.com)
to try a copy of the compiler on my local machine.

The current implementation only provides support for a subset of the
Cilk Plus language extensions: `cilk_spawn`, `cilk_sync`, and
`hyperobjects`.  Leaving features like `cilk_for`, array notation and
SIMD work for future releases.  Despite the not providing a full
feature-set, the Cilk Plus/LLVM tools can already be used to make
non-trivial parallel programs.

I dusted off some benchmarking code I used for previous experiments
and gave it a go on my laptop.  While not a high-performance big-steel
machine, my laptop does weigh in well with 16GB of RAM and a quad-core
Intel i7.

The code I ran uses a well-known matrix multiplication algorithm---
[Strassen's algorithm](http://en.wikipedia.org/wiki/Strassen_algorithm)---to
create large parallel workloads.  The code was based on example code
shipped with the original 1996 [MIT Cilk language and runtime](http://supertech.csail.mit.edu/cilk/).  I modified the code
slightly to include the new language extensions as well as to provide
slightly more performance information. (I'll post the changes
shortly.)

I ran the code through a variety of benchmarks, which I've included
bellow.

![speedup](/images/speed-up.png "Speedup of Strassen's Algorithm with
 Cilk Plus/LLVM")

The legend labels describe the size of the matrices being used.  For
instance, 1024 mean that Strassen's Algorithm is being run using
inputs of size 1024 by 1024.  Notice we don't achieve linear
[speedup](http://en.wikipedia.org/wiki/Speedup), but we do get very
close.  I think with a little extra fine tuning (1) and the explicit
use of some SIMD instructions, near-linear speedup could be
achieved. If you want the data, you can get it
[here](/files/llvmcilk-results.xlsx).

1. Yes, I know: cache oblivious algorithms aren't *supposed* to
require tuning.  In practice, unfortunately, they do.  Maybe not as
much tuning as their cache aware brethren, but still the require some.
Even the original MIT code had some tuning code in it, so I'm not
alone in thinking it necessary.

