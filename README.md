# Making a ternary computer
I don't think I need to introduce to anyone reading this the concept of binary numbers and how they are the basis of every computer since their invention.
But actually this would be wrong, binary was not always the only option for building computers.
In 1958, the Setun was developped in Moscow with balanced ternary as a basis.
And it is believed that a return to ternary could open up potential to improve computation, notably with LLM systems.

This, however, is entirely outside of my concern.
The main takaway I get is that it is possible to get 3 different states on a wire, as it's already been done before, and therefore I set out to see if I could manage to create an architecture that would make this work.

## Understanding extended boolean algebra
The first step is to understand how our usual gates would even work with 3 values.
Obviously it has to be some esoteric nonesense we've never seen before.
But actually binary computers can already simulate perfectly accurate ternary state logic if you introduce the $\color{blue}{NUL}$ value.

First the simplest gate:
- NOT $\color{red}{FALSE}$ $\implies$ $\color{green}{TRUE}$
- NOT $\color{green}{TRUE}$ $\implies$ $\color{red}{FALSE}$
- NOT $\color{blue}{NUL}$ $\implies$ $\color{blue}{NUL}$

There doesn't seem to be any other way to map this.

Let's take it up a notch with $AND$, and go exaustively:
- $\color{red}{FALSE}$ AND $\color{red}{FALSE}$ $\implies$ $\color{red}{FALSE}$
- $\color{red}{FALSE}$ AND $\color{green}{TRUE}$ $\implies$ $\color{red}{FALSE}$
- $\color{green}{TRUE}$ AND $\color{green}{TRUE}$ $\implies$ $\color{green}{TRUE}$

So far, no surprises.
- $\color{blue}{NUL}$ AND $\color{blue}{NUL}$ $\implies$ $\color{blue}{NUL}$

I mean what else would it be ?
We have no information going in so we should expect no information comming out.
- $\color{red}{FALSE}$ AND $\color{blue}{NUL}$ $\implies$ $\color{red}{FALSE}$

Seems logical, $\color{red}{FALSE}$ AND anything should be $\color{red}{FALSE}$, after all.
- $\color{green}{TRUE}$ AND $\color{blue}{NUL}$ $\implies$ $\color{blue}{NUL}$

That is a bit more interesting, but it still makes some sense.
We have no way of assigning a meaningful value to the output since the only two straight forward cases we know of are that either one of the inputs is $\color{red}{FALSE}$, or both of the inputs are $\color{green}{TRUE}$.

So we arrive at this truth table:
| AND | $FALSE$ | $NUL$ | $TRUE$ |
|:---:|:---:|:---:|:---:|
| $FALSE$ | $\color{red}{FALSE}$ | $\color{red}{FALSE}$ | $\color{red}{FALSE}$ |
| $NUL$ | $\color{red}{FALSE}$ | $\color{blue}{NUL}$ | $\color{blue}{NUL}$ |
| $TRUE$ | $\color{red}{FALSE}$ | $\color{blue}{NUL}$ | $\color{green}{TRUE}$ |

Through similar reasoning, we can get to the conclusion that this should be the truth table for OR:
| OR | $FALSE$ | $NUL$ | $TRUE$ |
|:---:|:---:|:---:|:---:|
| $FALSE$ | $\color{red}{FALSE}$ | $\color{blue}{NUL}$ | $\color{green}{TRUE}$ |
| $NUL$ | $\color{blue}{NUL}$ | $\color{blue}{NUL}$ | $\color{green}{TRUE}$ |
| $TRUE$ | $\color{green}{TRUE}$ | $\color{green}{TRUE}$ | $\color{green}{TRUE}$ |

We can also notice that our trusted rule of $A$ OR $B$ = NOT $A$ NAND NOT $B$ holds true.\
Actually we can directly see the effects of it in the table: 
negating A or B flips the table around the vertical or horizontal axis, and negating the outputs changes all the truth values.

Similarly we could do the same reasoning for XOR, or use the formula that $A$ XOR $B$ = ($A$ AND NOT $B$) OR (NOT $A$ AND $B$), which gives out this truth table:
| XOR | $FALSE$ | $NUL$ | $TRUE$ |
|:---:|:---:|:---:|:---:|
| $FALSE$ | $\color{red}{FALSE}$ | $\color{blue}{NUL}$ | $\color{green}{TRUE}$ |
| $NUL$ | $\color{blue}{NUL}$ | $\color{blue}{NUL}$ | $\color{blue}{NUL}$ |
| $TRUE$ | $\color{green}{TRUE}$ | $\color{blue}{NUL}$ | $\color{red}{FALSE}$ |

And here we go, a full definition of ternary boolean algebra, every other law should stay true, and we can build all the gates we need.
