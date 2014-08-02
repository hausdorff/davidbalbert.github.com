---
layout: post
title: "Generating a Pokemon Blue FSM"
permalink: grammar-of-pokemon-blue-ch-2.html
comments: true
analysis: ■
nopost: true
---


**This post is part 2 of 3 in a [series about solving Pokemon Blue using only a regular expression](grammar-of-pokemon-blue.html).**

In the **[previous chapter](grammar-of-pokemon-blue-ch-1.html)** we:

* saw that Pokemon Blue is a Finite State Machine (FSM),
* learned that all FSMs can be converted into regexes,
* asserted that creating the "Pokemon Blue regex" is thus a matter of (1) generating an FSM representing the game, and (2) tansforming it into a regex.

In **this chapter**, we'll learn how to encode most of Pokemon Blue as a FSM. In particular:

* We'll start by generating FSMs for very simple mazes.
* We'll expand this these FSMs to account for increasingly complicated features, like random encounters, random-ish battle behavior, menu screens, *etc*.
* We'll provide code that takes a Pokemon Blue map and auto-generates an FSM for that map.

In the **[next chapter](grammar-of-pokemon-blue-ch-3.html)**, we will show how to programmatically convert these FSMs to regexes.


## Building FSMs that represent simple mazes

The bulk of Pokemon Blue is spent walking around in the world.

Here is a simple example maze, encoded in vanilla ASCII:

```
****
*S *
*  G
****

S = start
G = goal
* = wall
```

A reasonable question to ask here is, what does it mean to encode this maze as an FSM?

Straightforwardly, **the FSM here will represent the moves a player can make to get from space to space in the maze.** We'll present the actual FSM for this text maze first, and explain it immediately after.

<center><img src="images/pkmn/simple-pkmn-fsm.png" height="300"></center>

As we can see, in the starting position (in the upper lefthand corner), moving `L`, `U`, or `N`, will cause us to stay in the same space &mdash; we are running into a wall.

But, moving `R` in the starting position causes us to move into the space in the upper righthand corner.

And moving `D` will move us to the bottom left corner.

And so on.

**Notice the double circle at the bottom.** This is the "goal" state in the maze, and in standard FSM parlance, it is called the "accept" state. This state is precisely the same in both the FSM and in the original maze: we can continue moving as we like in the maze, but to win we have to get to the goal state in the bottom right corner. Likewise, we can transition anywhere we like in the FSM, but to "win" the maze, we have to end our journey in this double-circle "win state."

Notice also that in our FSM, once you get to the goal state, there is no way to leave it &mdash; if you win, you win.


## Dealing with random encounters

Randomness 
