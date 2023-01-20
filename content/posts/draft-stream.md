---
title: "Implementing a lazy sequence in Perl"
date: 2023-01-16T20:19:28+01:00
draft: true
---

In this article I want to explain how to create a Stream (Java), LINQ (C#),
Seq (F#) like library in Perl. I explain how to implement it with functional
programming techniques. The API will resemble the F# API.

First, we need an iterator. In functional programming an *iterator* is just a
function that returns the next element whenever you call the function. We achieve
this by creating a function that returns a function that contains some *state*.
This is named a *closure*.

```perl
sub range($start, $stop) {
    # This is the state
    my $current = $start;
    # a function is returned, but every function has access to its own unique
    # $current. This is called a closure.
    return sub {
        if ( $current <= $stop ) {
            return $current++;
        }
        else {
            return;
        }
    }
}
```

We use this function in the following way.

1. We create the function to return an iterator.
2. The iterator is a function that we execute until the function returns `undef`.

```perl
my $iter = range(1,10);

while ( defined(my $num = $iter->()) ) {
    printf "%d\n", $num;
}
```

We often want to iterator through all items. We can use a `while` loop, but
it is better to abstract the looping into its own function. So we create an
`iter` function first.

```perl
sub iter($iter, $f) {
    while ( defined(my $x = $iter->()) ) {
        $f->($x);
    }
    return;
}
```

With the `iter` function in place, and accepting a lambda, its like creating
our own looping constructs in the language. Now we can replace the while loop
with the following code. This way it is shorter and handles the `defined`
correctly.

```perl
iter($iter, sub($x) {
    say $x;
});
```

the lambda is like the looping body we write in any other loop.

This so far will print all numbers from `1` to `10`. But this kind of iterator
has one problem. Once you reached the end of the iterator it will always return
`undef`. This means passing `$iter` to `iter()` will always go through the
iterator once. Adding an additional `iter(...)` will never do anything.

```perl
# prints: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
iter($iter, sub($x) {
    say $x;
});

# prints nothing because $iter is exhausted.
iter($iter, sub($x) {
    say $x;
});
```

But we can change this behaviour. We can add one layer of indirection. We don't
assume that `range()` returns an iterator anymore. We change the implementation
so that `range()` returns a function that produces a new *iterator* every-time
the function was called. A producer of iterators. Functions that produce
functions!

```perl
sub range($start, $stop) {
    return sub {
        my $current = $start;
        return sub {
            if ( $current <= $stop ) {
                return $current++;
            }
            else {
                undef;
            }
        }
    }
}
```

because we changed `range` we also must change `iter` to reflect the change.

```perl
sub iter($iter, $f) {
    my $i = $iter->();
    while ( defined(my $x = $i->()) ) {
        $f->($x);
    }
    return;
}
```

**From now on** whenever we talk about an *iterator*, I assume this double
indirection as an *iterator*. A user of our API should **never** call `$iter->()`
directly. We should provide all needed functions to work with `$iter`.

This is also the reason why we created the `iter` function and didn't do the
looping ourself. It's a general concept in programming to hide implementation
details. The advantage is that we now can consider `range(1,10)` as
representing the idea to go from `1` to `10` no matter how often we use this
iterator.

This code is also an iteresting idea from a functional perspective. In functional
programming we usually strive for the idea that code should not have state
(mutate data) or have side-effects. Even though the implementation itself
rely on state change this is not seen from the interface anymore.

When we now write `my $iter = range(1,10)` it just looks like `$iter` will
always be the same. As long we don't go into the implementation of `$iter`
it can be considered a *pure* functional interface.

# map

One function common to functional programming is the `map` function. Perl
already implements it. But we want a similar function for our own iterator. This
means we want to create a `map` that takes an `$iter` and a *function* and
the function will be applied to every element returning a new *iterator*.
Because `map` is already used in Perl we name it `imap` at the moment.

```perl
sub imap($iter, $f) {
    return sub {
        my $i = $iter->();
        return sub {
            if ( defined(my $x = $i->()) ) {
                return $f->($x);
            }
            return;
        }
    }
}
```

The nice part, thanks to the change we did, `$iter` now can safely be used
in multiple places. So we can create two more iterators based on `$iter`.

```perl
my $iter   = range(1,10);
my $double = imap($iter, sub($x) { $x * 2  });
my $square = imap($iter, sub($x) { $x * $x });

# prints: 2, 4, 6, 8, 10, 12, 14, 16, 18, 20
iter($double, sub($x) {
    say $x;
});


# prints: 1, 4, 9, 16, 25, 36, 49, 64, 81, 100
iter($square, sub($x) {
    say $x;
});
```

# filter

Next we want to create a `filter` function. In Perl this is built-in and
known as `grep`. `filter` has the purpose to only keep elements in an *iterator*
if an *predicate* (a function that returns true/false) returns true for a given
value.

```perl
sub filter($iter, $f) {
    return sub {
        my $i = $iter->();
        return sub {
            while ( defined(my $x = $i->()) ) {
                if ( $f->($x) ) {
                    return $x;
                }
            }
            return;
        }
    }
}
```

In `imap` we fetched the next element with `$i->()`, then passed it to `$f`
to produce a new value and returned. `imap` always consumed one element
when `$i->()` is called. But `filter` on the other hand must be able to skip
elements.

In a `while` loop we extract one element, pass it to `$f`, and if this returns
a *truish* value we abort by returning this value. Otherwise we must consume
the next value of `$i`. `filter` must consume as much elements until it
find a matching element or exhausted the *internal iterator*.

```perl
my $evenSquared = filter($square, sub($x) { $x % 2 == 0 });

# prints: 4, 16, 36, 64, 100
iter($evenSquared, sub($x) {
    say $x;
});
```

# take

At the moment we only have `iter` to iterate all elements of an iterator. But
one advantage of an iterator is that we don't need to iterate all elements.
Sometimes we want only to iterate a specific amount of elements. So we create
a `take` function that only returns a given amount of elements.

```perl
sub take($iter, $amount) {
    return sub {
        my $i             = $iter->();
        my $returnedSoFar = 0;
        return sub {
            if ( $returnedSoFar < $amount ) {
                $returnedSoFar++;
                if ( defined(my $x = $i->()) ) {
                    return $x;
                }
            }
            return;
        }
    }
}
```

This is our first function that besides `$i` keeps track of an additonal variable
`$returnedSoFar`. We need it to store the information how many elements we already
returned. Once it reaches the `$amount` the internal `sub` will immediately
`return` and never call `$i->()` again even if `$i` is not exhausted.

```perl
my $evenSquared2 = take($evenSquared, 2);

# prints: 4, 16
iter($evenSquared2, sub($x) {
    printf "%d\n", $x;
});
```

# Iterators

Let's see our iterator again in action. We are now able to write the following.

```perl
my $range         = range(1, 1_000_000_000);
my $squared       = imap($range, sub ($x) { $x * $x });
my $evenSquared   = filter($squared, sub ($x) { $x % 2 == 0 });
my $evenSquared10 = take($evenSquared, 10);

# prints: 4, 16, 36, 64, 100, 144, 196, 256, 324, 400
iter($evenSquared10, sub ($x) {
    say $x;
});
```

this will print us the first `10` numbers that are squared and even. The
advantage compared to an array is that through the whole execution only
one element is computed. The whole interface is lazy and only consume/compute as
much as needed. It doesn't have to go through all elements and save all
elements. The above code will almost immediately (for humans) prints all
10 numbers.

This is not the case when we write it with typical Perl code. In fact the
following code made my computer crash with an out-of-memory kernel panic.

```perl
my @range       = 1 .. 1_000_000_000;
my @squared     = map  { $_ * $_ } @range;
my @evenSquared = grep { $_ % 2 == 0 } @squared;

for (my $i=0; $i < 10; $i++) {
    say $evenSquared[$i];
}
```

When we remove the `take` on the *iterative* solution. The code will still work.
This is one of the advantage of *iterators*. It will take some time, but it will
start immediately to print only even squared numbers up to
`1_000_000_000 * 1_000_000_000` only consuming memory of a single element
at a time.

# ifor

We could go on and implement more and more functions that produce iterators
with the same pattern we see in `imap`, `filter` and `take`. When you look
closely you see that all those functions have a loot in common. But functional
programmers don't like design patterns. Instead of a copy-paste solution
with modification we want to create abstractions that can be re-used.

Before continuing we think what those three functions have in common.

1. All functions take an `$iter`.
2. All those functions returns a `$new_iter` maybe consuming all elements
   from the passed `$iter`.
3. In `take` we can see that besides keeping just track of `$i` we need to
   keep track of an additional value `$returnedSoFar`. We can consider this
   as *state*.


# Syntax

TODO: Describe better syntax

# fold

# re-write functions with fold

# unfold

# create range, rangeStep, init, infinity with unfold

# other functions

TODO: toArray, map2, sort, groupBy, indexed, zip, bind, flatten, append