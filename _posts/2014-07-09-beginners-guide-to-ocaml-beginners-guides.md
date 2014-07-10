---
layout: post
title: "Beginner's guide to OCaml beginner's guides."
permalink: beginners-guide-to-ocaml-beginners-guides.html
comments: true
analysis: ■
---


So you want to learn OCaml. Where do you start? What do you do?

I've been an OCaml beginner probably a dozen times &mdash; picking it
up, dropping it, and picking it up again so many times I've lost
count.

This time it's stuck, and I think it's because the community has
fundamentally changed.

Here's what worked for me.


What books are available?
-------------------

Read **[Real World OCaml](https://realworldocaml.org/)**
(colloquially: RWO), and **accept no substitutes**. It might be the
best computer language book I've ever read in my life.

In addition, I would advise **against** reading other books, as they
tend to be incorrect and/or in French.

Here are some nice things about RWO.

* **Cogent and well-written.**
* **Fully available as HTML
  [online](https://realworldocaml.org/v1/en/html/index.html).** This
  means (1) you don't have to buy the book, and (2) you can copy +
  paste code segments directly.
* **Provides clear operational justification for nearly every
  important language feature.**
  * By "operational justification", I mean that the authors provide
    enough knowledge of how the feature works under the covers that
    you could be safe justifying its use for a production system.
  * A good example is their coverage of pattern matching. It turns out
    that matching desugars down to a finite state machine. So, if two
    patterns overlap, checking them will look like going through a
    bunch of very cleverly nested if statements, rather than checking
    one pattern, and always starting completely over again for the
    next pattern. Providing this knowledge is valuable to someone like
    me, who needs this sort of intuition when writing production code.
  * This is in contrast to a lot of books sort of walk you through the
    core abstractions (*e.g.*, "here's a `for` loop"), and the core
    library (*e.g.*, "call this to get a `List<T>`), and give you no
    other context.
* **A whole section on the runtime.** If you don't have good access
  to knowledge about how the runtime of your production language
  works, you are taking a huge risk. This section provides a lot of
  the knowledge you would need to make this risk manageable. **Every
  computer language book should have a section like this;** I'm not
  sure why almost none of them do.
* **A whole section on tooling to solve important tasks.** Includes
  things like parsing and lexing, serialization, concurrent libraries,
  etc.
* (There's also a section about the language, the features, and the
  Core API, but that goes without saying.)
* **Ignores the horrible standard library and tooling.**
  The book is focused around the *vastly* superior toolchain put out
  (mostly) by Jane Street Capital and OCaml Labs. Before RWO, the
  standard library and the standard tooling (*e.g.*, package
  managers) were a massive stumbling point for learning OCaml,
  because they were so uniformly terrible. Thankfully this new
  toolchain is amazing, well-documented, and well-supported, and RWO
  rightfully centers around this instead.


What tools should I use?
------------------------

OCaml has a *very* strong type system. A combination of this fact,
plus the fact that the types are usually *inferred* (*i.e.*, they are
usually not written down explicitly), makes OCaml language where your
intuition about what *should* be correct will be regularly shot down,
and then shoved in your face until you get it right.

Please.

Please, please make this easy on yourself.

Invest in tooling that will shorten the gap between your writing
something, and your compiler telling you it's totally wrong.

Follow the
[RWO installation instructions](https://github.com/realworldocaml/book/wiki/Installation-Instructions)
*to the end*. In particular:

* **Install utop (the OCaml REPL).** You'll use this to experiment a lot
  and confirm your intuition by running code.
* **Install OPAM (the OCaml package manager).** This should go without
  saying. It used to be hard and annoying to switch the version of
  OCaml or install simple libraries. OPAM is truly a joy to use
  compared to those days, and even compared to other languages.
* **Editor tooling. Just do it.** You want Tuareg and Merlin, which
  work on both emacs and vim. Don't use emacs and vim? **Use them for
  OCaml then. The tools are that much better than using another editor
  and no tools.**

Since it takes time and energy to invest in tooling, I'll try to
entice you by showing you some stuff that's cool that you can do with
them.

**Files are compiled on save,** which means that **things that don't
compile are highlighted in yellow:**

<center><img src="images/ocaml-beginner/wrong.png" width=600/></center>

As you type, Merlin will **produce a list
of autocomplete suggestions:**

<center><img src="images/ocaml-beginner/autocomplete-1.png" width=600/></center>

Merlin also has a hotkey (on emacs it's C-c <TAB>) that will bring up
a **list of suggested autocompletes:**

<center><img src="images/ocaml-beginner/autocomplete-2.png" width=600/></center>

Another hotkey (on emacs it's C-c C-t) takes  the expression that the
cursor is currently at **and tells you what type it is!!!** (It's
included at the bottom of the screen.) This is **incredibly**
convenient because the compiler then doesn't have an opportunity to
complain about types.

<center><img src="images/ocaml-beginner/typeof-2.png" width=600/></center>

If you press this same key combo on an expression that is a type, it
simply **brings up the type definition!!**

<center><img src="images/ocaml-beginner/typeof-1.png" width=600/></center>

There are a **lot of things you can do here**. This is just a
taste. It's an optional thing, but really, it's **well worth the time
savings.**


What are some good examples of source I can read?
-------------------

Good question! It's important to see really skilled programmers use
OCaml in a really idiomatic way!

The best open OCaml system is probably the
[Jane Street Core](https://github.com/janestreet/core) and the
[Jane Street Core Kernel](https://github.com/janestreet/core_kernel). For
example, here is their
[map implementation](https://github.com/janestreet/core_kernel/blob/master/lib/core_map.ml),
which is very good.

The source for utop and anything coming out of OCaml Labs is good, but
unfortunately, good open source OCaml projects are still somewhat hard
to come by in general. Eventually this will probably change.


What are some good tutorials?
-----------------------------

The [standard OCaml tutorials](http://ocaml.org/learn/tutorials/) are
now awesome. They used to be non-existant, then they were
terrible. Now they are awesome.

The 99
[OCaml problems exercise](http://ocaml.org/learn/tutorials/99problems.html)
is ok. It might not be the best way to learn, depending on your
background, but I found it to be a useful way to learn common
manipulations for standard OCaml data structures. In particular, I
found that writing something, and then looking at the solution, showed
me where I was not using idiomatic OCaml. That was helpful.

There are both contained on the
[the official OCaml "learn" page](http://ocaml.org/learn/)
. It's a pretty good resource to start, and it will be consistently
updated in the future.


Where is good documentation?
----------------------------

A good start is the documentation for
[Core.Std](https://ocaml.janestreet.com/ocaml-core/111.17.00/doc/core/#Std). This
explains the standard data structures of the Jane Street Core, which
is a good place to start.

To get started with that, it may be necessary to look through the
relevant online
[RWO chapters](https://realworldocaml.org/v1/en/html/index.html). (Certainly
this was true for me.)


Who can I talk to?
------------------

I have friends who I talk to. I hear IRC is good, and I hear there are
mailing lists. Personally I don't use either.


What do you recommend for beginners?
------------------------------------

Everyone's path varies, but personally I had the most luck with the
following.

* Start by implementing applications commensurate to your level of
  experience. I picked a Boggle solver, a command line parser, a regex
  parser, and a couple machine learning problems. You might pick
  something easier or harder, depending.
* Use your editor to check types of expressions often. This reduces
  compiler frustration. Use your editor compile-when-you-save feature
  to check if things compile. It's *way* faster to do this
  interactively with an editor than to compile outside your editor.
* I find it very useful to start by sketching out my types and their
  interfaces before I write any code. YMMV.
* Try finding a dual in OCaml to all the things you like to do in
  another language. For example, Python's list comprehensions are a
  special case of the monadic bind. Sweet! Once you have all these
  down, you'll be more comfortable in your language, because
  translating your thoughts to code gives you something solid to fall
  back on.
* Read a lot of source if that helps. I found the Jane Street core
  very useful for understanding the idioms of OCaml.
* 99 problems, *etc*., let you implement something, and then see
  idiomatic ways to do it. This can be very very useful.
* Get code reviews. I have friends who help me with this. It's
  incredibly helpful for understanding where you do something not
  OCaml-kosher.


Conclusions
----------

I guess I don't really have anything to say in parting, other than
that if you have comments about what does/does not work, I'd be happy
to hear them.
