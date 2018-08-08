---
title: Inko compared to other languages
---

## Table of contents
{:.no_toc}

1. TOC
{:toc}

## Introduction

Inko shares similaries with a variety of other languages. This can lead one to
wonder, what are the differences? This guide provides an unbiased overview of
the differences between Inko and several other programming languages.

## Comparing with Go

[Go](https://golang.org/) is a statically typed programming language, primarily
developed by Google. Go sits between systems programming languages such as
[Rust](https://www.rust-lang.org/en-US/) and C, and high level languages such as
[Python](https://www.python.org/).

### Execution model

Go is a compiled language, whereas Inko is an interpreted language. In practise
this means Go executables are easier to distribute, as you do not need to
distribute both your code and a virtual machine.

### Paradigm

Go is a multi-paradigm language, though the authors of Inko argue it's primarily
a procedural programming language. This is because of it's heavy focus on
directly calling functions, instead of relying on message passing (although this
is possible in Go, at least to a certain extent).

Inko on the other hand is an object-oriented programming language, using message
passing instead of directly invoking functions.

### Multitasking

Go uses lightweight tasks called "goroutines", which are basically green
threads. Inko uses lightweight processes, which are closer to OS processes, as
each lightweight processes is fully isolated.

### Memory

Goroutines use shared memory, whereas Inko processes have their own heaps.

Go's garbage collector is a concurrent mark & sweep garbage collector,
priorising low pause timings over application throughput. To the best of our
knowledge, the Go garbage collector may still suspend all goroutines in certain
cases (known as a "stop-the-world" phase).

Inko's garbage collector is a parallel generational garbage collector, based on
[Immix][immix]. The Inko garbage collector only suspends the process that is
being garbage collected, but it is suspended for the _entire_ duration of the
garbage collection cycle. Inko's garbage collector doesn't focus on one specific
area (e.g. low pause timings), instead it tries to provide a healthy balance
between low pause timings and application throughput.

### Scheduler

Go's scheduler is partially preemptive. This means that Go _can_ suspend
goroutines and have others run in their place, but only when certain conditions
are met. Inko's scheduler on the other hand is fully preemptive, meaning every
process is guaranteed a certain amount of execution time, no matter what code it
is running.

## Comparing with Erlang and Elixir

[Erlang](http://www.erlang.org/) and [Elixir](https://elixir-lang.org/) are two
different languages running on
[BEAM](https://en.wikipedia.org/wiki/BEAM_(Erlang_virtual_machine)). Since both
languages are running on the same virtual machine, we'll treat them as one.

### Execution model

Erlang, Elixir, and Inko are all interpreted languages. All three compile source
code into bytecode, which is then executed. Inko draws a lot of inspiration from
Erlang and Elixir.

### Paradigm

Erlang and Elixir are functional programming languages, while Inko is
object-oriented.

### Multitasking

Erlang, Elixir, and Inko all use a similar multitasking model: lightweight
processes.

### Memory

Erlang and Elixir use a combination of process-local memory, and reference
counted memory. Reference counting is typically used for larger objects, making
it cheaper to send them to other processes, at the cost of having to perform
reference counting.

Inko uses tracing garbage collection, although some internal data structures use
reference counting _in addition_ to tracing garbage collection. For example,
strings are reference counted to make it cheaper to send them to processes.
These reference counts are managed by the garbage collector, and typically only
modified when copying such an object or when it is garbage collected.

### Scheduler

All three use a similar scheduling setup: a number of threads perform work using
work stealing, and processes can be suspended whenever the scheduler decides
this is necessary. This is not surprising, as Inko's scheduling mechanism is
inspired by Erlang and Elixir.

## Comparing with Ruby

[Ruby](https://www.ruby-lang.org/en/) is an interpreted object-oriented
programming language, typically used for building web services such as Basecamp
and GitLab, although you can also use it for a wide variety of other tasks.

### Execution model

Both Ruby and Inko are interpreted, and execute bytecode. Ruby does not persist
the bytecode after compilation, instead it is directly executed. This means that
every time your program runs, the bytecode has to be compiled from scratch.

Inko's compiler is a separate program, and bytecode is saved to disk.
Incremental compilation is currently not supported, but will likely be added in
the future.

### Paradigm

Both Ruby and Inko are object-oriented languages. Inko takes things a few steps
further by using message passing for almost everything, including statements
such as `if` and `while`.

### Multitasking

In Ruby you can use OS threads, fibers (coroutines), and OS processes. There are
no high level structures such as thread pools, or work stealing schedulers.

In Inko you can only use lightweight processes, and the virtual machine takes
care of scheduling and running these in the best way possible.

The main Ruby implementation (MRI, also known as CRuby) uses a "Global
Interpreter Lock" (GIL), preventing Ruby threads from running in parallel,
except for a few cases. Inko has no such lock, allowing you to run many
processes in parallel.

### Memory

Ruby uses shared memory, whereas in Inko all processes have their own isolated
heap.

Ruby uses a generational, incremental, mark & sweep garbage collector that will
suspend all threads when running. The garbage collector is not parallel, meaning
only a single thread is used to perform garbage collection.

Inko uses a parallel generational garbage collector, and only suspends the
process that is being garbage collected.

### Scheduler

As Ruby uses OS threads for multitasking, it relies on the OS thread scheduler.
This means an OS thread will typically run until it is garbage collected or
terminates. This means it's possible for a few OS threads to consume all CPU
time, preventing other threads from performing their work.

Inko uses its own preemptive scheduler, and guarantees that every process is
given a fair share of execution time.

## Comparing with Pony

[Pony](https://www.ponylang.org/) is an object-oriented programming language
built on the actor model. Pony uses "capabilities" to make certain operations
secure. The definition of a "capability" described in ["Chapter 4:
Capabilities"](https://tutorial.ponylang.org/capabilities/) of the Pony
tutorial.

### Execution model

Pony is a compiled language using LLVM, whereas Inko is an interpreted language.
Both Inko and Pony use a separate program for compilation, inkoc and ponyc
respectively.

### Paradigm

Both Pony and Inko are object-oriented programming languages.

### Multitasking

Pony uses actors, which are defined similar to objects in Inko. This is similar
to Inko processes, though the way you define them differs a bit. To the best of
our knowledge, Pony requires you to define your actor before you can use it,
whereas in Inko you can spawn a process at any point in time.

### Memory

To the best of our knowledge, Pony uses separate heaps for actors, although we
haven't been able to confirm this. Memory is managed using a garbage collector,
although garbage collection does not run while an actor is performing a
"behaviour". A behaviour is basically an asynchronous method call. While this
may result in higher application throughput, it can also easily lead to an actor
exhausting memory.

Pony's advice for dealing with this appears to come down to "Just don't do it".
For example, from the [Garbage collection
guide](https://tutorial.ponylang.org/gotchas/garbage-collection.html):

> Long loops in behaviors are a good way to exhaust memory. Don't do it. If you
> want to execute something in such a fashion, use a Timer.

Inko uses a separate heap for every process, and garbage collection _can_ occur
while the process is performing work. The impact of this on application
throughput should be minimal, as most (large) processes won't be suspended for
more than roughly 5 milliseconds per garbage collection cycle.

### Scheduler

Pony's scheduler is not preemptive, meaning an actor will continue to run until
it yields control back to the scheduler. This means an infinite loop will
prevent the thread running the actor from doing any other work.

Inko's scheduler is preemptive, meaning every process is given a fair share of
execution time. As a result, long running code such as infinite loops will never
prevent a thread from doing other work indefinitely.

[immix]: http://www.cs.utexas.edu/users/speedway/DaCapo/papers/immix-pldi-2008.pdf
