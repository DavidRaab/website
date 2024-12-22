---
layout:  post
title:   "Seq::Manual::FromSub"
slug:    perl-seq-seq-manual-fromsub
date:    2023-11-21T00:00:00
lastmod: 2023-11-21T00:00:00
tags:    [perl,sq]
description: Seq::Manual::FromSub
---

# Seq::Manual::FromSub

A deeper explanation how `Seq->from_sub` works.

# THE PROBLEM

You could write the following in Perl

```perl
Seq->wrap(1 .. 10_000_000)
```

but it would be a bad idea. Perls range operator is non-lazy. When
you call the above code perl will create an array with 10 Mio numbers
and then pass that 10 Mio numbers to `Seq->wrap`.

This is not only time-consuming, it will also use a lot of memory. Maybe
with a bigger number your program or your computer could even crash
with out of memory.

This is the reason why you should use

```perl
Seq->range(1, 10_000_000)
```

instead. It returns a sequence but nothing is computed yet. It only starts
computing values when the sequence is request for values. And even then it will
still only compute as much as needed, or keep those values in memory
that are needed.

`Seq->range` is already provided by this module. But what would be the case
if not?

Then you could create your own range function using `Seq->from_sub`

# range

Here is how to implement your own `range` function.

```perl
sub range($start, $stop) {
    ###-- -- -- -- -- IMPORTANT -- -- -- -- --###
    #          NO CODE SHOULD BE HERE           #
    #    Otherwise it will be CAUSE of BUGS.    #
    # You also should never manipulate function #
    # arguments not even assign a new value to  #
    # it. Do an explicit new assignment in the  #
    #          INITIALIZATION STAGE             #
    ###-- -- -- -- -- -- --- -- -- -- -- -- --###
    return Seq->from_sub(sub {
        # INITIALIZATION STAGE:
        my $current = $start;

        # The iterator returning one element when asked
        return sub {
            # As long $current is equal or smaller
            if ( $current <= $stop ) {
                # return $current and increase by 1
                return $current++;
            }
            # otherwise return undef to indicate end of sequence
            else {
                return undef;
            }
        }
    });
}
```

The pattern you do with `from_sub` is always the same. It is.

```perl
my $sequence = Seq->from_sub(sub {
    # INITIALIZATION CODE HERE

    return sub {
        # RETURN ONE ELEMENT OR UNDEF TO ABORT SEQUENCE
    };
});
```

You can use it directly to create a special sequence as needed, or return
it from a function. So you have a reusable CONSTRUCTOR for creating your
own sequences.

Maybe the most simple sequence would be an infinity sequence always returning
the same value forever.

```perl
my $always = Seq->from_sub(sub {
    return sub {
        return 1;
    }
});
```

You could do

```perl
$always->take(10)->to_array
```

to just get an array `[1,1,1,1,1,1,1,1,1,1]`. Don't forget the `take(10)`
otherwise the sequence will run forever until all your computer memory
is exhausted and your program or computer crashes.

But you could do

```perl
$| = 1;
$always->iter(sub($x) {
    print '.';
});
```

and it will run in an endless-loop, forever printing dots until you abort
the program.

# Reference

* [Sq on Github](https://github.com/DavidRaab/Sq)