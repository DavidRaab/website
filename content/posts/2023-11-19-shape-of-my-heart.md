---
layout: post
title: "Shape of my Heart"
slug: shape-of-my-heart
date:    2023-11-19T03:26:00
lastmod: 2023-11-19T03:26:00
tags: [perl,linux,sequence,perl-seq]
description: Shape of my Heart
---

Here are two programs that produces the same result.

This is something I would have written without my Seq module.

```perl
[
    (map { [clubs   => $_ ] } qw/7 8 9 10 B D K A/),
    (map { [spades  => $_ ] } qw/7 8 9 10 B D K A/),
    (map { [hearts  => $_ ] } qw/7 8 9 10 B D K A/),
    (map { [diamond => $_ ] } qw/7 8 9 10 B D K A/),
]
```

With my module, i now can write the same in this way.

```perl
Seq::cartesian(
    Seq->wrap(qw/clubs spades hearts diamond/),
    Seq->wrap(qw/7 8 9 10 B D K A/),
)
```

This is the [Shape of my Heart](https://www.youtube.com/watch?v=QK-Z1K67uaA)

Which version can you understand better? Which version do you like more? What
do you prefer to write? And when, why?

My module is Lazy and will only compute as much data as needed.

Sometimes it can be better at performance. In some other cases not.

But instead you always have lower memory cost and code runs immiadtely, it does
not need to wait for all computation to finish.

A Seq is a more than just an *iterator*. We could say an *iterator* is the
mutable variant of an immutable *Sequence*.

A *sequence* always compute the same, no matter how often you start it. An
*iterator* must explictily asked and resetted. If that is even supported.

Anyway, the code expect you to work with immutability in mind.

That is what it makes it sane to work with. If that is violated, ohhhh, it will
hit you hard.

The module will allow LISP-style coding and OO-Chaining. As an example. You
also could have written.

```perl
Seq->wrap(qw/clubs spades hearts diamond/)->cartesian(
    Seq->wrap(qw/7 8 9 10 B D K A/),
)
```

I prefer the first version *in this case*, in some other cases i prefer
oo-chaining. Depends on how i think it is better to read.

No inheritance is allowed, code actively written that you cannot subclass the
code, or it will break.

I consider performance a more important part, as the error messages. Through
F# i have learned to love static typing, but all those type validation modules
in Perl are just to damn slow.

Instead I would try a different approach. You would swap modules instead. Both
have the Same API, and work the same. You either load "Seq" or "Seq::Learn".
But when `Seq::Learn` is loaded. You get special creafted error-messages that tell
you in detail what you made wrong, with examples.

When you fixed the code, you swap back to just `Seq` and get all the speed
advantages you want, for fast performance.

I also think about writing my own **Discriminated Union** and **Record**
implementation in Perl. **Discriminated Union** are such an awesome feature from F#,
it makes it a pain to work in any other language that doesn't have this as
a built-in language feature.

