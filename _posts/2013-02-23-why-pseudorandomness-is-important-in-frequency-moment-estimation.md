---
layout: post
title: Why psuedorandomness is important in frequency moment estimation
comments: true
permalink: freq-moments.html
theory: ■
---

Finding the `\( p \)`-th frequency moment, denoted `\( F_p \)`, is one of the most well-studied problems in streaming algorithms, with a broad set of applications ranging from traffic monitoring on networks, to efficient entropy estimation, to database query optimization. In the streaming setting, this task amounts to computing `\( F_p(\mathbf{x}) = \sum_{i=1}^n |x_i|^p \)` where `\( p>0 \)` and `\( \mathbf{x} \)` is a very very high dimensional vector that is being continuously updated.

In general it is difficult to do this both quickly and with small memory footprint. One popular approach is to "sketch" our `\( m \)`-dimensional update vector `\( \mathbf{x} \)` with a much smaller `\( n \)`-dimensional vector, `\( \mathbf{y} \)`. The general idea is to sample an `\( n \times m \)` matrix `\( A \)` from some probability distribution, and then derive the sketch `\( y \)` as a series of updates to the dot product `\( y = A \mathbf{x} \)` . One way this could be beneficial is, if the dimensionality of `\( \mathbf{y} \)` is much smaller than `\( \mathbf{x} \)`, then incurring the cost of storing the matrix `\( A \)` allows you to store a greater number of accurate sketches than you would be able to store if they were explicit vectors instead.

This motivates an important question in the area: how can we concisely represent the matrix `\( A \)`? Unfortunately, most (all?) results that use this general pattern require that each coordinate `\( A_{i,j} \)` of the matrix be the same throughout the algorithm, since this guarantees that each update to coordinate `\( x_j \)` will be processed the exact same regardless of the time of processing. But can we get away with maintaining this invariant without explicitly storing `\( A \)`?

If we know that our application is going to interact with `\( s \)` random bits, then the most naive solution would involve storing `\( s \)` bits pulled from a random source. This amounts to explicitly storing `\( A \)`. One critical insight is that we can obtain `\( s \)` "almost random" bits by building a pseudorandom number generator `\( R : \{0,1\}^r \rightarrow \{0,1\}^s \)` that takes a seed of `\( r \)` bits and maps that to a pseudorandom sequence, from which we can take the `\( s \)` needed bits. This would be a definitive win for our algorithm: instead of storing `\( s \)` "truly" random bits, we need only to store `\( r \ll s \)`, which we can use to generate blocks of pseudorandom numbers, which allows us to replicate the effect of having a stable matrix `\( A \)`.

One of the more interesting results in the area from the last few [years][1], uses basically this approach, showing (and I'm paraphrasing greatly here) that bounded independence of the pseudorandom numbers allows enough derandomization to construct an algorithm for moment estimation that is asymptotically space optimal.

I won't share the (somewhat complicated) derivation here, but there are many interesting questions that remain open in the area. The most important one in my opinion? So far, it seems that most algorithms for pseudorandom bitstreams lead to provably suboptimal space consumption in sketches. Is it possible to develop algorithms for generating such bitstreams that pathologically and on a very general basis lead to optimal space consumption? I suspect not, but either result would be very important for the future of the area of research surrounding streaming algorithms.


### Footnotes

[1]: Daniel M. Kane, Jelani Nelson, and David P. Woodruff. *An optimal algorithm for the distinct elements problem*. In Proceedings of the 29th ACM SIGACT-SIGMOD-SIGART Symposium on Principles of Database Systems (PODS), pages 41–52, 2010.
