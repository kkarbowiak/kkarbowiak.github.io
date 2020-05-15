---
title: "What I learned implementing argparse in C++, pt. 2"
excerpt: "Second installment"
last_modified_at: 2020-05-15
categories:
  - Programming
tags:
  - cpp-argparse
  - musings
  - c++
---

Header-only libraries, such as `cpp-argparse`, `doctest`, or many parts of
`boost`, share a common ailment. Namely, all of their code is exposed to their
users, while some or even most of that code is still private to the library and
should not be used by client code.

In some cases, where code visibility would pose a security problem, the
libraries simply cannot be distributed as header-only. In other cases, the types
and functions not meant for outside use are often put into a nested namespace
like `detail`.

This shows the intent, but does not preclude client code from using and relying
on library's private implementation.

Can we do better? Can we **really** hide the types?

# Intermission

During this short break, think of answers to two questions below. I came across
them in a job interview some years ago.

 1. Can you create a type without a name?
 1. Can you create objects of that type?

# Using a type you cannot name

What do you think of the below code? Is it valid? Is it reasonable?

```c++
class Foo
{
    private:
        class InternalFactory
        {
            public:
                // ...
        };

    public:
        InternalFactory getFactory() const
        {
            return InternalFactory();
        }

        // ...
};
```

Turns out that what the `Foo::getFactory` function does (returning an instance
of the class' private nested type) is valid and quite useful:

```c++
const auto foo = Foo();
auto factory = foo.getFactory();
```

We can:
 * Use the `InternalFactory` type in the outside code as if it were a public
   nested type
 * Access all of its public members

We cannot:
 * Call it by name (it's **private**, duh), so we have to use `auto`

This gives some benefits:
 * No namespace pollution with types or sub-namespaces
 * Enforces users to use `auto` more ;-)

I used this unexpected language feature in `cpp-argparse` several times, for
example in `add_argument` function:

```c++
class ArgumentParser
{
    public:
        template<typename ..Args>
        decltype(auto) add_argument(Args&&... names)
        {
            return ArgumentBuilder(m_arguments, std::vector<std::string>{names...});
        }

    private:
        class ArgumentBuilder
        {
            // ...
        };
};
```

In the above code `ArgumentBuilder` is a private nested type. Its only purpose
is to create arguments using Named Parameter Idiom. The client code has no
business instantiating and using this type in any other context.

# Using a type you cannot yet name

All good textbooks start with general introduction to the topic they cover and
only later get down to more and more details. This is true even for books for
advanced audiences and on very specific topics.

I believe all good source code should do the same: present the big picture view
first, and only then delve into details.

This is why I never ever write a class like this:

```c++
class Foo
{
    int private_member;

    void private_function();

public:
    Foo();

    void bar() const;
    int baz();
};
```

To me the above class definition seems to be inside-out. Why do I get to know
class' private details, while I still have no idea of its interface?

The users of the class are primarily interested in its interface, on ways to
interact with objects of that type. A small subset of people that read the code
need or want to know the private details (the maintainers and the curious,
mostly).

Having this in mind, I always start with public section and then go to private
details for those interested, even if the cost is a few more key strokes:

```c++
class Foo
{
    public:
        Foo();

        void bar() const;
        int baz();

    private:
        void private_function();

        int private_member;
};
```

However, this notation poses a problem for nested types:

```c++
class Foo
{
    public:
        Bar getBar() const  // error: unknown type name 'Bar'
        {
            return Bar();
        }

    private:
        class Bar
        {
        };
}
```

Trying to fix this with trailing return type does not help:

```c++
class Foo
{
    public:
        auto getBar() -> Bar const  // error: unknown type name 'Bar'
        {
            return Bar();
        }

    private:
        class Bar
        {
        };
}
```

Due to C++'s strange parsing rules, using a class' nested type that is declared
below in a function declaration does not compile, while using it in the function
body is perfectly fine.

Fortunately, function return type deduction comes to the rescue:

```c++
class Foo
{
    public:
        auto getBar() const  // look Ma, no return type
        {
            return Bar();
        }

    private:
        class Bar
        {
        };
}
```

The compiler perfectly knows the returned type and does not pretend otherwise.

# The answer

Are you still here? Wondering on the answers to the two questions?

 1. Can you create a type without a name?

    Yes:
    ```c++
    struct
    {
        int a;
        int b;
    };
    ```

 1. Can you create objects of that type?

    Also yes:
    ```c++
    struct
    {
        int a;
        int b;
    } object;
    ```

In case you are wondering, I knew the answers. And I got the offer :-)
