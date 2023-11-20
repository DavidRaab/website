---
layout: post
title: "Seq in Combination with I/O"
slug: seq-in-combination-with-io
date: 2023-11-20
tags: [perl,perl-seq]
description: Fixing the multi-millard dollar bug
---

In one of my scripts I am already using my module. I created a script
to faster create new test-files. From all files in that directory
I needed to pick the filename with the highest number so far.

So here is how i read all files from disk and only pick files matching pattern
"%d-$title.t". Then i want the maximum.

It shows how you can "connect" to other modules to use them. In this example
its `Path::Tiny`.

```perl
# get the maximum id from test-files so far
my $maximum_id =
    Seq
    ->wrap(   path('t')->children )
    ->map(    sub($x) { $x->basename })
    ->choose( sub($x) { $x =~ m/\A(\d+) .* \.t\z/xms ? $1 : undef } )
    ->max;
```

Here is how it works.

1. It begins with `max`.
2. `max` wants to know the highest numerical value. So he ask `choose` for the first value.
3. `choose` also don't know abbout any, he ask `map` to give him one.
4. `map` also don't know about any value, so he ask `wrap` about one.
5. `wrap` finally knows about a value. He gives him the first one, and asks Do you want this?
6. `map` looks at it. He says, yes i take all. Then he asks `choose` is this the *basename* you wanted?
7. `choose` looks at it. He runs a regex on it to proof the correct format.
 1. When it is the correct format, he gives `max` the requested file. (goto #8)
 2. When it not the correct format, he throws it away, and asks `map` again for the next value. (goto #4)
8. `max` stores the first value as the maximum value. Then he ask again for the next value (goto 2).
    He continues as long `choose` returns an item itself. And `choose` continues as long he gets
    one from `map` and so on ...

So `max` starts the evaluation. It bubbles back to `wrap`. So execution is basically
written in reverse. In the case of `max` all items must be scanned.

There could be multiple different ways to optimze it by performance. But, you
probably always will start with `wrap`.

When you give `wrap` all items at once. It internally creates an array and serves
all request from it. This is the operation that block. But only once, and at the
very beginning.

Theoretically there is only way to fix it. You just give it a function
that works like a sequence. You can pass it any function that returns another
function.

That is the internal. But there are multiple different helpers (Constructors)
to help you create such an construct.

Throughout the system, `undef` is used like an optional. When you return it,
its considered the end of the iterator. So `undef` values basically disappear
as long you use `Seq`.

Here a funny part. `Seq` can return `undef`. It's like talking with others.
Inside `Seq` there is no `undef`. But it returns it, just like it keeps getting one
from the outside itself.

So i am basically fixing "The multi-millard dollar-bug". Everything as a Perl
module to load.

Or at least, an attempt.