% Lexing
% Daniel Waterworth
% January 13, 2018

_This is the first post in my series on the plastic compiler[^1]. Each post
will cover one pass in the compiler._

There's already a lot on the web about lexing, so I won't go over it
again, but just share what makes the plastic compiler different. Also,
I'm not entirely happy with the way the lexer is written. Eventually,
it'll get rewritten, but until then it is what it is.

With that out of the way, the lexer is written in recursive descent
style. There's a class, called Lexer, that holds the tokens that have
been produced so far and the input that is left to lex.

Lexing in the compiler doesn't work using regexes like other lexers;
there's logic for lexing each kind of token.

Another problem with the current lexer is the way symbols are handled.
I was trying to avoid predefining the set of symbols that exist in
the language, so any contiguous group of symbol characters is treated
as a single symbol. However, there are some symbols where this isn't
convenient and so there's this special set of symbol characters that get
emitted as their own token.

One thing I think the lexer does right is that keywords are just special
identifiers. So, first an identifier is lexed, then if it's in the set of
keywords, a keyword token is produced instead of an identifier token.

The function that ties it all together is this:

~~~~ {.python}
class Lexer:
    ...

    def lex(self):
        # ws stands for whitespace
        self.skip_ws()
        while not self.eof():
            pos = self.pos
            if self.next in special_symbols:
                self.tokens.append(Token(special_symbols[self.next], pos = pos))
                self.advance(1)
            elif self.next == '"':
                self.tokens.append(self.lex_string())
            elif self.next == '\'':
                self.tokens.append(self.lex_char())
            elif self.next in symbol_chars:
                self.tokens.append(self.lex_symbol())
            elif self.next.isdigit():
                self.tokens.append(self.lex_number())
            elif self.next_is_valid_identifier_char():
                self.tokens.append(self.lex_identifier())
            else:
                print(self.next)
                raise NotImplementedError()
            self.skip_ws()
        return self.tokens
~~~~

So, it's pretty crude, but it works for now.

[^1]: [plastic on github](https://github.com/danielwaterworth/plastic-v2)
