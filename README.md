# TERNAIRIS -101
### Or how to make a ternary computer
I don't think I need to introduce to anyone reading this the concept of binary numbers and how they are the basis of every computer since their invention.
But actually this would be wrong, binary was not always the only option for building computers.
In 1958, the Setun was developped in Moscow with balanced ternary as a basis.
And it is believed that a return to ternary could open up potential to improve computation, notably with LLM systems.

This, however, is entirely outside of my concern.
The main takaway I get is that it is possible to get 3 different states on a wire, as it's already been done before, and therefore I set out to see if I could manage to create an architecture that would make this work.

## Understanding extended boolean algebra
The first step is to understand how our usual gates would even work with 3 values.
Obviously it has to be some esoteric nonesense we've never seen before.
But actually binary computers can already simulate perfectly accurate ternary state logic if you introduce the $\color{blue}{Null}$ value.

For the sake of clarity, I will call this third state that the $\color{blue}{Null}$ value emulates so well in standard computers the $\color{blue}{UNKNOWN}$ truth state, and abreviate the truth values as $\color{red}{F}$, $\color{blue}{U}$, and $\color{green}{T}$.

First the simplest gate:
| | NOT |
|:-:|:-:|
| $F$ | $\color{green}{T}$ |
| $U$ | $\color{blue}{U}$ |
| $T$ | $\color{red}{F}$ |

There doesn't seem to be any other way to map this.

Let's take it up a notch with AND, and go exhaustively:
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

If we were to extend boolean algebra to an even bigger base, we could by simply noticing that those two tables are actually the same as a min and a max function.

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

I used some unoptimized Python code to generate and filter all the gates for ternary, and it turns out we now have 162 unique gate combinations, which is *a lot*, but thankfully they're mostly all rotations of each other.\
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
I will personally only implement balanced ternary numbers in this machine.
I have considered an unsigned integer system, but realized that I don't actually gain any advantage by having that option.

This is how the truth values map to balanced ternary:
- $\color{red}{F}$ $\implies$ -1
- $\color{blue}{U}$ $\implies$ 0
- $\color{green}{T}$ $\implies$ 1

The trits with the value of -1 will be written as Z for simplicity.

I will also introduce a few notations here:
- `0t` followed by a balanced ternary number, accepting {`Z`, `0`, `1`}
- `0n` followed by a nonary number (base 9), accepting {`W` to `Z`, `0` to `4`}
- `0p` followed by a icosiheptimal number (base 27), accepting {`N` to `Z`, `_`, `A` to `M`}

As you can notice, the higher half of the alphabet is dedicated to representing negative values, which works fine for ternary and nonary, because the positives are just numbers.
But it can get a little confusing for icosiheptimal, just remember that anything after `N` is `negative`.\
`_` was also chosen to represent the value 0, to avoid confusion with the letter `O`.

Also we can shorten that name to "hept". I am aware that icosiheptimal is not the correct name for that base, and it should really be septemvigesimal.
But if sextadecimal can rebrand itself as "hex", I can do what I want.

So if I have six trits holding the truth value of $\color{red}{F}$ $\color{blue}{U}$ $\color{red}{F}$ $\color{red}{F}$ $\color{green}{T}$ $\color{green}{T}$, then it will be interpreted as `0tZ0ZZ11` (-275).\
That same number would be written as `0nXW4` and `0pQV` in our other bases.

A neat property of balanced ternary is that negating a number is as simple as passing it through a NOT gate:
- -1 = NOT $\color{green}{T}$ = $\color{red}{F}$ = Z
- -0 = NOT $\color{blue}{U}$ = $\color{blue}{U}$ = 0
- -Z = NOT $\color{red}{F}$ = $\color{green}{T}$ = 1

### Addition
Finally something easy.
Or so you might think at first, because we still need to figure ot what the truth table of our adder will be, as well as how to arrange the gates to get the desired result.
No, sadly it isn't as simple as an XOR and an AND gate like in binary, it gets a little more involved.

Here is the truth table that we need:
| Sum | $F$ | $U$ | $T$ | | Carry | $F$ | $U$ | $T$ |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $F$ | $\color{green}{T}$ | $\color{red}{F}$ | $\color{blue}{U}$ | | $F$ | $\color{red}{F}$ | $\color{blue}{U}$ | $\color{blue}{U}$ |
| $U$ | $\color{red}{F}$ | $\color{blue}{U}$ | $\color{green}{T}$ | | $U$ | $\color{blue}{U}$ | $\color{blue}{U}$ | $\color{blue}{U}$ |
| $T$ | $\color{blue}{U}$ | $\color{green}{T}$ | $\color{red}{F}$ | | $T$ | $\color{blue}{U}$ | $\color{blue}{U}$ | $\color{green}{T}$ |

