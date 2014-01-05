---
layout: post
title: What "viable search engine competition" really looks like.
comments: true
permalink: building-search-engines.html
analysis: ■
---

Hacker News is up in arms again today about the RapGenius fiasco. See [RapGenius statement](http://news.rapgenius.com/Rap-genius-founders-rap-genius-is-back-on-google-lyrics) and [HN comments](https://news.ycombinator.com/item?id=7010997). One response article argues that we need [more "viable search engine competition"](http://peebs.org/2014/01/04/we-need-viable-search-engine-competition-now/) and the HN community largely seems to [agree](https://news.ycombinator.com/item?id=7011472).

In much of the discussion, there is a picaresque notion that the "search engine problem" is really just a product problem, and that if we try really hard to think of good features, we can defeat the giant.

I work at Microsoft. Competing with Google is hard work. I'm going to point out some of the lessons I've learned along the way, to help all you spry young entrepreneurs who might want to enter the market.

(*This should go without saying, but in no way was this edited or approved by my employer. Conclusions are drawn purely from public knowledge, and/or my own foolish hunches. Nothing is derived from office talk.*)


### Lesson 1: The problem is not only hiring smart people, but hiring *enough* smart people.

The Bing search relevance staff is a fraction the size of Google's. And Google's engineers are amazing. Making up for a small difference in size could be easy; competing with a brilliant workforce that is *n times as large as yours* is very hard, especially when n > 3, which in our case, it is.

What's worse is that the problem is not even that we need to buy such a team. MS is rich enough that it could do that if it wanted to. The problem is *finding enough people* to build such a team. There are a limited number of search relevance engineers available, and many of them work for Google.

This is a constant problem for all who enter the field, and to be a viable threat to Google, you will need to take this into account and compensate somehow. Obviously we have our own strategies for dealing with this problem.


### Lesson 2: competing on market share is possible; relevance is much harder

Bing holds somewhere around > 20% of market share according to publicly-available sources. Google is still the player to beat in the field, but this is no small chunk. It is obvious that some of this market share comes from reach we have through things like IE and Windows, and through public partners like Facebook. This sort of reach isn't free, but it's not nearly as difficult as getting good relevance scores.

And getting good relevance is hard, make no mistake. Consider that Bing has invested *at least* millions into *just* search relevance &mdash; I'm not even counting infrastructure here. Since there aren't enough relevance engineers to go around, the only alternative is to make creative investments in this area. Certainly we have had no choice but to do this in order to get the reasonably good relevance ratings we have. In this sense, it is possible to get good mileage out of the team with the right strategy, as Bing has, though it is certainly not a given.

Still, some dissonance exists here: the difference in search quality &mdash; perceived or real, it doesn't matter &mdash; is noticeable to some subset of the people who use search regularly. As an entrepreneur you will have to confront that: **how would you make these investments differently, and how much money would you need to do it correctly?**

At this point, I'm honestly not sure that, given the goal of producing a scale search engine, we could have been done this much better. I think the only other option might have been to try an entirely different attack vector. Either way, if you try this yourself, you will see that this is a very hard, maybe impossible, gap to bridge directly. Entrepreneurs should plan accordingly.


### Lesson 3: social may pose an existential threat to Google's style of search

Google, like all search companies, is entasked with providing an easy way to access information that's important to people.

But the information people seem to care about the most is locked away in social sites like Facebook, or at least is only derivable from information that is locked in those websites. This is inaccessible to Google. Since they rolled out G+, they must think this is a credible threat, so they will probably keep an eye on you if you approach from this angle.


### Lesson 4: large companies have access to technology that is often categorically better than OSS state of the art

A good example of this is NoSQL datastores. It is generally a huge struggle to stably deploy current OSS NoSQL solutions on a couple hundred, let alone a couple thousand nodes. Facebook gave up on Cassandra, and Twitter stopped trying to migrate after a couple years' effort.

In contrast, Amazon and Google both have stable deployments on clusters that are *an order of magnitude* larger than the largest known OSS NoSQL store (excepting maybe Riak).

Another problem is that these solutions tend to be developed end-to-end, so that they all fit together, and are designed to work together. This is not usually true of OSS, where you tend to cobble together lots of ill-fitting tools until your system starts limping along.

This should give you a sense of the scale at which these companies operate. Entrepreneurs should not expect to compete with the raw processing power of a company like Google. You will have to either be very smart about holes in their stack, or you will have to find another way.

(*NOTE: this is not to say that OSS does not have its advantages, like the fact that some tools can be reused all over the place, and when you know how to stably deploy them, you can do so across your stack, for example.*)


### Lesson 5: large companies are necessarily limited by their previous investments

Big disclaimer here: this is my opinion and not something that's MS-official.

People use computers primarily to access the Internet. MS is now a devices and services company, hence its main job is to provide the Internet to people as a service on MS devices.

If MS wants to maintain its position as a field leader, it can't just be the OS and the browser used to access the Internet &mdash; it must be a substantive part of the service itself, which means that it needs to be the page people land on when they open their browser.

And that is what Bing is. For this reason, it is more important (for the moment) that Bing exists than it is that it is equal to or better than Google in every way. Of course, it is a huge priority to make Bing better, but this is not the only consideration MS must make.

Fortunately, a similar investment problem exists for all large companies, Google included with it's G+. This is an advantage for entrepreneurs, and you would be wise to use it.

Practically, this means that at a startup, one could have spent more time getting a small but rabidly positive set of users, and built up an engine slowly, rather than simply jumping into feature parity. This is a distinct advantage for the entrepreneur.

### Lesson 6: large companies have much more data than you, and their approach to search is sophisticated

IE can track, and has tracked users behavior even when they're not on Bing. (We got in trouble for this once. Google says they don't do that with Chrome, by the way.) Both Bing and Google will try to figure out things like how many times you pressed the back button, how many of the search results you visited for a particular query before you found what you wanted, and so on. They know about a lot of small details, like the fact that it's really important to serve results *fast*. (There's a talk by Marissa Meyer about this somewhere, but I forget where.)

Some more examples. MSR has published papers that indicate that it's useful to track your behavior [across tabs](http://jeffhuang.com/Final_Branching_WSDM12.pdf) and [based on where you point your mouse](http://jeffhuang.com/Final_CursorModel_SIGIR12.pdf). It's important to recognize that even if Bing doesn't end up using all of MSR's research, the fact that they've spent a lot of money doing the research means that they've tried and discarded a *lot* of things.

It also takes a lot of time and energy to iterate a new search algorithm. You discover some features, you put them in your model, you pilot your model, you use that model to discover more features, and so on. This has a compounding effect that is really hard to make up for if you are behind.

If you're looking to enter the space, be aware that most traditional search problems have really been investigated thoroughly. Someone should either know vaguely what they're doing, or else you should do something different. (Or else, everyone overlooked something really important somehow.)


### Conclusions

While this is by no means a comprehensive list of the challenges of building such a competitive engine, it should at least give a flavor of the sorts of problems you will have to negotiate in some way.