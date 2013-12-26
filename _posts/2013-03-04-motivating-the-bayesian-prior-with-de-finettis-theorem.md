---
layout: post
title: Motivating the Bayesian prior with de Finetti's theorem
comments: true
permalink: motivating-bayesian.html
theory: ■
---

The modern axiomatization of probability theory (proposed in 1933 by [Andrey Kolmogorov](http://en.wikipedia.org/wiki/Andrey_Kolmogorov)) was designed to provide a measure-theoretic *probability calculus*, that is, a definition of the rules for constructing and manipulating mathematical statements involving probabilities. Unfortunately, this axiomatization only tells us how to manipulate probabilities---it does not tell us what they are or how to interpret them. This is one reason why there are different constructive specifications for computing probabilities, *e.g.*, the Bayesian and frequentist perspectives.

One particularly sticky wicket for newcomers is understanding why the Bayesian prior exists as it does, in contrast to seemingly more straightforward techniques. (Or at least, this was true for me.)

My favorite motivation for the Bayesian perspective is well-known, though there is a particularly good presentation in the first chapter of Emily Fox's excellent dissertation [1]. The crux of the argument is that an important class of models called *exchangeable* models are guaranteed by de Finetti's theorem to decompose cleanly into the definition of the Bayesian framework. It's a connection direct enough that anything other than Bayesian inference seems a bit obtuse.

Definitions first. Intuitively, exchangeability means that the order in which data occurs is irrelevant for our model. More formally, a *finitely exchangeable sequence* is a sequence of `\( n \)` random variables `\( (X_1, X_2, \ldots, X_n) \)` such that, for any permutation `\( \pi \)`, and every possible choice of outcomes for the `\( n \)` random variables,

`\[ p(X_1, \ldots, X_n) = p(X_{\pi(1)}, \ldots, X_{\pi(n)}) \]`

Here `\( p \)` is a probability density function.

And *infinitely exchangeable sequence* is a sequence of random variables such that any subsequence is finitely exchangeable.

One reason exchangeability is interesting is because *prima facie*, although it seems to be similar to the assumption of iid, it is actually much richer, and much less restrictive. Consider, for example, if we have a document whose words have been shuffled. Intuitively, if we see an Arabic word, then even it is much more likely that we will encounter other Arabic words elsewhere in the document, since there is a reasonable probability the document itself is in Arabic. The assumption of iid does not capture this, because the words are assumed to be *completely independent*. Exchangeability doesn't assume independence, however, only that ordering doesn't matter, so this will be captured in a model that is exchangeable.

As suggested earlier, de Finetti's theorem will allow us to break the density `\( p(X_1, \ldots, X_n) \)` into components that look very much like the definition of the Bayesian framework.

The crux of the theorem is that, for every infinitely exchangeable sequence, there exists some probability measure `\( \mathcal{V} \)` such that the sequence is iid when conditioned on it. More formally (don't worry, we will unpack this in a minute), *de Finetti's theorem* states that a sequence `\( (X_1, \ldots, X_n) \)` is infinitely exchangeable iff

`\[ p(X_1=x_1, \ldots, X_n=x_n) = \int_{P} \prod_{i=1}^n \mathcal{V}(x_i) \mu(\text{d}\mathcal{V}) \]`

where `\( P \)` is the space of all probability measures, `\( \mu \)` is a measure on the space of measures `\( P \)` , and `\( \mathcal{V} \sim \mu \)` is defined (with caveats I won't mention here) to be the limiting empirical measure

`\[ \mathcal{V}(B) \triangleq \lim_{n\rightarrow \infty} \frac{1}{n} \sum_{i=1}^n I_B (x_i) \]`

with `\( B \)` ranging over the elements of a the probability space. See [2] and [3] for detailed proofs.

Unpacking this a bit, de Finetti's theorem basically says that we're sampling `\( \mathcal{V} \)` from the measure `\( \mu \)`, which is sort of like a distribution over all possible distributions. The claim de Finetti's theorem is making is that, for any infinitely exchangeable sequence `\( (X_1, \ldots, X_n) \)`, there will always exist a measure `\( \mathcal{V} \)` according to which the sequence is completely iid. The last part, about the limit, can be seen as saying that `\( \mathcal{V} \)` is our "belief" in the outcome of some random variable `\( X_i \)`.

This result is very convenient from the Bayesian perspective, but it only really becomes obvious as part of the following corollary. First, we notice that a measure that if `\( \mathcal{V} \sim \mu \)` is a density on `\( \{0,1\} \)`, then we need only one parameter `\( \theta \in \mathbb{R} \)` to fully specify the distribution, since if `\( \mathcal{V}(x_i=0) = \theta \)`, then `\( \mathcal{V}(x_i=1) = 1-\theta \)`. More generally, if the density is defined over `\( K \)` outcomes, then we require only `\( K-1 \)` parameters: `\( \mathbf{\theta} = \{\theta_1, \ldots, \theta_{K-1} \} \)` to completely specify this sampled measure `\( \mathcal{V} \sim \mu \)`.

This implies that, given de Finetti's theorem, there must exist a distribution function `\( Q \)` such that:

`\[ p(X_1=x_1, \ldots, X_n=x_n) = \int_{\Theta} \prod_{i=1}^n p(x_i | \theta) Q(\text{d}\mathbf{\theta}) \]`

From here the dots are pretty straightforward to connect. `\( p(x_i | \mathbf{\theta}) \)` is basically the Bayeisan likelihood---the "belief" about the outcome `\( x_i \)`, according to distribution `\( \mathbf{\theta} \)`. `\( Q(\text{d}\mathbf{\theta}) \)` is therefore the prior---our description of how probable `\( \mathbf{\theta} \)`, our explanation for our data, is.

This is the standard Bayesian recipe. QED.

I like this example for a number of reasons:
By construction, both `\( Q \)` and `\( \mu \)` absolutely must exist, always. So you can always apply this to an infinitely exchangeable sequence.
The density `\( \mathbf{\theta} \)` is fully factorized, that is, each `\( p(x_i|\mathbf{\theta}) \)` is iid of every `\( p(x_i|\mathbf{\theta}), j \neq i \)`, which implies a lot of nice properties, like the fact that any selection of outcomes can be considered a random sample.
The fact that, in general, `\( Q(\text{d}\mathbf{\theta}) \neq p(\mathbf{\theta})\text{d}\mathbf{\theta} \)` indicates that the distribution `\( \mathbf{\theta} \)` does not always have a density, and can in fact be something much richer, like a function space. This is a critical aspect of Bayesian inference that many people simply don't understand, and it has all sorts of consequences, like making Bayesian nonparametrics possible, and making conditional probabilities uncomputable in general.
It is so convenient to think of this class of problems as Bayesian that it seems like a crime to not use Bayesian inference.

I should say, in closing, that I wrote this to be helpful to the me from two years ago. If you think something could be phrased better I'd love to hear it.


### Footnotes

[1] E. B. Fox. *Bayesian Nonparametric Learning of Complex Dynamical Phenomena*. Ph.D. thesis, MIT, Cambridge, MA, 2009.

[2] E. Hewitt and L.J. Savage. *Symmetric measures on cartesian products*. Transactions of the American Mathematical Society, 80(2):470–501, 1955.

[3] C. Ryll-Nardzewski. *On stationary sequences of random variables and the de Finetti's equivalence*. Colloq. Math., 4:149–156, 1957.
