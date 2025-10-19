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

For the sake of clarity, I will call this third state that the $\color{blue}{NUL}$ value emulates so well in standard computers the $\color{blue}{UNKNOWN}$ truth state, and abreviate the truth values as $\color{red}{F}$, $\color{blue}{U}$, and $\color{green}{T}$.

First the simplest gate:
- NOT $\color{red}{F}$ $\implies$ $\color{green}{T}$
- NOT $\color{green}{T}$ $\implies$ $\color{red}{F}$
- NOT $\color{blue}{U}$ $\implies$ $\color{blue}{U}$

There doesn't seem to be any other way to map this.

Let's take it up a notch with AND, and go exaustively:
- $\color{red}{F}$ AND $\color{red}{F}$ $\implies$ $\color{red}{F}$
- $\color{red}{F}$ AND $\color{green}{T}$ $\implies$ $\color{red}{F}$
- $\color{green}{T}$ AND $\color{green}{T}$ $\implies$ $\color{green}{T}$

So far, no surprises.
- $\color{blue}{U}$ AND $\color{blue}{U}$ $\implies$ $\color{blue}{U}$

I mean what else would it be ?
We have no information going in so we should expect no information comming out.
- $\color{red}{F}$ AND $\color{blue}{U}$ $\implies$ $\color{red}{F}$

Seems logical, $\color{red}{F}$ AND anything should be $\color{red}{F}$, after all.
- $\color{green}{T}$ AND $\color{blue}{U}$ $\implies$ $\color{blue}{U}$

That is a bit more interesting, but it still makes some sense.
We have no way of assigning a meaningful value to the output since the only two straight forward cases we know of are that either one of the inputs is $\color{red}{F}$, or both of the inputs are $\color{green}{T}$.

So we arrive at this truth table:
| AND | $F$ | $U$ | $T$ |
|:---:|:---:|:---:|:---:|
| $F$ | $\color{red}{F}$ | $\color{red}{F}$ | $\color{red}{F}$ |
| $U$ | $\color{red}{F}$ | $\color{blue}{U}$ | $\color{blue}{U}$ |
| $T$ | $\color{red}{F}$ | $\color{blue}{U}$ | $\color{green}{T}$ |

Through similar reasoning, we can get to the conclusion that this should be the truth table for OR:
| OR | $F$ | $U$ | $T$ |
|:---:|:---:|:---:|:---:|
| $F$ | $\color{red}{F}$ | $\color{blue}{U}$ | $\color{green}{T}$ |
| $U$ | $\color{blue}{U}$ | $\color{blue}{U}$ | $\color{green}{T}$ |
| $T$ | $\color{green}{T}$ | $\color{green}{T}$ | $\color{green}{T}$ |

We can also notice that our trusted rule of $A$ OR $B$ = NOT $A$ NAND NOT $B$ holds true.\
Actually we can directly see the effects of it in the table: 
negating A or B flips the table around the vertical or horizontal axis, and negating the outputs changes all the truth values.

Similarly we could do the same reasoning for XOR, or use the formula that $A$ XOR $B$ = ($A$ AND NOT $B$) OR (NOT $A$ AND $B$), which gives out this truth table:
| XOR | $F$ | $U$ | $T$ |
|:---:|:---:|:---:|:---:|
| $F$ | $\color{red}{F}$ | $\color{blue}{U}$ | $\color{green}{T}$ |
| $U$ | $\color{blue}{U}$ | $\color{blue}{U}$ | $\color{blue}{U}$ |
| $T$ | $\color{green}{T}$ | $\color{blue}{U}$ | $\color{red}{F}$ |

And here we go, a full definition of ternary boolean algebra, every other law should stay true, and we can build all the gates we need.

## Dimensional constraint issues
### New dimensions
Actually if you look at the truth tables we got, you might notice two pretty big related issues:
- No combination of NOT, AND, OR, and XOR can yield a result other than $\color{blue}{U}$ when evaluating $\color{blue}{U}$ with $\color{blue}{U}$.
- No combination of NOT, AND, OR, and XOR can yield a $\color{blue}{U}$ when evaluating only $\color{green}{T}$ and $\color{red}{F}$ together.

The gates we have simply do not have enough dimensions to reach every truth table we need, and trust me, we will need them.

So are we stuck ?
Well unless we create new gates, this is the end of the adventure for us.\
So let's do just that, and create new gates.

We already have NOT defined as the gate that swaps $\color{red}{F}$ and $\color{green}{T}$ and leaves $\color{blue}{U}$ unchanged.\
Symetrically, let's define AVER, the gate that swaps $\color{red}{F}$ and $\color{blue}{U}$ and leaves $\color{green}{T}$ unchanged,\
as well as DENY, the gate that swaps $\color{blue}{U}$ and $\color{green}{T}$ and leaves $\color{red}{F}$ unchanged.

Why those names ? Aver means to forcibly state something as true, and deny means to forcibly state something as false, so it makes sense that they would be the gates that respectively force $\color{green}{T}$ and $\color{red}{F}$ to stay in place.\
I do not claim to be the first one to come up with those gates (I would actually be surprised), and I admit I did not look long for if they were already thought of by someone else and given different names.
But this is what we're gonna roll with from now on.

### Theoretical analysis
Now that we can switch around the truth values any way we want, let's see how they interact with each other.\
This is a diagram that shows each node as an ordering of the truth values, connected by the gate that bridges the gap:\
<img width="809" height="512" alt="image" src="https://github.com/user-attachments/assets/c4bf9e95-d10a-40e4-879e-68fd6ccb7286" />

We can notice multiple things from this.
1. The three colored nodes and the three white nodes are all rolled versions of each other.
2. Starting from any node, you can reach any other node by applying at most two swap gates.
3. There are multiple paths that lead from one node to another

