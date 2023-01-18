---
title: ""
date: 2023-01-16T18:29:54+01:00
draft: true
---

Now to the final task. Let's consider we want to parse an array that is provided
by a string. Sure, we could use `eval` or other helpers. But we want to do it
ourself. We consider that the array only can contain numbers. The string
could look like this.

```
[1,2,3,4]
```

For parsing this array we could split it into four different regexes. First
an array must start with `[`. The end of an array is `]`. The delimeter is
`,` and finally we have the digit. So wouldn't it be cool if we can parse
a string by checking one of those regexes to get information what comes next?

We would start with something like this.

```perl
parseArray("[1,2,3,4]");

sub parseArray($str) {
    while ( $str !~ m/\G\z/gxms ) {
        # start of array
        if ( $str =~ m/\G \[ /gxms ) {
            say "Start";
        }
        # end of array
        elsif ( $str =~ m/\G \] /gxms ) {
            say "End";
        }
        # delimeter
        elsif ( $str =~ m/\G ,/gxms ) {
            say "Comma";
        }
        # digit
        elsif ( $str =~ m/\G (\d+) /gxms ) {
            say "digit";
        }
        else {
            die "Error!\n";
        }
    }
}
```

But when we run this code, the code will run in an endless loop, printing `Start`
forever.

So here is what happens. Our `while` loop starts with the condition that we want
to continously match on `$str` until `\G` is the same as `\z`. Or in other words,
reached the end of the string. This usually fails as long we did not reach the end.

Then in our loop we try different regexes to see what comes next. The first match
in our loop is sucessfull as our string starts with `[`. The code prints `Start`
and we are at `while` body again. Now the regex will check if `1,2,3,4]` is already
the end of the string. This fails as we want, but here is the cache.

When it fails the position of the regex engine starts again at `0`. Then again
we check if `[1,2,3,4]` starts with `[` this is true, and this will continue
forever.

The only thing we need is to say that the regex engine should not delete the
position. This can be achieved by adding the `/c` option to a regex. And
the important bit is, to **add it to every regex**!

```perl
parseArray("[1,2,3,4]");

sub parseArray($str) {
    while ( $str !~ m/\G\z/gxmsc ) {
        # start of array
        if ( $str =~ m/\G \[ /gxmsc ) {
            say "Start";
        }
        # end of array
        elsif ( $str =~ m/\G \] /gxmsc ) {
            say "End";
        }
        # delimeter
        elsif ( $str =~ m/\G ,/gxmsc ) {
            say "Comma";
        }
        # digit
        elsif ( $str =~ m/\G (\d+) /gxmsc ) {
            say "digit";
        }
        else {
            die "Error!\n";
        }
    }
}
```

now running the program will print

```
Start
digit
Comma
digit
Comma
digit
Comma
digit
End
```

Now, we want to create a Perl data-structure from it.

A first attempt is

```perl
sub parseArray($str) {
    my @array;
    while ( $str !~ m/\G\z/gxmsc ) {
        # start of array
        if ( $str =~ m/\G \[ /gxmsc ) {
            # say "Start";
        }
        # end of array
        elsif ( $str =~ m/\G \] /gxmsc ) {
            # say "End";
        }
        # delimeter
        elsif ( $str =~ m/\G ,/gxmsc ) {
            # say "Comma";
        }
        # digit
        elsif ( $str =~ m/\G (\d+) /gxmsc ) {
            push @array, $1;
        }
        else {
            die "Error!\n";
        }
    }

    return wantarray ? @array : \@array;
}
```

This succesfully parses the string and calling.

```perl
my $data = parseArray("[1,2,3,4]");
```

will give us an array-reference containing the numbers. But it still parses
invalid inputs. It for example also parses `[,,,,4,3,,,]` and turns
it into an `[4,3]` array in Perl. Nonetheless this is how you can
use the technique to parse more complex strings. Here is the example corrected
and also being able to parse arrays containg additional arrays.

