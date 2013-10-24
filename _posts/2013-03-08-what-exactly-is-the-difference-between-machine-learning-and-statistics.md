---
layout: post
title: What exactly is the difference between machine learning and statistics?
permalink: ml-vs-stats.html
theory: •
analysis: •
technique: •
---

I often get asked about the difference between statistics and machine learning. It is a tricky distinction because some things that were invented for ML (*e.g.*, PAC theory) also get a lot of play in statistics journals, and vice-versa.

To say they're completely equivalent (which is what I often hear) is probably a bit too strong. I tend to think of ML as mostly a contribution to the general problem of statistical inference, particularly as it is applied computationally. In contrast, statistics as a field is certainly not entirely devoted to inference.

That distinction goes a long way to explaining why they are different fields, but there are a bunch of idiosyncrasies that are worth pointing out specifically because they are illustrative.

* Unlike statistics, there are very few ideas that really "unite" ML as a field. People seem to have split up work into a set of common tasks (*e.g.*, classification, clustering, reinforcement learning), and these tasks really only seem to be unified by the fact that they depend on statistical inference.

* Statistics tends to focus on longer journal publications, which are heavy on mathematics and theoretical analysis. ML tends to focus on shorter publications at conferences, with empiricism (especially in explaining methodology and evaluation of results) playing a much bigger role.

* Distinctions important to statistical theory (*e.g.*, whether a procedure is frequentist or Bayesian) are often not emphasized or are totally absent from discussion in ML. In a lot of cases, this is because the discussion of the empirical results is emphasized, so discussion about the different types of guarantees given by different theoretical assumptions becomes less meaningful because the theoretical analysis and guarantees are not given as much play to begin with.

* ML people tend to import globs of statistics literature as it helps them to accomplish the tasks they care about. It is sometimes said that ML is a computational interpretation of statistical inference, and this is not hard to see that this is mostly a fair assessment.

One thing ML people should get credit for, is that they are usually transparent about borrowing heavy amounts of material from statistics and CS, and typically do not claim to have invented a new field. No doubt this saves them many angry letters from statisticians. On the flipside, statistics has benefitted greatly from ML as well, and especially (at least in my opinion) from their focus on empiricism. Since it's actually reasonably common for people of both fields to be confused about what the others do, it's reasonably important to note that this cross-fertilization does happen.