From this, we can extend the usual law of `¬¬x = x`.\
To clarify notation, N is NOT, A is AVER, D is DENY, and ND is ~neurodivergent~ NOT(DENY), so apply the DENY gate first, and then the NOT gate.
- NN = AA = DD = id
- NA = AD = DN
- AN = DA = ND

Which means we can now simplify any combination of swap gates into at most two.\
Ex:\
AADNDNNDDA = **\[AA\]** DND **\[NN\]** **\[DD\]** A = DNDA = D **\[ND\]** A = D **\[DA\]** A = **\[DD\]** **\[AA\]** = id\
If you follow the diagram, you can see that starting from the top node (the canonical ordering) you do end up back where you started.

### New basic combinations
With new ways to swap the data, we now need to catalogue all the new basic gate combinations we have access to.\
Binary has 8 that are unique up to switching $A$ and $B$:
- $A$ AND $B$
- NOT $A$ AND $B$
- $A$ NAND $B$
- $A$ OR $B$
- NOT $A$ OR $B$
- $A$ NOR $B$
- $A$ XOR $B$
- $A$ XNOR $B$

I used some unoptimized Python code to generated and filter all the gates for ternary, and it turns out we now have 162 unique gate combinations, which is **a lot**, but thankfully they're mostly all rotations of each other.\
Armed with the full list as a reference, we now have everything we need to actually connect some wires.

## Actually building a computer
### About the simulator
All of that theory is nice and all, but we still need to even construct those gates.\
I will be using the DLS program for my purposes. It has the really interesting feature that wires can be $\color{green}{on}$, $\color{red}{off}$, or $\color{blue}{undefined}$.\
An $\color{blue}{undefined}$ wire is simply a wire that is either not connected to anything, or has been cut by a switch, the purpose being mainly to cut the current in a wire so you can avoid conflicts when sending data to a bus.

But it just so happens that this third wire state behaves exactly as we would expect the $\color{blue}{U}$ truth value to, with the NOT, AND, OR, and XOR gates that the program gives us.

There are some finicky details we need to keep in mind however, mainly memory constraints when it comes to what the simulator can handle (sadly even a few kilos of custom RAM are too many circuits at once for the poor program), and the fact that when merging single wires into a more manageable data wire of bigger size, the $\color{blue}{undefined}$ status propagates to the entire merged wire, which leads to this ridiculous situation where the output **should** obviously be 3:\
<img width="520" height="373" alt="image" src="https://github.com/user-attachments/assets/889c657c-04da-4b2b-b527-bde8d21b9b06" />

No matter, it just means we need to manage every wire individually, which is cumbersome but far from impossible.

### First steps
So let's build those gates.\
As stated previously the default NOT, AND, OR, and XOR all behave as expected, so we just need to construct AVER and DENY, and here they are:\
<img width="687" height="249" alt="image" src="https://github.com/user-attachments/assets/8aeb0176-e3bf-479b-a501-5eaf47fa1dcc" />\
<img width="694" height="246" alt="image" src="https://github.com/user-attachments/assets/3ca367ea-01d9-45e5-a65f-85b18ca69d92" />

You can notice there is an extra component that might seem unfamiliar, it simply takes in a wire signal, if it's $\color{green}{on}$ or $\color{red}{off}$ it lets it pass through, but if it's $\color{blue}{undefined}$ it sets it to a hardcoded value of $\color{green}{on}$ or $\color{red}{off}$.

Another easy circuit we can tackle now is a single trit memory cell, which looks like this:\
<img width="984" height="370" alt="image" src="https://github.com/user-attachments/assets/c69f4e1a-1d72-4926-905e-06aeefdc53fb" />


## Arithmetics
### Numbering systems
So with ternary, we actually have a few options on how to handle integers.
I will personally implement two systems in my machine, an unsigned system, and a balanced system.

The unsigned integers will be as expected:
- $\color{red}{F}$ $\implies$ 0
- $\color{blue}{U}$ $\implies$ 1
- $\color{green}{T}$ $\implies$ 2

The balanced system will be the default for representing negative numbers, instead of using 3s complement:
- $\color{red}{F}$ $\implies$ -1
- $\color{blue}{U}$ $\implies$ 0
- $\color{green}{T}$ $\implies$ 1

The trits with the value of -1 will be written as N for simplicity.

I will also introduce a few notations here:
- `0u` followed by an unsigned ternary number, accepting {0, 1, 2}
- `0t` followed by a balanced ternary number, accepting {N, 0, 1}
- `0n` followed by a nonary number (base 9), accepting {0 to 8}
- `0s` followed by a septemvigesimal number (base 27), accepting {`_`, `A` to `Z`}

Note that in the septemvigesimal system, `_` represents the value 0, and `A` through `Z` represent values 1 through 26.

So if I have six trits holding the truth value of $\color{red}{F}$ $\color{blue}{U}$ $\color{red}{F}$ $\color{red}{F}$ $\color{green}{T}$ $\color{green}{T}$, then it could be interpreted as either `0u010022` (89) or `0tN0NN11` (-275).\
One interesting property is that, for an $n$ trit representation, the unsigned interpretation of the number will always be exactly $\frac{3^n-1}{2}$ more than its balanced counterpart, which can be represented as a ternary number composed of $n$ 1s in a row.\
This means you can transpose a number from one system to another in two equivalent ways:
- assume the number is in unsigned representation and add or subtract a number made of $n$ $\color{blue}{U}$
- assume the number is in balanced representation and add or subtract a number made of $n$ $\color{green}{T}$

Do note that transposing a negative number from balanced to unsigned will force it to be represented in 3s complement, and vice-versa, which is to be avoided
