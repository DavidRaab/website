---
layout: post
title: "Perl Regex Parsing with the g option"
slug: perl-regex-parsing-with-g-option
date: 2023-01-07
lastmod: 2023-01-09
tags: [perl,regex,parsing]
description: "Shows how to do parsing with Perl and the g modifier"
keywords: perl, parsing, parser, regex
---

Perl regexes have the `/g` modifier that is worth knowing about, as you
can do a lot of advanced things with this modifier. But here are the basics
first.

# Context

First we need to understand that Perl has two different contexts and in both
contexts the `/g` modifier works differently.

## List context

Let's see list context first. We can enforce list context by assigning to an array.

Without the `/g` option a regex just finds its first match and just returns it.

```perl
my $str  = "1234 and 4567 and 8901";
my @nums = $str =~ m/(\d+)/;  # 1234
```

Here `@nums` will only contain a single value `1234`. But with  `/g` we can extract
all matches at once.

```perl
my $str  = "1234 and 4567 and 8901";
my @nums = $str =~ m/(\d+)/g;  # (1234, 4567, 8901)
```

With `/g` we get three entries containing the digital numbers. It is important
to know that Perl will try to match everything and extract everything
at once. So when we have a very large string we will get back an array with
all matches at once. We will see later why this can matter.

We get list context in Perl by either assigning to an array, assigning to
a list, by using a regex match inside a `for` loop, or some other functions like
`map`, `grep` and so on.

But when we assign to a list we only extract as many values as we have defined.
```perl
my ($first, $second) = $str =~ m/(\d+)/g; # 1234, 4567
```

In a for loop you want to use it like this.

```perl
for my $num ( $str =~ m/(\d+)/g ) {
    say $num;
}
```

Maybe you are used to using `$1, $2, $3, ...` for accessing the capture group.
But with a `for` loop like above, you will have an unexpected output.

```perl
for my $num ( $str =~ m/(\d+)/g ) {
    say $1; # always prints 8901
}
```

In this case you will see `8901` three times. This is because the regex, like
said above, runs completly to the end before returning
control to the `for` loop. In this case you will only have access to the
last match. And this was `8901`.

Consider that using `if` or `while` is scalar context and it works a little bit
different in those.

## Scalar Context

Differing from a list context that returns all its matches at once, a regex
that is used in scalar context just returns a value if the match
was succesfull or not. So using a regex in a `if` statements does not return
the match itself. You now must use `$1, $2, $3, ...` to extract this information.

When you use a regex in scalar context on a string with the `/g` modifier
you might think nothing will happen. For example both of the
code examples will return the exact same thing.

```perl
say $1 if $str =~ m/(\d+)/ # 1234
```

```perl
say $1 if $str =~ m/(\d+)/g # 1234
```

Both will print `1234` to the console. But with the `/g` modifier there is a
difference. Perl internally saves the position where the regex engine stopped.
So when you do another new match against `$str` with a regex that has `/g`
enabled, it startes again where it stopped. For example:

```perl
say $1 if $str =~ m/(\d+)/; # prints 1234
say $1 if $str =~ m/(\d+)/; # prints 1234
say $1 if $str =~ m/(\d+)/; # prints 1234
```

But with `/g` we get

```perl
say $1 if $str =~ m/(\d+)/g; # prints 1234
say $1 if $str =~ m/(\d+)/g; # prints 4567
say $1 if $str =~ m/(\d+)/g; # prints 8901
```

In Perl you can get (or set) the position where the regex stopped with the
`pos` function. You can also get where it will start when you use a new regex against the
same string.

```perl
say pos($str);               # undef
say $1 if $str =~ m/(\d+)/g; # 1234
say pos($str);               # 4
say $1 if $str =~ m/(\d+)/g; # 4567
say pos($str);               # 13
```

The value returned by `pos` is between the characters
of a string, pointing to the next character to be considered. It start with `0` before the first character of a string. So
position `4` means it is between `4` and the space character ` `.

But it is also possible to set the starting position before you use a regex
on it. You can use `pos` as a left-value and assign a value to it.

```perl
my $hello = "Hello, World!";
pos($hello) = 7;
say $1 if $hello =~ m/(\w+)/g; # prints "World"
```

Without the `/g` in this example, you would get `Hello` as the regex engine
would then start at the beginning and discards the previous position.

