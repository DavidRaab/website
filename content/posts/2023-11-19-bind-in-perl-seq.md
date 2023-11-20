---
layout:  post
title:   "Perl Seq: bind & flatten"
slug:    perl-seq-module-bind-flatten
date:    2023-11-19T02:00:00
lastmod: 2023-11-19T02:00:00
tags:    [perl,perl-seq]
description: Code example with bind and flatten
---

I implemented the `bind` function in my Perl [Seq Module][1]. `bind` is
a very useful function. For example here is `flatten` that I implemented with
it.

# flatten

Flatten takes a *sequence* of *sequences* and returns a single sequence.

```perl
my $flattened =
    Seq->wrap(
        Seq->wrap(1,1),
        Seq->wrap(2,3,5,8,13),
    )
    ->flatten;
```

This is how a non-lazy Perl implemenation would look like:

```perl
sub flatten($aoa) {
    my @flattened;
    for my $outer ( @$aoa ) {
        for my $inner ( @$outer ) {
            push @flattened, $inner;
        }
    }
    return \@flattened;
};
```

Using it looks very similar.

```perl
my $flattened =
    flatten([
        [1,1],
        [2,3,5,8,13],
    ]);
```

This is how `flatten` is implemented in `Seq`.

```perl
# flatten : Seq<Seq<'a>> -> Seq<'a>
sub flatten($iter) {
    return bind($iter, $id);
}
```

`$id` is the id-function. It's implementation:

```perl
my $id = sub($x) { return $x }
```

I also could write.

```perl
sub flatten($iter) {
    bind($iter, sub($x) { $x });
}
```

`bind` is basically [binding a value from left to right][2]. Its like
assignment. It binds one value from the `$iter` *sequence* to the value `$x`.

The function we pass to, is executed for every value. But only when needed.

We could say that `bind` is like a lazy for-loop.

```perl
for my $x ( @$iter ) {
    ...
}
```

consider `for` as a function. It binds one value of `@$iter` to `$x` and executes
the function we pass with `{ ... }`. But a *for-loop* does it in a non-lazy way.
It computes all values at once. Even if you only want to acces the first value of it.

`Seq->flatten` instead creates a new *sequence* without doing anything as long
you never ask it to evaluate.

Inside the *lambda* we passed to `flatten` we must return another *sequence*.
But `bind` will flatten the results to a single *sequence*.

This is why `$id` is passed. It returns the inner *sequence* as-is.

`bind` itself could be implemented with `map` and `flatten`.

```perl
sub bind($seq, $f) {
    return $seq->map($f)->flatten;
}
```

[1]: https://github.com/DavidRaab/Seq
[2]: {{< ref 2023-11-18-asssigning-variables >}}