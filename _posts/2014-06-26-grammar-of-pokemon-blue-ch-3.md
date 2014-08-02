---
layout: post
title: "Transforming the Pokemon Blue FSM into a regex"
permalink: grammar-of-pokemon-blue-ch-3.html
comments: true
analysis: ■
nopost: true
---


## Converting the Pokemon FSM to a regex

So we have a simple Pokemon FSM, like this one:

<center><img src="images/pkmn/simple-pkmn-fsm.png" height="300"></center>

From here, the question is, how do we turn this FSM into a regular expression?

One method is the [state-removal method](http://cs.stackexchange.com/questions/2016/how-to-convert-finite-automata-to-regular-expressions), which involves continually removing states from the FSM, and updating the edges accordingly.

If you're interested in the specifics of the algorithm, I recommend looking over that link.

But, as an example, let's remove upper-right state of the following FSM as see an example.

<center><img src="images/pkmn/state-removal-1.png" height="300"></center>

Consider the leftmost node. It is possible to transition to the upper right node and then back to the leftmost node again by moving `R->[RUN]*->L`.

If we remove the upper-right node, though, this transition will be lost. So, the first step to take is to **combine this transition into a single edge**. In this figure we do the same for the bottommost node as well.

<center><img src="images/pkmn/state-removal-2.png" height="300"></center>

Both the leftmost node and the bottommost node already have self-looping transitions, though. So, we combine this new self-loop with that self-loop. **This constitutes updating the regex on each of the self-loop edges.** We're not done yet, but the updated FSM looks like this:

<center><img src="images/pkmn/state-removal-3.png" height="300"></center>

Finally, we notice that for the leftmost node to transition to the bottommost node, we need only transition `R->[RUN]*->D`.

Again, though, if we were to remove the rightmost node, this transition would be lost. So, **we must draw a new edge with this label between the leftmost node and the bottommost node.** If we do this in the reverse, too, so that the bottommost node is connected with the leftmost node, we can fully remove the top-right node.

<center><img src="images/pkmn/state-removal-4.png" height="300"></center>

And that is the state removal strategy.

And if we apply that exact strategy to the FSM for the maze we had previously we get something that looks very similar:


<center><img src="images/pkmn/simple-reduced-1.png" height="300"></center>

If we continue with this strategy, and delete a few more states, **our FSM will eventually look like this:**

<center><img src="images/pkmn/simple-reduced-2.png" height="300"></center>

And by concatenating regexes on these edges, **we finally obtain the regex that accepts any moveset that completes the maze:**

`([LUN]|R[RUN]*L|D[LDN]*U|(R[RUN]*D|D[LDN]*R)([DN]|U[RUN]*D|L[LDN]*R)*(U[RUN]*L|L[LDN]*U))*(R[RUN]*D|D[LDN]*R)([DN]|U[RUN]*D|L[LDN]*R)*R[RLDUN]*`

**All that for a dinky 5-space maze?!?!?!?** You can begin to see how quickly this gets out of control.


## *Programmatically* building such regexes


