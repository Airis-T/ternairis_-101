# Making a ternary computer
I don't think I need to introduce to anyone reading this the concept of binary numbers and how they are the basis of every computer since their invention.
But actually this would be wrong, binary was not always the only option for bilding computers.
In 1958, the Setun was developped in Moscow with balanced ternary as a basis.
And it is believed that a return to ternary could open up potential to improve computation, notably with LLM systems.

This however, is entirely outside of my concern.
The main takaway I get is that it is possible to get 3 different states on a wire, as it's already been done before, and therefore I set out to see if I could manage to create an architecture that would make this work.

## Understanding extended boolean algebra
The first step is to understand how our usual gates would even work with 3 values.
Obviously it has to be some esoteric nonesense we've never seen before.
But actually binary computers can already simulate perfectly accurate ternary state logic if you introduce the NUL value.

First the simplest gate:
- NOT FALSE => TRUE
- NOT TRUE => FALSE
- NOT NUL => NUL

There doesn't seem to be any other way to map this.

Let's take it up a notch with AND, and go exaustively:
- FALSE AND FALSE => FALSE
- FALSE AND TRUE => FALSE
- TRUE AND TRUE => TRUE

So far, no surprises.
- NUL AND NUL => NUL

I mean what else would it be ?
We have no information going in so we should expect no information comming out.
- FALSE AND NUL => FALSE

Seems logical, FALSE AND anything should be FALSE, after all.
- TRUE AND NUL => NUL

That is a bit more interesting, but it still makes some sense.
We have no way of assigning a meaningful value to the output since the only two straight forward cases we know of are that either one of the inputs is FALSE, or both of the inputs are TRUE.

So we arrive at this truth table:
| AND | FALSE | NUL | TRUE |
|:---:|:---:|:---:|:---:|
| FALSE | FALSE | FALSE | FALSE |
| NUL | FALSE | NUL | NUL |
| TRUE | FALSE | NUL | TRUE |

Through similar reasoning, we can get to the conclusion that this should be the truth table for OR:

| AND | FALSE | NUL | TRUE |
|:---:|:---:|:---:|:---:|
| FALSE | FALSE | NUL | TRUE |
| NUL | NUL | NUL | TRUE |
| TRUE | TRUE | TRUE | TRUE |
