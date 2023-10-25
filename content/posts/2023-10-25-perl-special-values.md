---
layout: post
title: "Special values in Perl"
slug: special-values-in-perl
date: 2023-10-25
tags: [perl,patterns,closure]
description: How to create true special values in Perl using references.
---

Sometimes in programming we want to create special values that are distinct to anything else. Consider the following problem. We want to create a `print_colored` function, that can be given special values to change the terminal color. Here is one way to do it:

```perl
use Term::ANSIColor qw(color);

my $red   = "RED";
my $green = "GREEN";
my $blue  = "BLUE";

sub print_colored {
    for my $line ( @_ ) {
        print(color('red'))   && next if $line eq $red;
        print(color('green')) && next if $line eq $green;
        print(color('blue'))  && next if $line eq $blue;

        print $line;
    }
}

print_colored($green, "red", "\n", $blue, "foo\n");
```

This solution successfully prints the string "red" in the green color and "foo" in the blue color. But as you maybe can guess, it has some flaws in it. For example the following command will not give the result we expect.

```perl
print_colored($green, "RED", "\n", $blue, "foo\n");
```

It will print an empty line, followed by "foo" in blue color. The problem is that we only use strings for our special values. And strings are not really distinguishable from each other. Even other languages that provides Enums can have the same problem. Because in a lot of languages Enums are just a replacement for integers, not real special values.

One way to improve the solution would be to come up with extremly unlikely special strings. So instead of just "RED" we choose a string like "%SPECIAL_VALUE_RED%" and so on. But the problem never goes away. What happens if I really want to print "%SPECIAL_VALUE_RED%" to the console? It can create a lot of confusion to a user of the function `print_colored` if we create special values like this.

So here is a better approach. Instead of using special strings, we use array-references instead. Here is how it will look.

```perl
use Scalar::Util qw(refaddr reftype);
use Term::ANSIColor qw(color);

my $red   = [];
my $green = [];
my $blue  = [];

sub print_colored {
    for my $line ( @_ ) {
        my $type = reftype($line) // "";
        if ( $type eq 'ARRAY' ) {
            my $addr = refaddr($line);
            print(color('red'))   && next if $addr == refaddr($red);
            print(color('green')) && next if $addr == refaddr($green);
            print(color('blue'))  && next if $addr == refaddr($blue);
        }
        print $line;
    }
}

print_colored($green, "RED", "\n", $blue, "foo\n");
```

This solution works because each array-reference is always unique. As soon we create an array-reference it is saved in memory at a specific memory location. So we can safely use `refaddr` to distinguish values. No other reference will ever have the same address!

We can also improve that solution. Instead of variables we could use functions to return our colors. It's better because this way the reference will be hidden and cannot be changed anymore.

```perl
use Scalar::Util qw(refaddr reftype);
use Term::ANSIColor qw(color);

sub red   { state $x = []; return $x }
sub green { state $x = []; return $x }
sub blue  { state $x = []; return $x }

sub print_colored {
    for my $line ( @_ ) {
        my $type = reftype($line) // "";
        if ( $type eq 'ARRAY' ) {
            my $addr = refaddr($line);
            print(color('red'))   && next if $addr == refaddr(red);
            print(color('green')) && next if $addr == refaddr(green);
            print(color('blue'))  && next if $addr == refaddr(blue);
        }
        print $line;
    }
}

print_colored(green, "RED", "\n", blue, "foo\n");
```

It also feels more like a DSL to be able to just write `green` instead of `$green`. We also could supply a function like `is_red` for better checking. But then we cannot use `state` anymore and must rely on the older Perl way to achieve the same.

```perl
use Scalar::Util qw(refaddr reftype);
use Term::ANSIColor qw(color);

# Special values Red, Green and Blue
{ # <-- this creates a new lexical-scope
    my ($red, $green, $blue) = ([], [], []);

    sub red   { $red   }
    sub green { $green }
    sub blue  { $blue  }

    my $is_color = sub {
        my ($source, $color) = @_;
        my $type = reftype($source) // '';
        return
            ( $type eq 'ARRAY' and refaddr($source) == refaddr($color) ) ? 1 : 0;
    };

    sub is_red   { $is_color->($_[0], $red)   }
    sub is_green { $is_color->($_[0], $green) }
    sub is_blue  { $is_color->($_[0], $blue)  }
}

# at this place, variables $red, $green, $blue and
# $is_color are not accessible anymore

sub print_colored {
    for my $line ( @_ ) {
        print(color('red'))   && next if is_red($line);
        print(color('green')) && next if is_green($line);
        print(color('blue'))  && next if is_blue($line);

        print $line;
    }
}

print_colored(green, "RED", "\n", blue, "GREEN", "\n", red, "BLUE", "\n");
```

This now prints "RED" in green color, "GREEN" in blue color and "BLUE" in red color.

The variables `$red`, `$green`, `$blue` and `$is_color` are restricted to the enclosing `{}` that creates a new lexical-scope. Function defined with `sub ...` become global accessible and the variables they use become closures. That's why all of this works.

This way nobody can access and change those crucial variables anymore and we have functions like `red` and `is_red` to get those special values and check for them.

We can even further automatize the creation of those functions. But that is for another article.