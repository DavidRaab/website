---
layout:  post
title:   "Perl Sq: bind & cartesian"
slug:    perl-sq-bind-and-cartesian
date:    2023-11-20T20:00:00
lastmod: 2023-11-20T20:00:00
tags:    [perl,sq]
description: How cartesian is implemented in Perl Seq
---

The `cartesian` function returns the [Cartesian Product][1]. The Cartesian
Product is the combination of all possible values.

```perl
my $data =
    Seq::cartesian(
        Seq->wrap(qw/clubs spades hearts diamond/),
        Seq->wrap(qw/7 8 9 10 B D K A/),
    )->to_array;
```

Calling `to_array` will evaluate the expression and generates the following
Perl data-structure.

```perl
[
    ['clubs'  ,'7'],['clubs'  ,'8'],['clubs'  ,'9'],['clubs'  ,'10'],
    ['clubs'  ,'B'],['clubs'  ,'D'],['clubs'  ,'K'],['clubs'  ,'A' ],
    ['spades' ,'7'],['spades' ,'8'],['spades' ,'9'],['spades' ,'10'],
    ['spades' ,'B'],['spades' ,'D'],['spades' ,'K'],['spades' ,'A' ],
    ['hearts' ,'7'],['hearts' ,'8'],['hearts' ,'9'],['hearts' ,'10'],
    ['hearts' ,'B'],['hearts' ,'D'],['hearts' ,'K'],['hearts' ,'A' ],
    ['diamond','7'],['diamond','8'],['diamond','9'],['diamond','10'],
    ['diamond','B'],['diamond','D'],['diamond','K'],['diamond','A' ],
]
```

The implementation of `Seq::cartesian`.

```perl
# cartesian : Seq<'a> -> Seq<'b> -> Seq<'a * 'b>
sub cartesian($seqA, $seqB) {
    bind($seqA, sub($a) {
    bind($seqB, sub($b) {
        wrap('Seq', [$a, $b]);
    })});
}
```

`wrap('Seq', [$a, $b])` is the same as `Seq->wrap([$a, $b])`.

This is how you can implement cartesian in a non-lazy way.

```perl
# cartesian : Array<'a> -> Array<'b> -> Array<'a * 'b>
sub cartesian($arrayA, $arrayB) {
    my @output;
    for my $a ( @$arrayA ) {
        for my $b ( @$arrayB ) {
            push @output, [$a, $b];
        }
    }
    return \@output;
}
```

and call it.

```perl
my $data =
    cartesian(
        [qw/clubs spades hearts diamond/],
        [qw/7 8 9 10 B D K A/],
    );
```

## Related Posts

* [Shape of My Heart][3]
* [Understanding bind][2]

# References

* [Sq on Github](https://github.com/DavidRaab/Sq)

[1]: https://en.wikipedia.org/wiki/Cartesian_product
[2]: {{< ref 2016-04-03-understanding-bind >}}
[3]: {{< ref 2023-11-19-shape-of-my-heart >}}
[4]: /tags/perl-seq
