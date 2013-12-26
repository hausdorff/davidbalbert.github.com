---
layout: post
title: General lessons from scaling large systems at Microsoft
comments: true
permalink: scaling-systems-at-ms.html
technique: ■
---

One of the serious disadvantages of working at a place like Microsoft is that *everything* is built for scale, even prototypes. When we roll something out, it gets used[*]. There is no slapping something up in Django "to see if it works" because it only works if it works for millions of clients immediately.

When you build systems in this way, the problems of scale are more brutal and more obvious than when you slowly ramp up over time. Scaling gradually basically consists of putting out a series of fires; scaling at MS is like being lit on fire.

Here are some things that I've learned the hard way.


## A transparent and optimizable runtime is as important as availability of good programming abstractions (*e.g.*, lisp macros, lambdas, *etc*.).

When developing systems that must scale *immediately* to millions of users, it is necessary to be able to reason both about what the code means (*e.g.*, "this code sorts numbers") *AND* how the code behaves (*e.g.*, "slower for in-memory sorting but much faster over network").

The specific balance you choose between these things will depend on your application, but roughly, it breaks down like this.

* When your initial batch of users is in the millions, you'll need to be able to quickly diagnose and (most importantly) *isolate* performance problems. In a language like Python, monitoring for page flush or looking at syscall patterns are hopeless, because it will be hard to tell what is caused by your code and what is caused by the runtime. This is much less the case for C or OCaml, though of course strict type systems and manual memory management have dev costs of their own.
* After identifying the problem, you'll need to be able to reliably and quickly fix it. This may be impossible for languages with prohibitive runtimes like Python (whose GIL, for example, makes it difficult to do many useful things). It could be less hard for languages like C and OCaml.
* On the other hand, if you don't choose a language that's expressive, refactoring becomes almost impossible, and the project becomes unmaintainable.
* As your project becomes mature, seeing transparently into the runtime usually becomes less important, since most of the scaling issues will be worked out. Abstraction becomes more important, because you will need to read the code many more times to maintain it.
* Keep in mind, though, that before your system gets to the point of being mature *it must survive long enough to be mature*. The longer you spend on perf issues per feature you develop, the harder it will be to pivot the system when you find you've designed something wrong, which means your project has a higher probability of dying. More on this in a minute.

Balancing these is a decision that should be considered soberly. In the end, it's hardly ever obvious how to balance these -- for example, when do you choose Java over C++?

Beware of people who want to choose one of these things to the exclusion of the other. They are angels of death, and they herald the demise of whatever they touch.


## Heavy scaling curve kills almost all projects.

Building all products to service `\( n \)` millions of customers at the outset carries a massive hidden burden. By the time we've built and scaled a product, if we find out that what we've built is not what customers want or need, it is usually too late to pivot.

Practically speaking, this leaves us with two options for planning projects:

* Build the system in isolation and pray it's what people need.
* Or, onboard clients *while you're developing the system*.

The first of these options carries a much higher risk of failure, some of which can be mitigated by really keeping a handle on the number of black boxes in the system.

A good example of system simplicity is [Storm](https://github.com/nathanmarz/storm), which really does not have a lot of black boxes. The messaging system is a black box, but the network topology, system configuration, *etc*., is all completely and simply customizable.

A bad example of system simplicity is Hadoop, which is a nightmare to configure, maintain, and develop on.

The second of these options is more taxing for the core team, and usually slower, but also usually more stable. Its main advantage is that you can make sure your system really works for your clients. Ultimately, it also means you can usually have more black boxes (*e.g.* query optimizers, load balancers, *etc*.), because the risk you are misappreciating how the system will be used is dramatically decreased.

It's worth noting that this risk of failure in the face of a steep scaling curve is hard to understate. The project I'm currently on at Microsoft is the successor to *two* other systems that failed to accomplish roughly the same task. This curve simply proved too difficult, and considering the engineering talent around here, this should be a good indicator of how hard either of these options is to do.


## Strong distinction between testing and developments modes is crucial.

The impact of a system is probably best measured in terms of time it saves aggregated across all the developers who interact with it.

If your developers routinely spend hours re-running broken things at scale because there are not good tools to help approximate runtime behavior, you are throwing a good chunk of your potential impact in the trash.

Developing tools that make your system usable is a mission-critical task, not only because it makes other people want to use your tool, but also because it will help you to develop your tool faster, since it will make it easier to diagnose and fix your system to gracefully handle problems encountered downstream.

Generally there are two parts to developing this distinction:

1. establishing good expectations for which types of issues you want to catch in test mode, and 
2. designing test mode specifically to confront those issues.

For example, if you're performing queries on streams of data, test mode should not cache queries, while prod mode might.

This sounds obvious, but it's actually hard to find projects that actually do this.


## Robust and comprehensive test suites are a mandatory prerequisite to development.

It is nearly impossible to debug problems that occur on systems distributed across dozens of poorly-behaved machines in the real world.

If you don't take good precautions here, you will literally spend millions of dollars of developer-hours to find issues that could have been caught immediately.

It should be a core objective to flesh out test cases that cover the essence of the problem you're trying to solve. You should strive for as complete code coverage as you can muster.

As you develop, these tests should be iterated and maintained as you write the code.

Again, **you should do this *before* you write code**, or your debugging will basically consist of running something over a cluster of dozens of badly-behaving machines, and simply waiting for the error. You will have to manually instrument your code with logging to catch the error, and you will be upset that you didn't invest the time to catch the error beforehand.

This takes a lot of discipline but it is so, so worth it in the long run.


## Use simple tools that you understand deeply, and can reuse constantly.

This is a shortcut that is not obvious, but it will save you a lot of time in the long run. Seeing the same issues crop up around your stack is a blessing, because the alternative is encountering new issues in different technologies. Having a good intuition for where perf issues come from is also more than worth the price of admission here.


## Deployment should be a single command.

For Christ's sake. Keep your devs sane.


## Footnotes

\* Well, ok, sometimes our products *don't* get used. But here at Microsoft we like to remain optimistic!

<p></p><br/>

*If you enjoyed this post, you should consider applying to work on my team at Microsoft. Drop me a line and we can chat: `clemmer.alexander@gmail.com`*

*Finally, a big thank you to [James Dennis](http://j2labs.io/) and Malcom Matalka for reviewing preliminary versions of this article.*


