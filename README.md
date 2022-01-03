# Advent of Code 2021

I thought I'd have a go at the [Advent of Code
2021](https://adventofcode.com/).
I've learnt a bit about programming in LaTeX3 this year, so decided to
practise a bit more by doing the AoC in LaTeX3.

I've decided to work with a `dtx` file so that I can contain the
documentation alongside the code.
To compile it, run:

~~~
pdflatex advent_code.dtx
~~~

## Using LaTeX3

One of the strengths of LaTeX for me is the programming language that
is built in.
I like the fact that I can do programmy-like stuff in my documents,
taking advantage of the computer's ability to do stuff that I'd rather
not without complaining.
Since, usually, my goal is to produce a document then I've learnt to
work with and appreciate the fact that it is a macro language.
The paradigm of "_this_ command gets replaced by _this_ text in the final
document" is one that I find particularly apt when writing documents.
_Most_ of the time, that's sufficient.

However, sometimes I need something a bit more like standard
programming.
Not quite enough to warrant generating my document from a more
standard programming language, but definitely enough to get frustrated
with `\expandafter`.
I've found that [LaTeX3](https://www.latex-project.org/latex3/)
provides what I need to fill that gap.

Indeed, my [spath3](https://ctan.org/pkg/spath3) library is written
almost entirely in LaTeX3 and I find it much easier to maintain and
work with than some of my earlier packages which aren't.

Nevertheless, LaTeX3 is still built on TeX and so still has that macro
paradigm sitting underneath.
For all it pretends to be a proper programming language, everything
ends up working through the medium of macros, and the effect of
expansion can never be ignored.

So one reason for taking up the challenge of the Advent of Code in
LaTeX3 was to get a better feel for that boundary.
Effectively, I want more information to help me answer the question
"Why not just write this in `lua`?" (or other programming language -
the existence of LuaTeX means that `lua` is a not-unreasonable
choice).

I feel I gave it a good go, and got quite far.
The main reason I haven't finished is time.
The Christmas break is nearly over and although I would quite like to
finish this, there will be other calls on my time so this seems an
appropriate stage to sum up.

## What Went Well

There were challenges where my code _just worked_.
That was quite surprising.

Parsing input is something that was more simple than I was expecting.
Task 5 is a good example of this.
A typical line of the input for that task looks like:

```
141,530 -> 141,630
```

So the function that I use to parse that input was:

```
\cs_new_protected_nopar:Npn \@@_grid_parse:w #1,#2->#3,#4\q_stop
{
  \tl_set:Nn \l_@@_tmpb_tl {#1}
  \tl_set:Nn \l_@@_tmpc_tl {#2}
  \tl_set:Nn \l_@@_tmpd_tl {#3}
  \tl_set:Nn \l_@@_tmpe_tl {#4}
}
```

The input line was stored in a temporary variable so the call was:

```
\exp_last_unbraced:NV \@@_grid_parse:w \l_@@_tmpa_tl \q_stop
```

Task 13 had another nice feature.
The result was a series of coordinates to mark, so using TikZ to
render the final solution seemed quite a nice touch.

Although it doesn't quite count as a "What went well", Task 15 had a
nice aspect to the algorithm.
I found that the full algorithm took some time, in particular as it
had an open-ended terminating condition, so I put in place file
caching for each stage, thus taking advantage of the compulsive habits
of a LaTeX writer to recompile their document after every sentence.

## Even Better If

It's always easier to spot the "even better if"s than the "what went
well"s, so the fact that this list is longer than the above shouldn't
be taken as anything.

### Functions Aren't ... Functions

One of the difficulties with LaTeX3 (and TeX) "functions" is that they
don't work in the same way as functions in a more usual programming
language.
Two of the most obvious are the lack of scoping and return values.

Because TeX works by macro replacement, its standard method for
returning a value is to leave it in the stream, but to then assign
this to a variable requires the function to be expandable.
If the function isn't expandable, the alternative method is to pass in
a suitable variable and have the function assign it at its end.
However, there are a variety of different ways that the assignment
might happen: it might be local or global, and it might be that the
variable is one of the inputs or not.

Then there's scoping.
Functions aren't naturally scoped.
That can be fixed by starting and ending with an explicit group, but
then the issue of assignment returns.
If the assignment is meant to be local, then it has to happen outside
the function scope but not globally.
So this leads to the problem of getting a value out of a group.

My solution to this is one that I came up with when writing the
`spath3` library.
When writing a function, I have a core function that does all the
work.
This is enclosed in a group and at its end it globally sets a variable
to its return value, according to its type.
Then there are wrapper functions which are designed to organise their
input for the core function and assign the output accordingly.
These deal with the difference with local and global assignments, with
redefining one of the inputs, and other variations.

I brought that concept over into this code and it worked quite well.

### Integer Arithmetic

A few tasks ran into the issue of arithmetic overflow.
I didn't need a full `bigint` implementation, so I went for something
a bit more rough and ready.
When the issue was with addition, I used two integer registered and
overflowed at 100000000.
This meant that typesetting the final answer could be done by
concatenation (with leading zeros).
For multiplication, the overflow was at 10000.
This got a bit complicated when I needed an array of large integers,
but it was manageable.

The one that "got away" was Day 22 which needs all of addition,
subtraction, and multiplication.
For this, I've implemented a slightly more structured `bigint` but as
yet I haven't gotten the right answer for that task.

### Complex Data Types

Some of the tasks would have been best suited with a two dimensional
array of some sort.
For two dimensional arrays of integers then it's not too hard to
unravel it to a one dimensional `intarray` (though the 1-based aspect
makes it slightly more intricate than it needed to be), except that
many of the tasks where this was needed also needed considering the
neighbours of a cell, and that requires complicated bookkeeping to
know when a cell is on an edge.

Having a sequence of sequences, for example, is something that I'm not
sure how best to handle.
Whether the elements of the container sequence should be the child
sequences themselves or pointers to them, but then that latter
introduces its own issues with names.

Property lists and sequences, being the more flexible data types, also
suffer from the issue that they can't be passed-by-value to a
function.
This means that even with scoping then there is potential for name
clashes with functions.

### Variable Names

In LaTeX3, the convention is that variables have names like:

```
\l_<module name>_<variable name>_<variable type>
```

This can lead to really, really long variable names.
That's even with the module name replacement from the `l3doc` class.
The difficulty with this is that it makes the code very verbose by
default.
When combined with the fact that arguments to functions are not
clearly delimited, this can make code hard to read.

### DTX

I used a `dtx` file to document my code at the same time as writing
it.
I wouldn't do that again.
I'd just write the code in a separate file and include it (not
necessarily as a `sty` file, probably just a `tex` file).
The odd way that `dtx` implements commenting within code just adds to
the unreadability.
It makes sense for packages, but this isn't a package.
