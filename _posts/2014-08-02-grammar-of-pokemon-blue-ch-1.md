---
layout: post
title: "Pokemon Blue is a Finite State Machine, and what that means"
permalink: grammar-of-pokemon-blue-ch-1.html
comments: true
analysis: ■
nopost: true
---


**This post is part 1 of 3 in a [series about solving Pokemon Blue using only a regular expression](grammar-of-pokemon-blue.html).**

In this chapter, we'll make the following points:

* We'll intuitively argue that Pokemon Blue is a "finite state machine" (colloquially, an "FSM").
* We'll define how to encode a sequence of Pokemon Blue moves as a string (*e.g.*, LLRU == left left right up).
* And finally, since it is well know that FSMs can be turned into regular expressions, in principle we should be able to derive a regex that consumes a string that represents a moveset (*e.g.*, LLRU or RDRDUL) and accepts *only* if that moveset causes the player to *win the game*.


## Intuitive argument that this is possible

The world of Pokemon Blue is governed by an invisible grid with finitely many squares. So, pressing right moves you to the square to your right.

<center><img src="images/pkmn/pkmn_grid.png" height=300></center>

If you're not familiar, [watch some of the gameplay](https://www.youtube.com/watch?v=yJC8lRLq8LM) to see what I mean.

What we will argue more formally in a minute is that this makes Pokemon Blue a **[finite state machine](http://en.wikipedia.org/wiki/Finite-state_machine)** (FSM).

And, since it is well known that all FSMs can be encoded as regular expressions, we can recognize whether a moveset wins Pokemon Blue using only a regex, for some definition of "win."

Of course, Pokemon prominently features elements of randomness, *e.g.*, the random Pokemon encounters in the long grass, or the moves your opponents make.

But, while this seems difficult to deal with at the outset, we will see that we can preserve the random elements of the game and still create such an FSM.


## A slightly more formal argument

In more theoretical terms, **we want to see if we can find the formal grammar of Pokemon,** and then we'd like to ask: **is that grammar regular?**

First, some definitions.

Consider the following map. There is a player and a ball-like object.

<center><img src="images/pkmn/pkmn_obj_1.png" height=300></center>

We say that a **game's "state"** is uniquely defined by the position on the map of all objects that exist. In this case, we have a ball and a player, but in the Pokemon world, there are a lot more objects.

Two game states are said to be **"equal"** if and only if all the objects in the respective maps occupy precisely the same locations.

Because there are finitely many positions that objects (including players) can occupy, there are **finitely many possible game states.** In Pokemon Blue, the number of states is massive, of course, but it's still finite.

We can **transition** from one game state to another by moving any combination of objects in the map to other positions. For example, if the player moves one square to the left, the game has "transitioned" from one state to a different state:

<center><img src="images/pkmn/pkmn_obj_2.png" height=300></center>

The same is true if another object, or another set of objects, moves positions. Any movement at all results in a new game state.

Since we have a finite number of states, and a well-defined transition function for moving from one state to another state, Pokemon is nothing more than a big **[finite state machine](http://en.wikipedia.org/wiki/Finite-state_machine)**.

If you took a theory of computation class (see the excellent [Sipser text](http://www.amazon.com/Introduction-Theory-Computation-Michael-Sipser/dp/113318779X/ref=sr_1_1?s=books&ie=UTF8&qid=1401259492&sr=1-1&keywords=michael+sipser) on the subject), then you may remember that **all finite state machines can be expressed as regular expressions.**

**QED.** Kind of. This is all pretty hand-wavy, but it's good enough for our purposes.


## Ok, but what's the "win" state?

It may have occurred to you that, for the regex to accept a successful moveset, you would have to define a goal state for the FSM &mdash; the state the game of Pokemon Blue has to be in to be considered a "win".

In the case of Pokemon Blue, we will assume that the game ends after beating the Elite Four (*i.e.*, the "final boss"), and therefore, any string of moves that continues after such a win will not change the fact that we've won.


## Encoding movesets as strings

In the introduction, we gave an example string, LLRU, which corresponds to "left left right up".

But the Game Boy actually has 8 buttons, and to build our regex, we need to agree on how we're going to encode all 8, plus one character that represents not moving (which we'll use later):

* "`U`" will stand for "up"
* "`D`" will stand for "down"
* "`L`" will stand for "left"
* "`R`" will stand for "right"
* "`A`" stands for the A button
* "`B`" stands for the B button
* "`S`" stands for the Start button
* "`T`" stands for the Select button
* "`N`" stands for no move at all

Repeatedly appending characters from this alphabet can create arbitrarily long strings of moves simliar to the example above. So appending "`L`" infinitely will cause our character to move left infinitely.


## On to the next section

At this point, we should all agree that you can encode a game of Pokemon Blue as a FSM, and we should all take for granted that a transformation exists that will turn this into a regex.

In the [next section](grammar-of-pokemon-blue-ch-2.html), we will start by showing how to programmatically build regexes that accept if a moveset solves a simple maze, and progressively add on to this model, until we can programmatically construct a regex that solves significant portions of Pokemon Blue.
