---
title: "LaaF, or Logging as a Feature"
excerpt: "On the importance of logging and some practices"
last_modified_at: 2019-04-11
---

Logging is very important. In fact it is so important that I would even paraphrase the famous quote and say it is _too important to be left to chance_.

Quite often logs, or perhaps also a memory dump, are all a developer has while investigating a problem reported by a customer. If the reproduction rate is low or there is no access to the machine on which the problem occurs, the importance of high quality logs becomes evident.

During my career as a software developer there were quite a few times when my successful investigation of the problem relied solely on available logs. On the other hand, I also worked on a project that, believe me, did not use any logging at all. It was a big project, consisting of many modules, importing and exporting data in a number of different formats. None of the parts logged any information. Every time a problem was found, either by automated tests, the testers, or by the users, the developer's role was to reproduce the problem and investigate with a debugger attached. There was simply no other way. Needless to say, this wasted a lot of time.

Based on experience of mine and my colleagues, below are some guidelines that should help developers create more meaningful and useful logs. They are not in any particular order, but I numbered them nonetheless for easier reference.

 1. Write meaningful messages

When writing log messages always keep in mind that sometimes logs are all you may have when investigating a problem. If the problem is reported by a customer or revealed by automated tests, and also difficult to reproduce, the logs may be the only clue to understand the nature of the problem.

Therefore, the log messages should contain enough context and information to make troubleshooting possible.

On the other hand, meaningless messages should be avoided. They only clutter the logs and may hide more important information.

 2. Try to be concise, but not obscure

Nobody wants to read a log entry that goes across the whole screen and provides some simple information in a very verbose way. Messages that span multiple lines should also be avoided.

On the other hand, the log entry should not be abbreviated to the point of needing deciphering.

 3. Log with context

Without proper context, logging messages are uninformative, do not add value, and just increase noise.

Consider the following bad examples:

`Postcondition check failed` _What exactly was wrong?_

`Failed to read message` _Why? What was the source of the problem?_

`Operation failed with error code 3` _Which operation? What does error code 3 mean?_

And a good one:

`Failed to open journal file for writing. Error code 0x00000005: Access is denied.`

 4. Always include object's name or identifier

This is especially important if you are logging information on one of many instances of an object. And even if you think this is not necessary as there is only a single instance and you know which one the message relate to, believe me, sooner or later there will be more.

Let's say you are coding a server and want to log clients activities. If you do it like this:

```
Client connected
Client disconnected
Client connected
Client disconnected
```

There is no way to tell which client the messages regard to. Was it the same one and had some intermittent network problems? Consider an alternative:

```
Client1: connected
Client1: disconnected
Client2: connected
Client2: disconnected
```

Not it is clear that two client were involved.

 5. Take care when logging exceptions

When a situation happens that requires throwing an exception, it is tempting to write code like this:

```c++
if (!succeeded)
{
    LOG(ERROR) << "Calling function Foo(param1, param2, param3) failed.";
    throw BadFooCall();
}
```

This is tempting, as it ensures that the information is logged. However, this will most likely result in the error printed twice, as at some point the exception is caught, and it's information logged.

To avoid this, the exception object should store all the relevant information that describes the problem, to be acted upon and logged at the catch site, e.g.:

```c++
if (!succeeded)
{
    throw BadFooCall(param1, param2, param3);
}
```