So overall using `/g` on the same string in scalar context is like an *iterator*.
It will always return the next match as long there is any. As soon the regex
cannot find a match an `undef` will be returned. Then it will start over again.

```perl
say $1 if $str =~ m/(\d+)/g; # prints 1234
say $1 if $str =~ m/(\d+)/g; # prints 4567
say $1 if $str =~ m/(\d+)/g; # prints 8901
say $1 if $str =~ m/(\d+)/g; # not true
say $1 if $str =~ m/(\d+)/g; # prints 1234
say $1 if $str =~ m/(\d+)/g; # prints 4567
```

With this behavior you can use it in a `while` loop.

```perl
while ( $str =~ m/(\d+)/g ) {
    say $1;
}
```

Compared to the `for` loop it is now safe to use `$1`. In fact, you
must use `$1`. This becomes especially handy if you extract more than just one
value per match. The loop will print all three numbers once and then finish.

# different regexes

Here is one interesting aspect. When you use `/g` on a string it saves
the position where it stopped. But this information is relative to the string
and is independent of the regex you use. This means you can use different
regexes with `/g` on the same string.

```perl
my $str = "1234 and 4567 and 8901";
say $1 if $str =~ m/(\d+)/g; # 1234
say $1 if $str =~ m/(\w+)/g; # and
say $1 if $str =~ m/(\d+)/g; # 4567
say $1 if $str =~ m/(\w+)/g; # and
say $1 if $str =~ m/(\d+)/g; # 8901
```

This can help in parsing as you can parse with one regex, advance the
regex engine some characters forward and use another regex. But it has one
flaw; it resets the position to `0` once it fails (but we can fix
that, as you will see in a moment).

# \G anchor

But first let's talk about the `\G` anchor. So far we have used `/g` succesfully
on the string. But the way regexes work, they usually skip characters to find
a starting match. For example, suppose you want to check if a string only contains digits.
THen, you must add anchors like `^`, `\A`, `$`, `\Z` and/or `\z`. Otherwise
you allow more matches than you want.

```perl
my $only_digits = "hello 1234";
if ( $only_digits =~ m/(\d+)/ ) {
    # ...
}
```

The above code will be true as the regex just scans for the first digits
in the string. So it will extract `1234`. But often we want to check if the
whole string matches a given regex. That's why we write.

```perl
my $only_digits = "hello 1234";
if ( $only_digits =~ m/\A (\d+) \z/xms ) {
    # ...
}
```

In this example the `if` statement would not be executed as our string starts with
`hello` and not with digits. In the same way, we want to have the same kind of
restriction when we use `/g`. We need a way to say: *Now check something, and
it must start where the last regex stopped*. This is what the `\G` anchor
does in a regex.

Consider the case in which we want to parse an alphanumeric followed by a number. When we don't
use any anchor this will be sucessful, even if it shouldn't.

```perl
my $str = "hello world 1234";
say $1 if $str =~ m/(\w+)/gxms; # hello
say $1 if $str =~ m/(\d+)/gxms; # 1234
```

The first regex parses `hello`, but then the next line will search for the next
digits. But we want it to be restricted that only digits must follow. We get this
restriction by using `\G` as an anchor.

```perl
my $str = "hello world 1234";
say $1 if $str =~ m/\G (\w+)/gxms; # hello
say $1 if $str =~ m/\G (\d+)/gxms; # 1234
```

Now, only `hello` will be printed because after `hello`, the remaining string
not yet consumed by the regex engine will be ` world 1234`. And that doesn't start
with digits. It starts with an space character.

So it seems that this way, we can split our parsing into multiple chunks
(or tokens). But there is another problem. As we learned before, as soon
a regex doesn't match on a string it will reset its position to `0`.

This makes sense when we use a single regex with a `while` loop as otherwise
we would have an infinite loop. But it would be cool if we could turn this off.

# /c option

For now, let's think of another use case. Consider we want to parse lines
in the following format.

```
maximum = 1234
verbose = true
```

This seems like a configuration file that supports just key and value assignment.
Sure, it would be very easy to just do it with a single regex. But it also
can make sense to split it up into multiple statements and parse it step after
step. The separation can be easier to read and understand, but also easier to
write. Especially if what we try to parse is a lot more complex.

The idea goes as follows. We first try to just parse the key, then the equal
sign, and finally expect a bool or an integer. We could start with something like
this.

