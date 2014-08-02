---
layout: post
title: "Theory interlude"
permalink: grammar-of-pokemon-blue-ch-4.html
comments: true
analysis: ■
nopost: true
---


## Preliminary definitions

What does 

A **language** is a set of strings. For example:

* The English language consists of strings like: "The cow is spotted" and "Subscribe to my blog".
* The Python 2.7 language consists of strings like: "`print 3`".
* The Perl language consists of strings like: "`@(#*$)@#(@#)(%@)`".

A **grammar** is the set of rules from which you can derive *all* strings in a language, and *only* the strings in that language. For example:

* Suppose we have a language that is simply the set of strings consisting of 0 or more `a` characters (*e.g.*, `a`, `aa`, `aaa`, and so on). A Python program that could generate all such strings would be "`'a' * n`" for all non-negative `n`. There you go, that's a grammar for that language! Though, using more mathematical notation, we might say: \\( a^n \\) (for non-negative \\( n \\) of course).
* A grammar that specifies all possible valid Python 3 programs (and *only* those programs) is [here](https://docs.python.org/2/reference/grammar.html). (Note: to understand this document, you'll need to understand something called [context free grammars](http://en.wikipedia.org/wiki/Context-free_grammar).)
* A grammar that specifies all possible valid Perl programs may be obtained, but involves a pentagram made of salt and covering your basement in goat's blood.


