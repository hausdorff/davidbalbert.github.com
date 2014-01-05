---
layout: post
title: What "viable search engine competition" really looks like.
comments: true
permalink: building-search-engines.html
analysis: ■
---

Hacker News is up in arms again today about the RapGenius fiasco. See [RapGenius statement](http://news.rapgenius.com/Rap-genius-founders-rap-genius-is-back-on-google-lyrics) and [HN comments](https://news.ycombinator.com/item?id=7010997). One response article argues that we need [more search engine competition](http://peebs.org/2014/01/04/we-need-viable-search-engine-competition-now/) and the HN community largely seems to [agree](https://news.ycombinator.com/item?id=7011472).

In this response article, and in much of the discussion, there is a picaresque notion that the "search engine problem" is really just a product problem, and that if we try really hard to think of good features, we can defeat the giant.

I work at Microsoft. Competing with Google is hard work. I'm going to point out some of the lessons I've learned along the way, to help all you spry young entrepreneurs who might want to enter the market.


### Lesson 1: The problem is not only hiring smart people, but hiring *enough* smart people.

The Bing search relevance staff is probably ~20% the size of Google's. And Google's engineers are amazing. It might be possible to make up for a small difference in size, but competing with a brilliant workforce that is *5 times as large as yours* is very hard.

What's worse is that the problem is not even that we need to buy such a team. MS is rich enough that it could do that if it wanted to. The problem is *finding enough people* to build such a team. There are a limited number of search relevance engineers available, and many of them work for Google.

Thus, to be a viable threat to Google, you will need to take this into account and compensate somehow.


### Lesson 2: competing on market share is possible; relevance is harder

Bing holds somewhere around ~20% of market share. The field is still dominated by Google, but this is no small chunk. We can get this sort of adoption by doing things like tying it to IE. In this sense it is reasonably simple, if expensive, for Bing to compete with Google.

In contrast, consider that we've invested *at least* tens of millions into *just* search relevance &mdash; I'm not even counting infrastructure here. Since there aren't enough relevance engineers to go around, we've been forced to make creative investments in order to get the relevance ratings we have.

In this sense, we've gotten pretty good mileage out of our team, but the fact remains: the difference in search quality &mdash; perceived or real, it doesn't matter &mdash; is noticeable to a large subset of the people who use search regularly. More specifically, as an entrepreneur you have to answer this question: **how would you make these investments differently, and how much money would you need to do it correctly?**

Try coming up with an answer, it's actually very hard. So far I don't think anyone has succeeded. Ultimately I think this will be a very hard gap to bridge directly, and entrepreneurs should plan accordingly.


### Lesson 3: social may pose an existential threat to Google's style of search

Google, like all search companies, is entasked with providing an easy way to access information that's important to people.

But the information people seem to care about the most is locked away in social sites like Facebook. This is inaccessible to Google. Since they rolled out G+, they must think this is a credible threat, so they will probably keep an eye on you if you approach from this angle.


### Lesson 4: large companies have access to technology that is often categorically better than OSS state of the art

A good example of this is NoSQL datastores. It is generally a huge struggle to stably deploy current OSS NoSQL solutions on a couple hundred, let alone a couple thousand nodes. Facebook gave up on Cassandra, and Twitter stopped trying to migrate after a couple years' effort.

In contrast, Amazon, Google, and MS all have stable deployments on clusters that are *an order of magnitude* larger than the largest known OSS NoSQL store (excepting maybe Riak).

Another problem is that these solutions tend to be developed end-to-end, so that they all fit together, and are designed to work together. This is not usually true of OSS, where you tend to cobble together lots of ill-fitting tools until your system starts limping along.

This should give you a sense of the scale at which these companies operate. Entrepreneurs should not expect to compete with the raw processing power of a company like Google. You will have to either be very smart about holes in their stack, or you will have to find another way.

(*NOTE: this is not to say that OSS does not have its advantages, like the fact that some tools can be reused all over the place, and when you know how to stably deploy them, you can do so across your stack, for example.*)


### Lesson 5: large companies are necessarily limited by their previous investments

People use computers primarily to access the Internet. MS is now a devices and services company, hence its main job is to provide the Internet to people as a service on MS devices.

If MS wants to maintain its position as a field leader, it can't just be the OS and the browser used to access the Internet &mdash; it must be a substantive part of the service itself, which means that it needs to be the page people land on when they open their browser.

And that is what Bing is. For this reason, it is more important (for the moment) that Bing exists than it is that it is equal to or better than Google in every way. Of course, it is a huge priority to make Bing better, but this is not the only consideration MS must make.

Fortunately, a similar investment problem exists for all large companies, Google included with it's G+. This is an advantage for entrepreneurs, and you would be wise to use it.

Practically, this means that at a startup, one could have spent more time getting a small but rabidly positive set of users, and built up an engine slowly, rather than simply jumping into feature parity. This is a distinct advantage for the entrepreneur.

### Lesson 6: large companies have much more data than you, and their approach to search is sophisticated

IE can track, and has tracked users behavior even when they're not on Bing. (We got in trouble for this once, hehe.) Google says they don't do that with Chrome, but both of them will try to figure out things like how many times you pressed the back button. They know about a lot of small details, like the fact that it's really important to serve results *fast*.

Some more examples. MSR has published papers that indicate that it's useful to track your behavior [across tabs](http://jeffhuang.com/Final_Branching_WSDM12.pdf) and [based on where you point your mouse](http://jeffhuang.com/Final_CursorModel_SIGIR12.pdf). It's important to recognize that even if Bing doesn't end up using all of MSR's research, the fact that they've spent a lot of money doing the research means that they've tried and discarded a *lot* of things.

It also takes a lot of time and energy to iterate a new search algorithm. You discover some features, you put them in your model, you pilot your model, you use that model to discover more features, and so on. This has a compounding effect that is really hard to make up for if you are behind.

If you're looking to enter the space, be aware that most traditional search problems have really been investigated thoroughly. Someone should either know vaguely what they're doing, or else you should do something different. (Or else, everyone overlooked something really important somehow.)


### Conclusions

While this is by no means a comprehensive list of the challenges of building such a competitive engine, it should at least give a flavor of the sorts of problems you will have to negotiate in some way.