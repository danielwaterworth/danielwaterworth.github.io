% Parsing
% Daniel Waterworth
% January 14, 2018

_This post is part of a series on the plastic compiler[^1]. Each
post will cover one pass in the compiler. The first post is
[here](/compiler/lexer.html)._

After lexing comes parsing. Again this is written in recursive descent
style, but this time, we have the option to backtrack (by storing a
stack of parser states). In practice, backtracking makes producing
coherent error messages more difficult, so I do this as little as
possible. Eventually, I'd like to do away with it completely.

If you've never seen expressions parsed with a recursive descent parser,
it's quite interesting. Let's say that we want a parser that can parse
expressions with numbers, plus operators and multiplication operators. So that:

    1+2*3+4

is parsed as:

    1+((2*3)+4)

We'll have three functions: one for the numbers, one for add expressions and
one for multiply expressions.

Plastic's syntax is also quite interesting.

[^1]: [plastic on github](https://github.com/danielwaterworth/plastic-v2)
