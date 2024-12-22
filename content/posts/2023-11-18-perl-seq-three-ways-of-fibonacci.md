---
layout: post
title: "Perl Sq: Three ways of doing Fibonacci"
slug: lazy-sequence-in-perl-three-ways-of-doing-fibonacci
date:    2023-11-18T01:00:00
lastmod: 2023-11-18T17:00:00
tags: [perl,linux,sq]
description: First preview of my Perl lazy Sequence implementation
---

Hi there, I am developing a new Perl module to bring a lazy Sequence to Perl.

It should provide the functionaly you see in C# LINQ or Java 8 Stream. The
origin of those interfaces comes from functional programming. Thus i decided
to primarily pick the F# API and port it to Perl.

I already implemented a useful set of functions. Here is an example of the
module and what you can do with it. Source code of my [Sq Module](https://github.com/DavidRaab/Sq) can be found at GitHub so far.

When i have written more functions and documentation i will release it to CPAN.

# First solution

```perl
my $fib =
    Seq->concat(
        Seq->new(1,1),
        Seq->unfold([1,1], sub($state) {
            my $next = $state->[0] + $state->[1];
            return $next, [$state->[1],$next];
        })
    );

is(
    $fib->take(20)->to_array,
    [1,1,2,3,5,8,13,21,34,55,89,144,233,377,610,987,1597,2584,4181,6765],
    'First 20 Fibonacci numbers');
```

# Second Solution

```perl
# Same Fibonacci as above but unfold does not create a new arrayref on every
# iteration. It changes the $state instead. This way less garbage is created
# and could be potential a little bit faster. But it envolves writing more code.
{
    my $fib =
        Seq->concat(
            Seq->new(1,1),
            Seq->unfold([1,1], sub($state) {
                my $next = $state->[0] + $state->[1];
                $state->[0] = $state->[1];
                $state->[1] = $next;
                return $next, $state;
            })
        );

    is(
        $fib->take(20)->to_array,
        [1,1,2,3,5,8,13,21,34,55,89,144,233,377,610,987,1597,2584,4181,6765],
        'First 20 Fibonacci numbers');
}
```

# Third Solution

```perl
# You also can use a hash as a state.
{
    my $fib =
        Seq->concat(
            Seq->new(1,1),
            Seq->unfold({x => 1, y => 1}, sub($state) {
                my $next = $state->{x} + $state->{y};
                $state->{x} = $state->{y};
                $state->{y} = $next;
                return $next, $state;
            })
        );

    is(
        $fib->take(20)->to_array,
        [1,1,2,3,5,8,13,21,34,55,89,144,233,377,610,987,1597,2584,4181,6765],
        'First 20 Fibonacci numbers');
}
```

# Bonus: My concat implementation

```perl
# Concatenates a list of Seq into a single Seq
sub concat($class, @iters) {
    my $count = @iters;

    # with no values to concat, return an empty iterator
    return empty('Seq') if $count == 0;
    # one element can be returned as-is
    return $iters[0]    if $count == 1;
    # at least two items
    return reduce { append($a, $b) } @iters;
}
```

# References

* [Sq on Github](https://github.com/DavidRaab/Sq)
