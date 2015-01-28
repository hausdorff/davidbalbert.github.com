---
layout: post
title: "More than half of IBM's profits come from mainframes"
permalink: mainframes.html
comments: true
analysis: ■
---


IBM was at the top of the news aggregators a couple days ago as [irresponsible rumors](http://www.donotlink.com/www.forbes.com/sites/robertcringely/2015/01/22/next-weeks-bloodbath-at-ibm-wont-fix-the-real-problem/) spread that it was laying off more than 100,000 people. IBM eventually posted a [scathing denial](https://ibmhkblog.wordpress.com/), though not quickly enough to stem the flow of Internet pundits who showed up in droves (at, *e.g.*, [Hacker News](https://news.ycombinator.com/item?id=8944637)) to explain just how irrelevant IBM really is.

This is a bad habit of the industry: the proclivity of technologists to write off entire markets simply because they have written off the technology that supports them is astoudingly arrogant and maddening.

The IBM discussion, thankfully, revolves partially around a particularly egregious example of this problem: IBM's flagship mainframe business.


## Why is IBM in the mainframe business?
In 2012, IBM released the z12 mainframe after investing $1 billion on development. In 2015, they are scheduled to release the z13, having spent roughly $1 billion more. And when IBM sold a huge swath of its hardware business in 2014, it chose to keep the mainframe business for itself.

Why?

Because it turns out that mainframes are hugely lucrative. As reported by [The Economist](http://www.economist.com/blogs/schumpeter/2012/09/ibms-mainframes) (emphasis mine):

> At any rate, the mainframe is a hugely profitable business for IBM. Only around 4% of the firm’s revenues come from mainframe sales. But once additional hardware, storage, software and all kinds of related services have been factored in, **the mainframe accounts for a quarter of IBM’s revenue and nearly half of profits,** estimates Toni Sacconaghi of Berstein Research.

For perspective, IBM's revenue is about $100 billion. The same research was cited by the [NYT article](http://bits.blogs.nytimes.com/2015/01/13/ibm-introduces-z13-a-mainframe-for-the-smartphone-economy/) on the same matter.

Part of the reason the business is so lucrative is because the lock-in is so high. For one thing, as the European Commission noted when it launched two separate antitrust investigations against IBM's mainframe business, the vast majority of coporate data is still on mainframes (emphasis mine):

> It is estimated that **the vast majority of corporate data worldwide resides on mainframes.** In 2009 approximately **€ 8.5 billion worldwide** [ed. note: ~$10 billion] and € 3 billion in the European Economic Area **were spent on new mainframe hardware and operating systems.**

Another problem is that it is difficult to transition these systems from a highly reliable mainframe to a cluster of very unreliable commodity machines. A single z13 supports running a "private cloud" of up to 8,000 VMs, with 100% uptime and  -- IBM [claims](http://www.fool.com/investing/general/2015/01/24/heres-why-ibm-is-still-building-mainframes.aspx) -- at 49% the cost of writing software to account for commodity hardware that you expect to fail. And then there's the hardware-enabled encryption features. In most nontrivial cases where these guarantees are important, this transition would have to be a rewrite of the system.

Yet another problem is that many companies don't *want* to switch, because it saves them money to run a small, expensive machine instead of a small, expensive cluster. In The Economist article above, Eurocontrol reported saving 50% on software costs. The NYT article reports that Radixx switched from a cluster to an IBM mainframe and saved 50%. If you dig, you find many such stories. It is hard to imagine they will go away.

## A lesson for technologists
It is easy to forget that the number of people who work at "non-tech" companies that *consume* technological products is orders of magnitude larger than the number of people who work at tech-for-tech's-sake companies, like Google, Facebook, or Microsoft.

Yet, the people who work at these companies are largely unheard in forums like HN and Reddit.

As an industry, it often feels like we have a fundamental inability to appraise companies whose job is to appeal to this silent majority. This myopia has to go, or we are leaving money -- potentially billions of dollars -- on the table for someone else to take.





























