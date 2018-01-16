% Sort Checking
% Daniel Waterworth
% January 14, 2018

_This post is part of a series on the plastic compiler[^1]. Each
post will cover one pass in the compiler. The first post is
[here](/compiler/lexer.html)._

So, by this stage we have a representation of what each file contains
and it's time to start working out what it means. To do this, we need
to go through the process of type checking.

However, in order to check that terms have well-defined types, we first
need to know that the types have well-defined kinds and in order to do
that, we need to know that the kinds have well defined sorts.

Yes, you read that correctly, plastic has four layers of
things. Fortunately though, the sort layer is simple. There's only one
sort and users don't currently have the ability to define new kinds, so
sort checking is straight forward.

All we have to do is ensure that when we mention a kind, the kind actually
exists.

*This article is a work in progress*

[^1]: [plastic on github](https://github.com/danielwaterworth/plastic-v2)
