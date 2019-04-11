---
title: "LaaF, or Logging as a Feature"
excerpt: "On the importance of logging and some practices"
last_modified_at: 2019-04-11
tags:
  - logging
  - guidelines
  - practices
---

Logging is very important. In fact it is so important that I would even paraphrase the famous quote and say it is _too important to be left to chance_. I might even argue that logging is as important as the application's features. Perhaps not for the user, but definitely for you, the maintainer.

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

 6. Remove no longer necessary logging

Often, when a new feature is developed or a difficult debugging session is in progress, additional logs are added. Such logs are often meaningful only for their author, as they can be unstructured, placed randomly, and contain abbreviations. All other suffer from background noise when analysing logs looking for signs of defects.

Once development or debugging is over, the logging statements should be combed through and the non-essential ones should be removed.

 7. Try to avoid ping-pong messages

If you are logging a process that involves exchanging several messages (like some handshake) and perhaps is governed by a state machine, you might be tempted to log every step taken, like for example sending a request and getting a response. This is fine at the beginning, when the code is developed, but afterwards it adds verbosity without actual gains. Let's assume some code goes from state A to state D sending requests and getting responses:

```
A: sent request
A: got response
B: sent request
B: got response
C: sent request
C: got response
D: sequence complete
```

A much more concise and as informative way would be to produce the log that summarizes the whole operation:

```
Transition from A to D complete
```

or, in case of errors:

```
Error: could not send request from B
```

or:

```
Error: incorrect response in C
```

The above single lines are enough to inform that 1) sequence was completed successfully; 2) step A completed, but there was problem in B; 3) all was good up till response in C.

Unless the timestamps of the log messages may help locate problems with slow communication in themselves, let's reduce their number.

 8. Use proper message level

This one is tricky, as it is often difficult to determine at what level a particular entry should be logged. Many logging frameworks provide more or less the following message levels:

 * **FATAL**<br/>Things have gone bad beyond recovery. Use this very sparingly, as this signifies program termination.
 * **ERROR**<br/>The level at which all errors should be logged.
 * **WARNING**<br/>The level at which potential problems or genuine warnings should be logged. Examples include informing a debug/beta build is running, experimental algorithm is used, an operation timed out, but was successfully retried.
 * **INFO**<br/>The level at which normal operations should be logged, e.g. database connection, operation mode change, messages from OS.
 * **DEBUG**<br/>The level at which function calls, parameter or return values, and algorithm steps should be logged. Mostly used during debugging, and should be trimmed down or removed completely before committing to main branch.
 * **VERBOSE**<br/>Use sparingly, and mostly during development or debugging. Think twice whether you should really leave those messages afterwards.

It should be noted here that an error on one (low) application layer may not be considered an error on other (higher) application layer. Examples include code that cannot read a value from registry, to which upper layer code is prepared by using a default value, or some connection being timed-out, which upper layer handles by retrying several times. In those cases it may be good to let upper layer code decide which condition should be considered error and which not (a warning perhaps) and whether the situation should be logged.

There is also one more thing. I do not like the message severity and verbosity being conflated into a single hierarchy. **These are two separate concepts!** We can have a very verbose error message providing lots of details, as well as a very short debug message that just names current algorithm step. Unfortunately, this mixing is very common in logging frameworks. I have only once seen a framework (internal, unfortunately) which kept the concepts independent and had separate macros for logging short, normal, and verbose messages at each of the levels. It also allowed the same granularity when filtering the messages.

 9. Keep the logs machine-readable

This may seem irrelevant as long as the logs you read are short, but once you are confronted with a multi-megabyte (or even gigabyte) log file, looking for a problem may turn into looking for a needle in a haystack. In such cases having the logs in machine-friendly format is a game changer. You can write scripts to help analyse the logs and provide statistics on how often different problems occur, how regular they are, and what are the common issues.

 10. Review logging during code reviews

I hope by now you understand that logging is far too important to be neglected. Therefore, when reviewing code, pay attention to logging statements. Are the logs informative, meaningful, relevant? Do they provide enough context and use proper level? In other words, do they conform to above guidelines? If not, do not hesitate to flag them. After all, some day you may need to read them.
