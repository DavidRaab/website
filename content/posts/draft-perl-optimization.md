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

When the subroutine-ref has no closures, you sould use state to create function
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

`$x == $some` instead of `$x eq "some"`

# regex: not using captures when not needed

When you don't need the capture,

    my $r = qr/\A (\d | \d\d)/x; # this populates $1

then use `(?: )`.

    my $r = qr/\A (?: \d | \d\d)/x; # this not

# regex: possesive quantifiers

A state machine must backtrack, this can be slow. When you know bcktracking is
useless than add an additional `+`.

    \d++

# Avoiding function calls (inlining)

A hard truth, but function call has a big performance hit. Especially in Perl.
There is also no JIT that can inline. So when you expect a loop that runs
a million times you can gain a lot of performance when you inline the whole
code.

```perl
my $nums = Sq->rand->int(1,1_000_000)->to_array(10_000);

sub add1($x) { $x + 1 }

Sq->bench->compare(-1, {
    inline => sub {
        my @new = map { $_ + 1 } @$nums;
    },
    no_inline => sub {
        my @new = map { add1($_) } @$nums;
    },
});
```

result

```
            Rate no_inline    inline
no_inline  922/s        --      -66%
inline    2715/s      194%        --
```

# hash vs binary search

Don't use Binary Search. Built a Hash.

* Creating a Hash is faster than Sorting.
* Adding new elements to a hash is faster as insertion sort into a sorted array.
* A Hash-Lookup is faster than Binary Search.
* It's also less code and easier to understand.

# general using hashes vs. array

Most time I had performance problems in Perl was because i was using (scanning)
an Array instead of using a Hash. A typical example, you want to check if
a string is one of some valid cases. So you do a nested for loop. Just
scanning 3 elements are already 3x slower than using a Hash!

Slow variant
```perl
my $count  = 0;
my @valids = qw(yes no maybe);
for ( 1 .. 1_000 ) {
    for my $word ( @words ) {
        for my $valid ( @valids ) {
            if ( $word eq $valid ) {
                $count++;
                last;
            }
        }
    }
}
```

Building a Hash and just doing a hash-lookup is three times faster as the
nested for-loop.

```perl
my $count = 0;
my %valid = map { $_ => 1 } qw(yes no maybe);
for ( 1 .. 1_000 ) {
    for my $word ( @words ) {
        $count++ if $valid{$word};
    }
}
```

[oneOf Benchmark](https://github.com/DavidRaab/Sq/blob/master/benchmarks/one_of.pl)

# Regex (?> )

Regular expressions can sometimes be improved a lot with useful possesive quantifiers.
Is I already mentioned like `\d++`. Using `(?> )` is the more general concept.

For example even using the following regex is faster than the above array scanning.

```perl
my $count = 0;
for ( 1 .. 1_000 ) {
    for my $word ( @words ) {
        $count++ if $word =~ m/\A(?>yes|no|maybe)\z/;
    }
}
```

But only when you add `(?> )`. So what does it exactly? It avoids back-tracking.
It is better explained with this regex. `\d+a`

A regex by default is greedy and tries to match as much as possible. The regex
`\d+a` run against the string "1234b" would first match all the digits `1234`
with `\d+` and then checks if the next character is an `a`. This fails because
it is an `a`.

A regex than back-tracks, it removes one character from `1234` so `\d+` than
matches `123`, then it checks if `4` is `a` and fails again. Obviously this
is silly. A regex just tries every possible combination even when it doesn't
make much sense. Adding `\d++` or writing `(?>\d+)` removes the back-tracking
and often can speed up regexes a lot.

But you also must ensure that your regexes are still correct. Just blindly
adding those possessive quantifiers can result that your regex don't match
smething even if you expect it.

# Plain Array/Hash vs classes



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

