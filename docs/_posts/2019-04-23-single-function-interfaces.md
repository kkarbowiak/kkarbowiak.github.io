---
title: "Single-function interfaces considered harmful"
excerpt: "...or overkill, or unnecessary complexity, at least in C++"
last_modified_at: 2019-04-23
categories:
  - Programming
tags:
  - OOD
  - inheritance
  - C++
---

Have you ever seen an interface class with just a single function meant to be overridden in a derived class? Something like this:

```c++
class IInterface
{
    public:
        virtual void doSomething() = 0;
};
```

Or more properly and common like this:

```c++
class IInterface
{
    public:
        virtual ~IInterface() = default;

        virtual void doSomething() = 0;
};
```

which includes the virtual destructor, which ensures polymorphic deletion and prevents from potential memory leaks, or (far less common) like this:

```c++
class IInterface
{
    public:
        virtual void doSomething() = 0;

    protected:
        ~IInterface() = default;
};
```

which includes a protected non-virtual destructor, which prevents from polymorphic deletion and saves one entry in the v-table.

The above three definitions already foreshadow the upcoming problems. You just wanted to customise a single function, but already had to avoid a pitfall (memory leak) and make a decision on the form of the destructor. Of course, this pitfall is not common to all programming languages, but it is not the only one.

## Single-function interfaces ##

Imagine you are designing a class hierarchy and want to customise a single behaviour. The natural and common in Object Oriented Design approach is to expose this behaviour via an interface like in the C++ examples above. In Java this is probably the way to go. In Python, this may be the way to go (although there are other quite elegant options). In C++, this is a well-trodden path.

But is it the best one? The safest? The simplest? And lastly, the most efficient?

## 