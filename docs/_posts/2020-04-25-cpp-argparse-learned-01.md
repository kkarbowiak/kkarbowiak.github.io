---
title: "What I learned implementing argparse in C++, pt. 1"
excerpt: "First installment"
last_modified_at: 2020-04-25
categories:
  - Programming
tags:
  - cpp-argparse
  - musings
---

# Look around

When I decided to implement Python's `argparse` in C++, I thought this was a
really great idea. I was so excited to be working on this, I almost pat myself
on the back for coming up with this :-)

What I did not think about was that other people surely thought of the same,
and there must be other projects with the same goal. When it finally occured to
me to do a search on Github for "cpp argparse" string, I saw that there were
plenty of results. Some projects dated back to 2014!

Luckily for me, my findings did not invalidate my efforts, at least not
entirely. Most of the projects were not header-only, were incomplete, or did not
offer the facilities of modern C++.

But honestly, even if I found something suitable to use as is, or something I
could contribute to, I would probably start my project nonetheless just to see
how much I could do.

# C++17, where art thou?

Where is C++17? Everywhere. Meaning that every major compiler[^1] supports it
fully. Does that mean that you can write modern C++17 code, depending on the
utilities, and compile it without problems on Linux and Windows? Unfortunately,
not.

It turns out that compiler's support for language standard is one thing, and the
completeness of the implementation of the standard library the compiler ships
with is another.

I was already bitten by this when I wanted to do 2016's edition of Advent of
Code in C++17. I wanted to utilise the newly added string to number conversions
available in `<charconv>` header in the form of `std::from_chars`. To my
dissapointment, GCC rejected my code stating that there was no such header file.

Back then I thought that it's OK. C++17 was not yet officially approved and the
compilers shipped a preview.

As it turns out, currently only Visual Studio 2019 ships with complete
implementation of `std::from_chars` and `std::to_chars` (go see Stephan T.
Lavavej's [great talk](https://www.youtube.com/watch?v=4P_kbF0EbZM) on the
subject). Both GCC and Clang lag behind offering no support for floating point
types.

It seems writing **portable code** no longer means relying on standard library
and avoiding compiler extensions. You need to use the *subset* of standard
library that is fully shipped by the compilers you will use.

[^1]: My list of major C++ compilers includes Clang, GCC, and MSVC.