```perl
my $config = "verbose = true";

if ( $config =~ m/\G (\w+)/gxms ) {
    my $key = $1;
    if ( $config =~ m/\G \s* = \s* /gxms ) {
        if ( $config =~ m/\G (true|false) \s*/gxms ) {
            my $value = $1 eq 'true' ? 1 : 0;
            printf "Key=[%s] Value=[%s]\n", $key, $value;
        }
    }
}
```

This parses the boolean line sucessfully. But when we try to add the integer
part, it wouldn't work.

```perl {hl_lines=[1,"10-13"]}
my $config = "maximum = 1234";

if ( $config =~ m/\G (\w+)/gxms ) {
    my $key = $1;
    if ( $config =~ m/\G \s* = \s* /gxms ) {
        if ( $config =~ m/\G (true|false) \s*/gxms ) {
            my $value = $1 eq 'true' ? 1 : 0;
            printf "Key=[%s] Value=[%s]\n", $key, $value;
        }
        elsif ( $config =~ m/\G (\d+) \s*/gxms ) {
            my $value = $1;
            printf "Key=[%s] Value=[%s]\n", $key, $value;
        }
    }
}
```

In the last case we first parse `maximum`, then it would succeed to parse
the equal sign `=`. But then it will first check if what comes next is either
`true` or `false`, but it will fail. The problem is that when it fails the regex
engine will reset its position back to the beginning.

So when the `elsif` checks for `(\d+)`, it will not succeed. So we just need a
way for checking for `(true|false)` without that the regex engine resets the position
if it fails. We could save the position with `pos` before we match and set it again
after a fail. But it is just easier to add the option `/c` to the regex, as this will
change the regex engine to not reset the position anymore. And a best practice is
to just add it to all of them.

```perl {hl_lines=[3,5,6,10]}
my $config = "maximum = 1234";

if ( $config =~ m/\G (\w+)/gxmsc ) {
    my $key = $1;
    if ( $config =~ m/\G \s* = \s* /gxmsc ) {
        if ( $config =~ m/\G (true|false) \s*/gxmsc ) {
            my $value = $1 eq 'true' ? 1 : 0;
            printf "Key=[%s] Value=[%s]\n", $key, $value;
        }
        elsif ( $config =~ m/\G (\d+) \s*/gxmsc ) {
            my $value = $1;
            printf "Key=[%s] Value=[%s]\n", $key, $value;
        }
    }
}
```

Finally, we want to parse not a single line. We expect to parse multiple lines.
So we need a loop that continues parsing up to the end. We can achieve this with
the following loop.

```perl
while ( $config !~ m/\G\z/gxmsc ) {
    # code above inserted here ...
}
```

This loop continues until `\G` that is the position of the regex engine has reached
`\z`, the end of the string. Maybe we also want some error handling and we want to create a hash
instead of printing the result to the console. So finally we can write the following
function.

```perl
my $cfg = parseConfig("maximum = 1234\nverbose=true");

sub parseConfig($config) {
    my $cfg = {};

    while ( $config !~ m/\G\z/gxmsc ) {
        # Parse the Key
        if ( $config =~ m/\G (\w+) /gxmsc ) {
            my $key = $1;
            # Then there must be an equal sign
            if ( $config =~ m/\G \s* = \s*/gxmsc ) {
                # Value can be a bool
                if ( $config =~ m/\G (true|false) \s*/gxmsc ) {
                    $cfg->{$key} = $1 eq 'true' ? 1 : 0;
                }
                # or an integer
                elsif ($config =~ m/\G (\d+) \s*/gxmsc ) {
                    $cfg->{$key} = $1;
                }
                # unknown
                else {
                    die "Expected: Bool or Integer\n";
                }
            }
            else {
                die "Expected: =\n";
            }
        }
        else {
            die "Expected: key that is alphanumeric\n";
        }
    }

    return $cfg;
}
```

`parseConfig` returns a hash with the content.

```perl
{
    maximum => 1234,
    verbose => 1,
}
```

The good part of it is that we can easily expand the parsing. For example
adding the ability for another type of value is very easy. We just
need to add another `elsif` pattern for the value we want. We also
can produce decent error messages as we always knew what must come
next. With `pos` we have access to the current position of the regex engine
and we can produce a meaningful error message if needed.
