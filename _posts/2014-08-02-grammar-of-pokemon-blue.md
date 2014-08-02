---
layout: post
title: "Can we decide whether a string of Pokemon Blue moves wins the game, using only a regex?"
permalink: grammar-of-pokemon-blue.html
comments: true
analysis: ■
---


Playing all the way through [Pokemon Blue](http://en.wikipedia.org/wiki/Pok%C3%A9mon_Red_and_Blue) is a huge hassle. It can take anywhere between 24 and 48 hours to complete a whole game.

48 hours of pressing buttons?! *Surely there is an easier way!*

I'd rather encode a series of moves as a string (*e.g.*, LLRU == left left right up), and then **decide whether that moveset wins Pokemon Blue using only a regular expression.**

Then, you could sit down, think hard about all the moves you're going to make, write them down all at once, and check if you win.

That should take, like, 10 minutes *tops*! What a time savings!

The rest of the article will be split into 3 posts:

**Chapter 1:** [Pokemon Blue is a Finite State Machine (FSM), and what that means](grammar-of-pokemon-blue-ch-1.html)

**Chapter 2:** [Generating a Pokemon Blue FSM](grammar-of-pokemon-blue-ch-2.html)

**Chapter 3:** [Transforming the Pokemon Blue FSM into a regex](grammar-of-pokemon-blue-ch-3.html)

