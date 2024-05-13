---
layout:  post
title:   "Perl Argument Aliasing"
slug:    perl-argument-aliasing
date:    2024-05-13T00:00:00
lastmod: 2024-05-13T00:00:00
tags:    [perl]
description:
---

Different to many other languages Perl has an aliasing feature when you
pass arguments to a function. As an example let's see the following code.

```perl
sub increment {
    $_[0]++;
}
```

As probably known, all values in Perl are put into the `@_` array. But when
you directly modify one of these elements, then the original values are
modified.

So executing the following.

```perl
my $x = 0;

increment($x);
say $x;

increment($x);
say $x;
```

will print `1` and then `2`. It allows you to create functions that directly
can modify the passed arguments without the need to return a new value.

A built-in function that works this way you probably know of is `chomp`. You
call it with `chomp($someString)` and it removes the newline at the end. No need
to pass a reference or returning it.

In fact `increment` is also known. But you probably know it as the `++` operator.

For example `$value++` is the same as `increment($value)`.

I think it is debatable if this is a good feature or not. But like every tool
it is about how you use that feature. A feature itself is neither good or bad.
At least Perl has some built-in functions that works this way, so you also can
create such functions yourself. Seems reasonable to me.

# About OO

I guess some people will complain about this, but when you do, think about the
following.

How is `increment($x)` different to `$x->increment()`? Isn't it the same?

In my previous article about [Why I Like Perl OOs][Perl-OO] I discuss this
in more depth.

In my opinion I don't like it, because it introduces some kind of side-effects.
It changes a value directly without that you maybe expect it. This is true for
the function call and also true for the method call on a object.

# Best Practice

Because of this feature, it is a common best practice in Perl to always copy the
values to a new variable. Or in other word: *Unpack @_* at the beginning of every
function.

```perl
sub increment {
    my $x = @_;
    $x++;
}
```

Changing `increment` to this will not anymore change `$x` when we call `increment($x)`.

# About Performance

This feature of directly modifing its original value in Perl is also known as
aliasing in Perl. But that doesn't mean passing values to a subroutine in Perl
is free of any cost.

When you call a function with `myfunc(@foo)` and `@foo` contains for example
1,000 values, then 1,000 of those aliases have to be created.

On the other hand, when you call `myfunc(\@foo)` then only one reference has
to be created and passed to the subroutine.

The difference is more of either being indirect or explicit. Directly passing
a reference makes it explicit and visible.

```perl
my @array = 1 .. 10_000;

sub noop {}

cmpthese(-1, {
    'alias' => sub {
        noop(@array);
    },
    'reference' => sub {
        noop(\@array);
    },
});
```

Running the above benchmark for example yields the following result.

```
                Rate     alias reference
alias        95467/s        --     -100%
reference 22752120/s    23732%        --
```

So its around 100K function calls vs. 22.5 Millions functions calls per second
difference (with an array of 10,000 values).

# Example

So let's consider you wanna write a function that increments every value by one
in an array. You could do the following implementations.

```perl
sub incr_alias {
    for my $x ( @_ ) {
        $x++;
    }
}

sub incr_ref {
    my ($array) = @_;
    for my $x ( @$array ) {
        $x++;
    }
}
```

Both implements will work. You can use them like this.

```perl
use Data::Printer;

my @foo = 1 .. 10;

p @foo; # 1 .. 10

incr_alias(@foo);
p @foo; # 2 .. 11

incr_ref(\@foo);
p @foo; # 3 .. 12
```

When you btw. wonder why those implementation work. The variable in a `for` loop
in Perl is also an alias. So changing the variable itself changes the value in
an array. I think it is nice sometimes because you don't need an index based
for loop.

But overall I prefer `incr_ref` because of the following reasons:

1. Passing an array by ref makes it explicit.
2. Passing it explicit makes it obvious that maybe it gets mutated.
3. It has a lot better performance (the bigger the array is).

# Immutability

The *functional-way* of doing this is btw. to create a new array with the
incremented values. Perl already has the built-in `map` function that does that.
So you can write.

```perl
my @increments = map { $_ + 1 } @foo;
```

Btw. writing

```perl
my @increments = map { $_++ } @foo;
```

also has the side-effect that not only a copy is created, but the values in `@foo`
are once again incremented, because `$_` is also an alias.


[Perl-OO]: {{< ref 2024-04-15-why-i-like-perls-oo.md >}}
