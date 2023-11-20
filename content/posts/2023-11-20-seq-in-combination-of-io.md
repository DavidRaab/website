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
i needed to pick the filename with the highest number so far.

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
all request from it. This is the operation that can block.

But there are different ways to start a Seq. `wrap` is maybe just the very good
default you should use. As long you never experience peformance problems.

It is in the spirit of YAGNI to not optimize what doesn't need
to be optimized.

How do you optimize? By creating just a single function. The only thing you
must do is to create a single function that always return the next element
when asked. It doesn't matter how you do that.

When you return `undef`, the computation aborts.

This way. `undef` works like an Optional.

Or at least, i threat it this way. So `undef` technically disappers when
you use `Seq`.

So i am basically fixing "The multi-millard dollar-bug". Everything as a Perl
module to load.