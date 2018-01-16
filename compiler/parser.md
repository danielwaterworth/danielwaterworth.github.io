% Parsing
% Daniel Waterworth
% January 14, 2018

_This post is part of a series on the plastic compiler[^1]. Each
post will cover one pass in the compiler. The first post is
[here](/compiler/lexer.html)._

After lexing comes parsing. Again, like the lexer, this is written in
recursive descent style, but now we are going through the list of tokens
instead of the characters of a string. Also, we have the option to
backtrack; this means that we can attempt to parse something one way and
it if doesn't succeed, we can try to parse it another way.

Backtracking works by keeping track of past parser states. When we start
trying to parse something that might fail, we keep a copy of the parser
state before attempting the parse. If it fails we revert to that state
and if it succeeds, we discard that state. In practice, backtracking
makes producing coherent error messages more difficult, so I do this as
little as possible. Eventually, I'd like to do away with backtracking
completely.

In order to make copying the current state cheap, I just keep a
reference to where in the input the next token is. This means that I
don't ever need to modify the list of tokens and so it can be shared
freely.

_Incidentally, if you've done any research, you'll have noticed that
most compilers/interpreters/static analysis tools/etc have their own
hand-written parser. The reason for this is that it's really hard to
get useful error messages out of auto-generated parsers (at least using
currently popular techniques)._

Now, if you've understood the above and the lexer, you should be able
to go away and produce a parser. Or at least, you might be able to, but
don't be surprised if start writing code to parse expressions and you
become unstuck.

Expressions are difficult because of **precedence**.

What is precedence? Precedence is the reason 3\*7+2 = 23 and not 27. In
this example, multiplication has a higher precedence than addition and
it is said to bind more strongly.

If you've never seen expressions parsed with a recursive descent
parser, it's quite interesting. Let's say that we want a parser that
can parse 3\*7+2.

We'll have three functions: one for the numbers, one for add expressions and
one for multiply expressions. The call chain is going to go:

    parse_add_expression -> parse_multiply_expression -> parse_number

We are always calling from low precedence to higher precedence. Here's
some psuedo code:

~~~~ {.python}
    def parse_add_expression(self):
        expr = self.parse_multiply_expression()
        while True:
            if the next token is a plus sign:
                consume it
                expr = Add(expr, self.parse_multiply_expression())
            else:
                return expr
~~~~

Parsing multiply expressions is exactly the same:

~~~~ {.python}
    def parse_multiply_expression(self):
        expr = self.parse_number()
        while True:
            if the next token is an asterisk:
                consume it
                expr = Multiply(expr, self.parse_number())
            else:
                return expr
~~~~

Hopefully you can see why this works. When you parse an expression like this:

    2*3*4+5*6*7

The `parse_add_expression` function is calling out to
`parse_multiply_expression` which consumes as much input as it can
without going below its precedence level. Then `parse_add_expression`
inspects the next token and can decide whether that token should be
dealt with at the current precedence level or whether it is lower.

Plastic's syntax is also quite interesting.

Here's an example of a function definition:

    define
      lookup
      @
        key: *,
        value: *,
        n: u64,
      <~
        hashable(key),
      <-
        table: ptr(hashtable(key, value, n)),
        key: key,
      ->
        maybe(value)
      = {
        ...
      }

 * The `@` section is for type arguments, where you specify their name and
   kind,
 * The `<~` section is for constraints,
 * The `<-` section is for parameters,
 * The type after `->` is the return type,
 * And `= { ... }` is for the body of the function,

I hope you found this interesting.

*If you'd like to continue your journey through the plastic compiler,
the next article on parsing is [here](/compiler/sort_checker.html)*.

[^1]: [plastic on github](https://github.com/danielwaterworth/plastic-v2)
