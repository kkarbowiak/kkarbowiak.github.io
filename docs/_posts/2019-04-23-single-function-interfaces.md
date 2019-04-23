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

## An alternative solution ##

An alternative to using inheritance is something much simpler and available already in C. If you want to customise a function's behaviour you ...write another one!

Going back to the Logger example, instead of doing this:

```c++
class IWriter
{
    public:
        virtual void writeMessage(std::string const & msg) = 0;
        virtual ~IWriter() = default;
};

class Logger
{
    public:
        explicit Logger(std::unique_ptr<IWriter> writer)
            : m_writer(std::move(writer))
        {
        }

        void log(std::string const & msg)
        {
            m_writer->writeMessage(msg);
        }

    private:
        std::unique_ptr<IWriter> m_writer;
};

class StdOutWriter : public IWriter
{
    // ...
};

class FileWriter : public IWriter
{
    public:
        explicit FileWriter(std::filesystem::path const & file);
    // ...
};

int main()
{
    Logger(std::make_unique(StdOutWriter)).log("this goes to stdout");
    Logger(std::make_unique(FileWriter("log.txt"))).log("this goes to a file");
}
```

you do this:

```c++
using write_func = void (*)(std::string const &);

class Logger
{
    public:
        explicit Logger(write_func write)
            : m_write(write)
        {
        }

        void log(std::string const & msg)
        {
            m_write(msg);
        }

    private:
        write_func m_write;
};

void write_to_stdout(std::string const & msg);
void write_to_file(std::string const & msg);

int main()
{
    Logger(&write_to_stdout).log("this goes to stdout");
    Logger(&write_to_file).log("this goes to a file");
}
```

This form is more concise. It does not force client code to use inheritance. It does not introduce potential memory leaks and does not force using dynamic allocation nor smart pointers. However, it is not perfect.

As you can see, the second customised function is supposed to write to a file, but due to its fixed signature, there is no way to provide the file name. It either has to be hardcoded (bad!), or accessible through a global variable (bad!).

The file to write to is some state that is not encompassed by the function. But we can fix this.
