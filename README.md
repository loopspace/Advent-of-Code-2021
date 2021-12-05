# Advent of Code 2021

I thought I'd have a go at the [Advent of Code
2021](https://adventofcode.com/).
I've learnt a bit about programming in LaTeX3 this year, so decided to
practise a bit more by doing the AoC in LaTeX3.

I've decided to work with a `dtx` file so that I can contain the
documentation alongside the code.
To compile it, therefore, takes two steps:

~~~
tex advent_code.dtx
pdflatex advent_code.dtx
~~~

The first creates the `advent.sty` file which is then included back in
on the second run.
(It is possible just to run `pdflatex` twice but the first run might
produce complaints.)

