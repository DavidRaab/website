---
title: "Some Perl Optimization Tricks"
date:  2023-01-16T18:29:54+01:00
draft: true
tags:  perl
---

# defining variables outside of loop

This is faster.

    my ($x, $y);
    for ( ... ) {
        $x = ...;
        $y = ...;
    }

than

    for ( ... ) {
        my $x = ...;
        my $y = ...;
    }

because the first version re-use mutable variables and storage. Second version
creates the memory on every loop creation.

# defining subroutine outside of loop.

A subroutine in a for-loop must be created in every loop.

    my $func = sub { ... };
    for ( ... ) {
        $something->method($func, ...);
    }

# using state

When the subroutine-ref has no closures, you can even use state to create function
only on first invocation.

    state $func = sub { ... };
    for ( ... ) {
        $something->method($func, ...);
    }

# comparing integers vs string

Comparing numbers is faster than comparing strings. Declaring variables that
hold numbers for decision making makes sense. Much like enums.

    my $none = 0;
    my $some = 1;

`$x == $some` instead of `$some = $x`

# regex: not using captures when not needed

When you don't need the capture,

    my $r = qr/\A (\d | \d\d)/; # this populates $1

then use `(?: )`.

    my $r = qr/\A (?: \d | \d\d)/; # this not

# regex: possesive quantifiers

A state machine must backtrack, this can be slow. When you know bcktracking is
useless than add an additional `+`.

    \d++

# Avoiding function calls (inlining)

A hard trush, but function call has a big performance hit. Especially in Perl.
There is also no JIT that can inline. So when you expect a loop that runs
a million time you must inline the function call.

# hash vs binary search

Just don't use Binary Search. Built a Hash. Building a Hash is faster than
Sorting. Adding new elements to a hash is faster as inserting
into an array. And a Hash-Lookup is faster than Binary Search.

# general using hashes vs. array



* Plain Array/Hash vs classes
* string-eval for code optimization
* picking good algorithm.
* avoiding type-checking (at runtime/release app)
* foreach loop vs. old-school for-loop with index based access
* map/grep vs building array with foreach loop
* Using Profiler (Hint: Advent of Code Array2D)
* Caching? (Memoization Fibonacci)
* Order of if-statements
* using hash instead of if/elsif/switch
* passing refs instead of varargs
* save result of hash into variable (my $v = $hash->{name})

