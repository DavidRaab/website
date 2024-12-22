---
layout:  post
title:   "Perl Sq: Combination with I/O"
slug:    seq-in-combination-with-io
date:    2023-11-20T01:00:00
lastmod: 2023-11-20T18:40:00
tags:    [perl,sq]
description: Fixing the multi-millard dollar bug
---

In one of my programs I already use my module. This program creates
a new test-file in the 't' folder from a template. From all files in that directory
I want to automatically pick the next highest number. I don't want to search
for the number myself.

So here is how i read all files from disk and only pick files matching the pattern
"%d-$title.t". Then i want the maximum number.

This example shows how you can "connect" to other modules to use them. In this example
its `Path::Tiny`.

```perl
# get the maximum id from test-files so far
my $maximum_id =
    Seq
    ->new(    path('t')->children )
    ->map(    sub($x) { $x->basename })
    ->choose( sub($x) { $x =~ m/\A(\d+) .* \.t\z/xms ? $1 : undef } )
    ->max->or(0);
```

Here is how it works.

1. It begins with `max`.
2. `max` wants to know the highest numerical value. So he ask `choose` for the next value.
3. `choose` also don't know abbout any value, he ask `map` to give him one.
4. `map` also don't know about any value, so he ask `wrap` about one.
5. `wrap` finally knows about a value. He gives him the next one, and asks Do you want this?
6. `map` looks at it. He says, yes i take all. Then he asks `choose` is this the *basename* you wanted?
7. `choose` looks at it. He runs a regex on it to proof the correct format, and extract the number from the *basename*.
    1. When it is the correct format, he gives `max` the requested file. (goto #8)
    2. When it not the correct format, he throws it away, and asks `map` again for the next value. (goto #4)
8. `max` stores the first value as the maximum value. Then he ask again for the next value (goto #2).
    He continues as long `choose` returns an item itself. And `choose` continues as long he gets
    one from `map` and so on ...
9. When `choose` does not return a value anymore. `max` returns the maximum value he stored so far. When `choose` does not give him a single value. He choose `0` as the default to return.

So `max` starts the evaluation. It bubbles back to `wrap`. So execution is basically
written in reverse. In the case of `max` all items must be scanned.

There could be multiple different ways to optimze its performance. But, you
probably always will start with replacing/optimizing `wrap`.

When you give `wrap` all items at once it internally creates an array and serves
all request from it. This is the operation that blocks. But only once, and at the
very beginning.

Theoretically there is only one way to fix it. You just give it a function
that works like a sequence. You can pass it any function that returns another
function.

This is how the internals of *sequence* works. But there are multiple different
helpers (Constructors) to help you create such an construct.

Throughout the system, `undef` is used like an optional. When you return it,
its considered the end of the iterator. Or in some functions returning `undef`
means = skip this item. So `undef` values basically disappear as long you use
`Seq`.

Here a funny part. `Seq` can return `undef`. It's like talking with others.
Inside `Seq` there is no `undef`. But it returns it, just like it keeps getting one
from the outside itself.

Like `max` that can return an `undef` when the sequence not only contains a single
value. Luckily `max` can be given a default value that in that case is returned
instead of an `undef`.

So I am basically fixing "The multi-millard dollar-bug". Everything as a Perl
module to load.

Or at least, an attempt.

# References

* [Sq on Github](https://github.com/DavidRaab/Sq)
