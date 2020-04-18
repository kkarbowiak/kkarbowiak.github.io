---
title: "The journey of cpp-argparse"
excerpt: "A summary of two years of development"
last_modified_at: 2020-04-18
categories:
  - Programming
tags:
  - cpp-argparse
---

Yes, it has been almost two years since I made the first commit to cpp-argparse
repository. Back then, it was still tracked in Mercurial and hosted on
BitBucket, which I soon changed to Git and GitHub combo.

## Motivation

Ever since I started programming in C++ I was creating some small command-line
utilities. All those programs had crippled support for argument parsing, reduced
to the bare minimum which was implementable without too much effort.

All the time I was wondering why didn't the standard library offer anything that
would make parsing the arguments easier.

New versions of the language were released (C++11/14/17) which offered a lot of
improvements, but none of those helped with the mundane task of converting the
old `argc` and `argv` duo into something more useful.

Over the years I mostly switched to using Python to write my small utilities.
I started using `argparse` and with every utility I wrote, in which parsing args
was a breeze, I liked it more and more.

At some point I decided I want the same kind of experience when writing code
in C++.

## Alternatives

I know that there are some third party libraries for parsing command line.

There is [Clara](https://github.com/catchorg/Clara) that started as part of the
Catch unit testing framework,
[docopt.cpp](https://github.com/docopt/docopt.cpp), or
[Boost.Program_options](https://www.boost.org/doc/libs/1_72_0/doc/html/program_options.html)
just to name a few.

Each has its own style and approach to creating the parsers. And each has its
own shortcomings.

## The start

Since the beginning I knew I wanted as much a hassle-free library as possible.
In C++ world, this mostly translates to header-only and no external
dependencies. I was pretty sure I would need an optional type and did not want
to code it myself, so I chose to rely on C++17, which amongst other goodies
provides `std::optional`.

I wanted to follow TDD approach as rigorously as I could. I decided to use
[doctest](https://github.com/onqtam/doctest) unit testing framework - a
departure from Catch which I often used in my previous projects.

I quickly switched to CMake as I wanted to build (at least) on Linux and
Windows, and did not feel like creating separate projects.

## Expectations

I set my expectations low. I did not want to aim at implementing the whole
functionality and then fail to deliver. Instead, I wanted to include the
bare minimum that still would be useful.

I did not create a feature list nor a roadmap, but I realised that handling of
positional and optional arguments, support for various argument types, automatic
help generation, and error handling all were a must. Everything else would be
an icing on the cake for me.

## The outcome

As of version 0.8.3, which is the latest release, `cpp-argparse` supports more
functionality that I hoped for (more in the
[previous post](https://kkarbowiak.github.io/docs/programming/cpp-argparse-083/)).

Still, there is more to come and 0.8.3 is definitely not the **last** release :-)

## Looking back

`cpp-argparse` did prove to be fun to work on and sucked me in almost
completely. During those two years of intermittent development I had lots of fun
and I learned quite a lot. But more on this in the next installment.
