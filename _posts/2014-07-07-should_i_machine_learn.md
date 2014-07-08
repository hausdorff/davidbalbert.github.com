---
layout: post
title: "Should I Machine Learn?"
permalink: should_i_machine_learn.html
comments: true
technique: ■
---


As a machine learning acolyte, I spent probably as much time trying to understand things like *how* and *when* to use machine learning as I did understanding the technical details of machine learning itself.

Unfortunately, most of the discussion around machine learning is about the former. The latter gets almost no press by people who are in the know.

So: while I'm certainly not the best person to be writing this, it seems no one else is stepping up to the plate, so I'm going to do it anyway. Failure is cheap, and all that.


What is machine learning?
----------

Machine learning is a loose confederation of problems whose only commonality is that they involve *statistical inference*. Examples of such problems include:

* **Classification.** *e.g.*, is this a picture of a cat or a gorilla?
* **Clustering.** *e.g.*, group all the cat pictures by ones that are most similar.
* **Reinforcement learning.** *e.g.*, learn to predict whether a picture is a cat on the fly! If you get the answer right, I'll give you a cookie. If you get it wrong, I'll punch you in the face. Go!
* *etc*.


What is this "model" thing that machine learning people are always talking about?
----------------

A model is a *concise summary of data* that simply *retains some statistical property of the data that you care about*. These summaries that "model" your data can be used for any number of things:

* **To simply tell you about the behavior of your data.** For example, the *mean* is a model. If you imaging picking numbers at random from 1-10, a *mean does summarize some useful information about your data. The same with the median and the variance. These are extremely lossy models, but they are models of your data.
* **To classify data.** Say you've trained a classifier that classifies whether a photo contains a cat or not. That classifier concisely summarizes your data as "cat photo" or "non-cat photo."
* **As a more space-efficient way to represent data for some other task.** For example, you might generate paraphrases of some document, and then use that paraphrase to classify a document. Or you might present it to a user. *etc*.

This goes on and on. It also gets dramatically more complicated, of course, but the idea that we're providing a concise summary of the data is always true.


What is statistical inference?
----------------

Perhaps unsurprisingly, statistical inference generally involves *inferring some quantity* based on *some set of statistics*.

For example, say I have a bin with some number of black balls and white balls in it. I draw one at random and show it to you. How many times do I need to repeat this before you can accurately estimate the number of ball in the bin?

Statistics! It's magic!

Of course this is a simple example. Generally you're doing something dramatically more complicated.


Wait, why is machine learning not just statistics?
------------------

Statisticians care a lot about things like "asymptotics", "guarantees", and "math." Machine learning people care a lot about things like "methodology" and "how do I *actually do* this statistical inference with a computer."

There's a lot of overlap of course, but if you pick up a paper from a machine learning conference, it will probably to spend most of its time on the latter, rather than the former.

Also, machine learning people often get things like "grants" and "money for grad students", while statisticians get things like "goodwill" and "maybe a grad student next year, if this DARPA thing works out."


Why should I use machine learning?
---------------

**To save people time.** Machine learning doesn't give you perfect answers, but sometimes it can save you a lot of time.

For example, some time in 2011, thousands of Sarah Palin's emails were released. If you were a reporter, you could read all the emails yourself, or you could hire someone like Edwin Chen to [do machine learning magic to automatically discover all the topics she talked about](http://blog.echen.me/2011/06/27/topic-modeling-the-sarah-palin-emails/).

Were the topics perfect? Absolutely not; some of them were junk. But would they have saved you a lot of time reading? Absolutely.

There are lots of other examples, too. If you're a programmer, and you want to implement an algorithm that detects if there's a cow in a picture, you could write an algorithm, or you could get it mostly right most of the time, and just use machine learning.


When should I use machine learning?
-------------

There's enough snake oil and pie-in-the-sky promises out there, that it's worth arming you with a couple simple heuristics that will help you to **disqualify a large number of problems you might use machine learning for.**

The first heuristic is two parts. If one of the following is not true, then you might still be able to use machine learning to solve your problem. But, if *neither* of the following is true, then machine learning probably can't help you. The heuristic is:

* If I give all your data to a real person, can they find a pattern?
* If I give a person your pattern, is it meaningful to them?

So take an example like search engines. If I ask a law librarian to find all relevant cases for something, they will usually produce good work. Likewise, if a search engine produces results for me, I can quickly determine if they're good or not.

And as we all probably know, machine learning has indeed helped search along.

In contrast, take music recommendations. If I give you a series of suggested songs, can you detect a pattern? If you give a list of your favorite songs to even a really good friend, can they generate a list of recommendations? I mean, maaaaaaaayyybe they can, but a lot of times, they can't. Like, I listen to Bach and also stuff like Dillinger Escape Plan. So good luck predicting what else I'll like.

Looking at content recommendation sites, it is obvious that machine learning might save you a bit of time, and it might generate interesting suggestions, but really it's not so much that the recommendations are good as it is that the recommendations are better than nothing.

Of course, it should If your task passes the above it **does not mean it can be solved with machine learning.** It takes a *lot* of work to make a good search engine, for example. So another good question to ask is:

* Can this problem be solved by looking at statistical outcomes?

Actually the answer is "yes" surprisingly often, like in the case of inferring topics in Sarah Palin's email (see above). But that was a surprising result even when it came out. And it is certainly not always true, so when starting out, you should pause and think, "ok, but what is the statistical nature of this."


How are the academic and industrial machine learning communities different?
---------------

Most of the recent academic activity has been oriented towards finding models that exploit the so-called *parsimonious complexity* of the data &mdash; in short, they seek to be *mathematically elegant and concise*, while *still capturing rich variation in real-world data*.

In industry it's much, much simpler: the question is simply *does this thing run in a reasonable amount of time, and give us ok-ish results*?

If you're an algorithms person, there is bad news here: basically everything is exponential. In neither case do people talk much about asymptotic running time.


"Conclusions"
----------------

This isn't really exhaustive so much as it is just a thing I would have found useful a couple years ago. Corrections welcome. I <3 you all.