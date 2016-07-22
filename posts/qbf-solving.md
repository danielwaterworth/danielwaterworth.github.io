% QBF Solving
% Daniel Waterworth
% July 22, 2016

## What is QBF solving?

> ![](https://imgs.xkcd.com/comics/travelling_salesman_problem.png)

> *[travelling salesman problem - xkcd 399](https://xkcd.com/399/)*

If you are a programmer, it is likely that you have at least a vague
understanding of what an [NP-complete][0] problem is; even if it's just, "those
really hard ones".

What you may not know, is that, although NP-complete problems are difficult in
general, a significant proportion of them are easy in practice and there are
tools, called SAT solvers, that can efficiently solve this subset.

SAT solvers allow you to ask questions like, "does there exist an x such that y
is true?". This is incredibly useful for hardware or software verification
since you can effectively ask, "is there an input that causes this invariant to
break?".

However, there are caveats. Since SAT solvers attempt to solve the
[SAT problem][1], it must be possible to translate your problem into a
[propositional expression][2]. In practice this means that you can only verify
properties about programs that don't have loops or hardware that forms
[combinational circuits][3]. (If you unroll your loops up to some fixed bound,
you are doing [bounded model checking][4]).

Another limitation is that, although you can pose questions like, "does there
exist an x such that y is true?" or "for all x is y true?", you can't ask
questions like "does there exist an x such that for all y, z is true?".

It turns out, although NP-complete problems are hard, there are more difficult
classes of problems.

![](../images/thinking.png)

## Puzzles vs Games

There is an interesting trend amongst the kinds of puzzles that people like to
solve. Many of them are [NP-complete][5], and conversely many NP-complete
problems can be made into interesting puzzles.

In the same manner, many two-player games are [PSPACE-complete][6]. PSPACE is
the class of problems that can be solved using polynomial space and
PSPACE-complete problems are the hardest problems in PSPACE. PSPACE contains
NP, so it is more difficult.

Just as the canonical NP-complete problem is SAT, the canonical PSPACE-complete
problem is [QBF][7] and so a QBF solver is a tool to help you win two-player
games (with caveats).

QBF has the same limitation as SAT in that, we still can't express loops, so
our two-players games must have a polynomial number of turns. In other words,
we can devise strategies for tick-tack-toe, but not chess.

PSPACE problems are hard, but there are yet more difficult classes of problems.
For example, chess belongs to a class called [EXPTIME-complete][8].

## Back to reality

Whilst SAT solvers have been widely adopted for real problems, QBF solvers have
not. Progress is being made, but we simply don't yet have QBF solvers that work
effectively on large problems.

This is a shame since, if we did, we could do things like program synthesis;
"does there exist a program, such that for all inputs, these invariants hold?".

For my part, I've have been writing [my own solver][9], which I hope to blog
more about in the future.

[0]: https://en.wikipedia.org/wiki/NP-complete
[1]: https://en.wikipedia.org/wiki/Boolean_satisfiability_problem
[2]: https://en.wikipedia.org/wiki/Propositional_formula
[3]: https://en.wikipedia.org/wiki/Combinational_logic
[4]: https://en.wikipedia.org/wiki/Model_checking
[5]: https://en.wikipedia.org/wiki/List_of_NP-complete_problems#Games_and_puzzles
[6]: https://en.wikipedia.org/wiki/PSPACE-complete
[7]: https://en.wikipedia.org/wiki/True_quantified_Boolean_formula
[8]: https://en.wikipedia.org/wiki/EXPTIME-complete
[9]: https://github.com/DanielWaterworth/qbf-rust
