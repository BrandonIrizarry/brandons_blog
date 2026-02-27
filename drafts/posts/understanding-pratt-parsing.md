+++
title = "Understanding Pratt Parsing"
tags = ["programming languages"]
date = 2025-12-02

summary = """

A post I had written about Pratt parsing, in the context of a \
programming language I was designing at the time.

"""

+++

# Table of Contents

+ [Introduction](#introduction)
+ ["It's like a burrito"](#its-like-a-burrito)
+ [Down To Brass Tacks](#down-to-brass-tacks)
+ [Wanting More](#wanting-more)

<a id="introduction"></a>

# Introduction

I've forgotten how I came across Pratt parsing specifically. I had
been working on an interpreter for a programming language based on the
one loosely described in Greg Michaelson's *An Introduction to
Functional Programming Through Lambda Calculus*. I had managed to
implement simple arithmetic, and even extended the basic lambda
calculus spec with assignment expressions (a feat which I was very
proud of.)

However, some parts of my implementation felt a bit hacky (for
example, how I had implemented `letrec`), and my implementation of
lazy evaluation, while mostly complete, ultimately turned out to be
buggy.

At first, I decided to rewrite the project from scratch. One major
guiding factor was to narrow the scope of the project, borrowing some
advice from [Zed Shaw](https://learncodethehardway.com/blog/32-very-deep-not-boring-beginner-projects/). I got around to writing a new tokenizer,
and then I was on to writing the parser. My initial parser used the
recursive descent technique, taking advantage of the simplified lambda
calculus grammar used in Michaelson's text (for example, parentheses
are always used for application terms there.)

This time though, I wanted to try something different. And so,
rummaging through the internets, I stumbled across Pratt parsing.

<a id="its-like-a-burrito"></a>

# "It's like a burrito"

Understanding Pratt parsing ended up being much harder than I
expected. I ended up searching through a bunch of examples online:

1. [Alex Kladov's](https://matklad.github.io/2020/04/13/simple-but-powerful-pratt-parsing.html) Rust-based tutorial.
2. [Eli Bendersky's](https://eli.thegreenplace.net/2010/01/02/top-down-operator-precedence-parsing) Python-based tutorial. 
3. Bob Nystrom's [intro](https://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/) to the subject, as well as the relevant
   chapter in his [Crafting Interpreters](https://craftinginterpreters.com/compiling-expressions.html).
4. Vaughan Pratt's [original paper](https://tdop.github.io/). Shout-out to the legend who
   put this up as a GitHub Pages site!
5. Douglas Crockford's celebrated [article](https://crockford.com/javascript/tdop/tdop.html) on the subject deserves
   honorable mention, though my JavaScript is currently rusty and so I
   didn't look into it in any depth.

Alex Kladov in his post calls Pratt parsing the "monad tutorial of
syntactic analysis". As I had recently become familiar with the
concept of "monad" in a [philosophical](https://en.wikipedia.org/wiki/Monad_(philosophy)) sense, I aksed ChatGPT
where the reference comes from: it turns out that it's an inside joke
involving *Haskell* monads. I'm not an expert, but my readings have
given me enough of an inkling to see the connection: Pratt parsing
isn't a discrete "thing" with exactly one shape: it's more of a
technique, if you will—a design pattern—which can assume various
manifestations.

A good example of this is iteration, because we all recognize it when
we see it, even when we don't know the language—but no one syntactic
construction sufficiently defines it. For-loops, while-loops, Python
generator functions, and [optimized tail-recursive functions](https://mitp-content-server.mit.edu/books/content/sectbyfn/books_pres_0/6515/sicp.zip/full-text/book/book-Z-H-11.html#%_sec_1.2.1 ) all
count as iteration.

After struggling for some time, I finally managed to distill the
essence of the algorithm, which I present here as pseudocode:

```
parse(level):
    t ← next(stream)
    acc ← nud_dispatch(t)

    while level < precedence(peek(stream)):
        t ← next(stream)
        acc ← led_dispatch(t, acc)

    return acc
```

Unwrapping what this does exactly is a surprisingly nuanced task,
precisely because the algorithm is more about technique than
structure. Because of this, I won't pretend to be up to the task
here. Nevertheless, a brief synopsis is warranted.

There is a global `stream` of tokens, such that `next(stream)`
consumes and returns the next token, and `peek(stream)` returns the
next token without consuming it.  The function `nud_dispatch`
interprets `t` as a *null denotation*, or "nud" for short, and
initializes `acc`. The function `led_dispatch` interprets `t` as a
*left denotation*, or "led" for short. It accumulates a value into
`acc` using the existing value of `acc` (hence the choice of name.)
Both dispatch functions call `parse` recursively in all but the most
trivial cases.

When `peek`ing the stream reveals a token with higher precedence than
the current `level`, the while loop exits and `acc` is returned.

The algorithm is initialized by calling `parse(0)`.

<a id="down-to-brass-tacks"></a>

# Down To Brass Tacks
 
My approach was to take Eli Bendersky's full source code at the bottom
of his post, and start chiseling away at it. What I ended up with was
the same simple arithmetic calculator, only with a different
architecture: I moved away from Eli's object-oriented approach towards
something closer to the formulation given in the previous section.

In the end, I was amazed at how simple and robust the actual
implementation turned out to be! I feel that what I came up with (at
this stage, anyway) is arguably simpler than even many of the examples
I initially came across: for example, it isn't necessary to add space
between precedence levels (10, 20, etc.), since you can use an enum to
take care of any ordering needed. Also, using even and odd precedence
levels (for handling right associativity) is unnecessary. For example,
say you have precedence levels `MULTIPLICATION = 2` and
`EXPONENTIATION = 3`. The algorithm cleverly avoids clashing
`EXPONENTIATION-1` with `MULTIPLICATION` when enforcing right
associativity for exponentiation. I found this to be one of the more
remarkable aspects of the algorithm.

<a id="wanting-more"></a>

# Wanting More

To be fair, my calculator app technically doesn't parse arithmetic
expressions: it evaluates them wholesale. This is OK: instead of
accumulating an AST, I'm accumulating an arithmetic result.

Because of how compelling the calculator app turned out to be, I
decided to stop work on the lambda calculus project, and instead work
on expanding the calculator into a full-blown programming language,
albeit a simple one. I've already made progress in this direction: in
addition to arithmetic (including trig functions!), the application
currently supports variable assignment.

Ideally, I'd like something with
<br></br>

+ Booleans
+ Conditionals
+ Loops
+ Functions
+ Proper lexical scoping, even for conditional and loop blocks
<br></br>
I'll see how many of these I manage.