I did not yet find a way to implement a K-map, so it's gonna be a bit of a hassle every time.\
But thanks to our 162 base gates catalogue, we can determine that those are the corresponding gates:\
`ND(a⊕b) ⊕ D(Da*Db)`, `A(Aa*Ab) ⊕ D(a⊕b)`

<img width="1065" height="338" alt="image" src="https://github.com/user-attachments/assets/8e6f988b-45f7-4f91-9e9d-f0a6aed033ff" />

This gives us our third adder, which is not simply named that way for theming purposes, but also because you genuinely need three to get a full adder.

<img width="821" height="270" alt="image" src="https://github.com/user-attachments/assets/a3a7de95-c671-4ff0-b246-a4db8abc9f8d" />

### Multiplication
The multiplier circuit is actually fairly easy to build, despite how builky it is.\
Every trit of the scaling factor is either a 0, which does nothing, a 1, which adds a shifted copy of the number to multiply, or a Z, which adds a negative copy, and we know negation is just applying the NOT operation.\
All of this can be done very simply by inverting the scaling factor, and XOR-ing each trit with the whole copy of the number to multiply, then adding everything up:\
<img width="1787" height="642" alt="image" src="https://github.com/user-attachments/assets/5775f4b4-6e7c-4ee9-81a5-762f5cba0616" />

### Comparison
Because comparison typically has 3 outputs, we have the advantage over binary here since we can compress it in a single trit.

There is the truth table we need:
| CMP | $F$ | $U$ | $T$ |
|:---:|:---:|:---:|:---:|
| $F$ | $\color{blue}{eq}$ | $\color{red}{lt}$ | $\color{red}{lt}$ |
| $U$ | $\color{green}{gt}$ | $\color{blue}{eq}$ | $\color{red}{lt}$ |
| $T$ | $\color{green}{gt}$ | $\color{green}{gt}$ | $\color{blue}{eq}$ |

It can be created with those two tables:
| T0 | $F$ | $U$ | $T$ | | T1 | $F$ | $U$ | $T$ |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $F$ | $\color{red}{F}$ | $\color{red}{F}$ | $\color{red}{F}$ | | $F$ | $\color{green}{T}$ | $\color{blue}{U}$ | $\color{blue}{U}$ |
| $U$ | $\color{blue}{U}$ | $\color{green}{T}$ | $\color{red}{F}$ | | $U$ | $\color{green}{T}$ | $\color{red}{F}$ | $\color{blue}{U}$ |
| $T$ | $\color{blue}{U}$ | $\color{blue}{U}$ | $\color{red}{F}$ | | $T$ | $\color{green}{T}$ | $\color{green}{T}$ | $\color{green}{T}$ |

Composed with a gate that has those properties:
| T2 | $F$ | $U$ | $T$ |
|:-:|:-:|:-:|:-:|
| $F$ | $x$ | $\color{red}{F}$ | $\color{blue}{U}$ |
| $U$ | $\color{red}{F}$ | $x$ | $\color{green}{T}$ |
| $T$ | $\color{blue}{U}$ | $\color{green}{T}$ | $x$ |

Which happens to be the same as our addition.

<img width="848" height="246" alt="image" src="https://github.com/user-attachments/assets/31050f36-7719-44ca-91a5-ff3124c04431" />

We can then scale to a full Tryte by simply making sure once we get the information we need in a high trit, we block any other calculations.

<img width="1021" height="956" alt="image" src="https://github.com/user-attachments/assets/d39a3e9c-0af3-4b10-9272-06990924fd5b" />

## Memory
Nothing fancy in terms of construction, but there are some naming conventions that I would like to introduce:
- A trit is the basic unit of data that can be stored in a single wire, equivalent to a binary bit
- 3 trits make a nittle
- 3 nittles make a Tryte
- $3^7$ = 2187 = 2 k
- $3^{14}$ = $3^7$ \* $3^7$ = 2 k \* 2 k = 4 M

And so on.

I will say I would have prefered to make a full Tryte address RAM, but the program was physically not able to handle it, so I had to limit myself to 2 nittles of addressing, which still allows for storing 729T of data in 1T chunks.
Here is the circuit that allows for selecting from 3 options :\
<img width="660" height="525" alt="image" src="https://github.com/user-attachments/assets/26e18a50-93df-45b7-afde-4f6fd1066857" />

And the construction of the RAM, remember that we need to handle every wire individually:\
<img width="1641" height="822" alt="image" src="https://github.com/user-attachments/assets/491c9017-f4fb-4843-aae4-e1259c9bd131" />\
<img width="1634" height="824" alt="image" src="https://github.com/user-attachments/assets/75fbee7d-d0f5-4e99-bcd7-d850f2b74436" />\
<img width="1051" height="914" alt="image" src="https://github.com/user-attachments/assets/f8d2ab2e-dc56-4b14-b220-7e8cdde8e93a" />

## Character set
This section will definitely move, but I just want it to be added to the document for now.

