---
layout: post
title: "Shape of my Heart"
slug: shape-of-my-heart
date: 2023-11-19
tags: [perl,sq]
description: Shape of my Heart
---

Here are two programs that produces the same result.

This is something I would have written without my `Sq` module.

```perl
[
    (map { [clubs   => $_ ] } qw/7 8 9 10 J Q K A/),
    (map { [spades  => $_ ] } qw/7 8 9 10 J Q K A/),
    (map { [hearts  => $_ ] } qw/7 8 9 10 J Q K A/),
    (map { [diamond => $_ ] } qw/7 8 9 10 J Q K A/),
]
```

With my module, i now can write the same in this way.

```perl
use Sq;

my $cards =
    Seq::cartesian(
        Seq->new(qw/clubs spades hearts diamond/),
        Seq->new(qw/7 8 9 10 J Q K A/),
    )
```

This is the [Shape of my Heart](https://www.youtube.com/watch?v=QK-Z1K67uaA)

My module is Lazy and will only compute as much data as needed.

Sometimes it can be better at performance. In some other cases not.

But instead you always have lower memory cost and code runs immiadtely, it does
not need to wait for all computation to finish.

A Sequence is somewhat an *immutable* version of an *iterator*. It always
represents the same calculation. You can run it as often as you wish.

Code is written to expect immutability, but does not enforce it.

You can violate the rule, and it can cause serious problems if you do. But maybe
some stuff is wanted.

The module will allow LISP-style coding and OO-Chaining. As an example. You
also could have written.

```perl
my $cards =
    Seq->new(qw/clubs spades hearts diamond/)->cartesian(
        Seq->new(qw/7 8 9 10 B D K A/),
    );
```

I prefer the first version *in this case*, in some other cases i prefer
oo-chaining.

No inheritance is allowed, code actively written that you cannot subclass the
code, or it will break.

I consider performance a more important part as the error messages. Through
F# i have learned to love static typing, but all those type validation modules
in Perl are just to damn slow.

Instead I would try a different approach. You would swap modules instead. Both
have the Same API, and work the same. You either load `Seq` or `Seq::Learn`.
But when `Seq::Learn` is loaded. You get special creafted error-messages that tell
you in detail what you made wrong, with examples.

When you fixed the code, you swap back to just `Seq` and get all the speed
advantages you want, for fast performance.

I also think about writing my own **Discriminated Unions** and **Record**
implementation in Perl. **Discriminated Unions** are such an awesome feature from F#,
it makes it a pain to work in any other language that doesn't have this as
a built-in language feature.

# Update

Development switched from a module that was just about a sequence `Seq` to
a new name `Sq`. This new module contains a lot of other stuff that work together
like wrappers arround `Array`, `Hash`, data-structures for `Option`, `Result`
and at the moment a `Sq::Parser` and it's own typing system `Sq::Type`.

[Sq on Github](https://github.com/DavidRaab/Sq)