I cannot simply use the Unicode standard to represent characters on this computer, so I had to implement my own character set.
It uses 5 trits of addressing, which means a limit of 243 characters, and I used this to try and map a subset of Code Page 437 onto it.

This is CP437:\
<img width="304" height="144" alt="image" src="https://github.com/user-attachments/assets/e71ca909-1206-4db4-92c1-7f4102ed9adb" />

And this is my character set:
```
    000 001 002 010 011 012 020 021 022 100 101 102 110 111 112 120 121 122 200 201 202 210 211 212 220 221 222
00  NUL ♥︎   ♦︎   ☺︎   •   ○   ♂︎   ♪   ↨   ↓   ▼   ◄   ←   ↔︎   →   ►   ▲   ↑   ↕︎   ♫   ♀︎   ◙   ◘   ☻   ♣︎   ♠︎   ·
01  %   -   \   «   ^   `   ,   ¿   ¡   {   [   (   <   =   >   )   ]   }   !   ?   .   '   "   »   /   +   *
02  α   ß   Γ   π   Σ   σ   µ   τ   Φ   Θ   Ω   δ   φ   ε   a   b   c   d   e   f   g   h   i   j   k   l   m
10  n   o   p   q   r   s   t   u   v   w   x   y   z   _   A   B   C   D   E   F   G   H   I   J   K   L   M
11  N   O   P   Q   R   S   T   U   V   W   X   Y   Z   0   1   2   3   4   5   6   7   8   9   æ   Æ   å   Å
12  â   ä   à   á   Ä   ê   ë   è   é   É   î   ï   ì   í   ÿ   ô   ö   ò   ó   Ö   û   ü   ù   ú   Ü   ç   Ç
20  ☼   ≡   &   ∩   ∞   ÷   #   £   ¢   §   ∟   ▬   ;   ‼︎   :   |   ¬   ¶   $   €   ¥   ±   °   √   @   ≈   ⌂
21  ╣   ╬   ╠   ╗   ╦   ╔   ╖   ╥   ╓   ▄   ▌   sp  ░   ▒   ▓   █   ▐   ▀   ╒   ╤   ╕   ┌   ┬   ┐   ├   ┼   ┤
22  ■   ║   ═   ╝   ╩   ╚   ╜   ╨   ╙   ╢   ╫   ╟   ñ   ~   Ñ   ╞   ╪   ╡   ╘   ╧   ╛   └   ┴   ┘   ─   │   nbsp
```
Which allows us to do stuff like this:
```
╔═╗                                                            ╔═╗
╚═════════════════════════════════════════════════════════════ ║ ╝
  ║                                                            ║
  ║  ▓▓▓▓▓▓▒▓▓▓▓▒▓▓▓▓▓▒ ▓▓▓▒ ▓▓▒ ▓▓▓▓▒ ▓▓▓▓▒▓▓▓▓▓▒ ▓▓▓▓▒ ▓▓▓▒  ║
  ║    ▓▓▒  ▓▓▓▒ ▓▓▒ ▓▓▒▓▓▓▓▒▓▓▒▓▓▒ ▓▓▒ ▓▓▒ ▓▓▒ ▓▓▒ ▓▓▒ ▓▓▒    ║
  ║    ▓▒░  ▓▒░  ▓▒▒▒▒░ ▓▒░▓▓▒▒░▓▒▒▒▒▒░ ▓▒░ ▓▒▒▒▒░  ▓▒░   ▓▒░  ║
  ║    ▒▒░  ▒▒▒▒░▒▒░ ▒▒░▒▒░ ▒▒▒░▒▒░ ▒▒░▒▒▒▒░▒▒░ ▒▒░▒▒▒▒░▒▒▒░   ║
  ║                                                            ║
  ║                 ▓▓▓▒       ▓▓▓▓▒      ▓▓▓▒                 ║
  ║              ▓▓▓▒▓▓▒      ▓▓▒ ▓▓▒      ▓▓▒                 ║
  ║                  ▓▒░      ▓▒░ ▓▒░      ▓▒░                 ║
  ║                 ▒▒▒▒░      ▒▒▒▒░      ▒▒▒▒░                ║
  ║                                                            ║
╔ ║ ═════════════════════════════════════════════════════════════╗
╚═╝                                                            ╚═╝
```
And that's pretty cool

This character set has a few intereting properties, first of all the characters `0` to `9` all map to the numerical values of 0 to 9, and following that logic you can see how the letters of the alphabet are arranged to agree with the hept system.\
Furthermore, numbers and letters aren't the only ones to correspond when negated, you can see that for most characters, negating the bottom 3 trits actually gives back an inverted or corresponding character.\
It's easiest to see with the pipes at the bottom, or on the first two lines of symbols, particularly this block: `{[(<=>)]}`. The pipes also have the property that negating the lowest trit mirrors them around the vertical axis.
