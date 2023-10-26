---
layout: post
title: "Meta-Programming in Perl"
slug: meta-programming-in-perl
date: 2023-10-26
tags: [perl,metaprogramming,design-pattern]
description: Introduction to Meta-programming in Perl. Understanding Type Globs in Perl.
---

In my last article about [Special Values in Perl]({{< ref 2023-10-25-perl-special-values >}})
I explained how we can create true distinct values that can be safely distinguished
by other values. The final result looked like this:

```perl
use Scalar::Util qw(refaddr reftype);

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
```

While the code above works, i am unhappy with it. Not because it doesn't work or
has some flaws in it. Mainly because it is not really reusable. Let me explain.

Consider you want to create additional special values. How would you solve it?
At the moment you would just Copy & Paste code, again and again ...

I think Copy & Paste is bad, a lot of people will probably agree. I could fix
that problem by just saying: This is the **Special Values Design-Pattern**. And suddenly
it would become *Good Practice* for a lot of people. Because obviously *Design
Patterns* are awesome and good practice, right?

I somehow find it interesting that giving just another name to *Copy & Paste*
and naming it *Design Pattern* suddenly makes shit a lot better for many
programmers, but that is just the way how things are. I would claim that every
*Design Pattern* (or just Copy & Paste) is a flaw that a language is not powerful
enough. This begs the question: Can we make the above code somehow reusable? Can
we create a function that creates the *Special Values* including
the functions all by itself?

What I have in mind is the following: I only want to write something like

```perl
enum('red', 'green', 'blue');
```

and the `enum` function should create the functions `red`, `green`, `blue` including
`is_red`, `is_green` and `is_blue` all by itself. Here I will show you, how to do
this in Perl. This kind of programming is named [Metaprogramming](https://en.wikipedia.org/wiki/Metaprogramming).

# Typeglobs & Symbol Table

To understand Metaprogramming in Perl, we first need to understand the [typeglob](https://perldoc.perl.org/perlref#Typeglob-Slots) and the [Symbol Table](https://perldoc.perl.org/perlmod#Symbol-Tables)
in Perl.

As you probably know, Perl has different types. Here I don't mean `int`, `float`
and so on, more the ability that Perl distinguish between a Scalar, Array, Hash,
Filehandle (IO) and a Subroutine (Code).

```perl
our $foo = "foo";          # Scalar
our @foo = ("foo");        # Array
our %foo = (foo => "Bar"); # Hash
open foo, '<', 'file';     # IO
sub foo { return "foo" }   # Code
```

All of those are named `foo`, and still they are completely different from each
other. You can safely use `$foo` along `@foo` or a function `foo()` without a
problem in Perl. Something that will not work in JavaScript for example and
many other languages.

```js
var foo = 10;

// This will redefine "foo" to a function
function foo() { return "Hello" }
```

Next is the Symbol Table. So what is the Symbol Table? As a user of a programming
language you maybe never asked this question, as it maybe seems dumb. But when
you type `$foo` or `@foo` or execute a function `foo()` where are those
variables, functions and so on saved? In the Symbol Table!

The Symbol Table contains all variables, functions and so on. So whenever you
type a variable name, it is looked up in the Symbol Table. In Perl the
Symbol Table itself contains *Typeglobs*. So what are Typeglobs?

A Typeglob contains all our special values we have in Perl (Scalar, Array, Hash, ...)
and can be used like a Hash in Perl. As an example. All of the above `foo` values
we created are found in the typeglob named `*foo`. And we can get *references*
to those values this way.

```perl
my $scalar_ref = *foo{SCALAR};
my $array_ref  = *foo{ARRAY};
my $hash_ref   = *foo{HASH};
my $io_ref     = *foo{IO};
my $code_ref   = *foo{CODE};
```

This is the reason why Perl can distinguish `$foo` from `@foo`. `$foo` is saved
in `*foo{SCALAR}` while `@foo` is saved in `*foo{ARRAY}` and so on.

# Manipulating the Symbol Table

Here comes the interesting part. The Symbol Table itself can be manipulated in Perl
at runtime.

```perl
no strict;

*foo = \"bar";                 # Assigns to *foo{SCALAR}
*foo = ["foo", "bar"];         # Assigns to *foo{ARRAY}
*foo = { foo => 1, bar => 2 }; # Assigns to *foo{HASH}

say $foo; # prints "bar"
say @foo; # prints "foobar"
say %foo; # prints either "foo1bar2" or "bar2foo1"
```

First of, when we assign to `*foo` we just assign it a reference. Depending
on the type of the reference, it will save it in the correct slot. But
we also can access values through the typeglob if wanted/needed.

```perl
say ${ *foo{SCALAR} }; # prints "bar"
say @{ *foo{ARRAY}  }; # prints "foobar"
say %{ *foo{HASH}   }; # prints either "foo1bar2" or "bar2foo1"
```

Consider that accessing the typeglob like `*foo{ARRAY}` returns a *reference*.
So we need to *de-reference* it to access the values. Hence the `@{ ... }` or
`${ ... }` around the typeglobs.

Writing `*foo` is hard-coded, but Perl also allows to use a *string* to access
the *typeglob*.

```perl
no strict;

my $name = "foo";
*$name = \"bar";
*$name = ["foo", "bar"];
*$name = { foo => 1, bar => 2 };

say $foo; # prints "bar"
say @foo; # prints "foobar"
say %foo; # prints either "foo1bar2" or "bar2foo1"
```

Writing `*$name` is a shortcut syntax for writing. `*{ $name }`. We can consider
the `*` as accessing the type glob and pass the name we want to access as
a variable name.

First note the [no strict;](https://metacpan.org/pod/strict). In Perl it usually is a good idea to `use strict;`.
Also requiring a new Perl version like `use v5.36` enables `use strict;` or loading
some module like [Moose](https://metacpan.org/pod/Moose) does so.

In 99% this is good because it deactivates certain Perl syntax to disable
manipulating the symbol table. But in this case that is exactly what we want. With
`no strict;` we can re-enable those features (or disable the disabling of features).

With all of this knowledge, now let's write our `enum`

# Implementing enum

First of I would create a single function only creating one special value
at a time.

```perl
use v5.36;
use Scalar::Util qw(refaddr reftype);
use Term::ANSIColor qw(color);

sub create_special($name) {
    # disable strict refs inside this function
    no strict 'refs';
    # our special value
    my $special = [];

    # create $name function that returns our special value
    *{ $name } = sub {
        return $special;
    };

    # create "is_"$name function, checking if passed value is
    # identical to $special
    *{ "is_" . $name } = sub {
        my ($value) = @_;
        my $type = reftype($value) // "";
        # check if arrayref was passed
        if ( $type eq 'ARRAY' ) {
            # check if addresses are the same
            if ( refaddr($value) == refaddr($special) ) {
                return 1;
            }
            else {
                return 0;
            }
        }

        return 0;
    };
}
```

This would now allow writing `create_special("foo")` that creates a `foo()` and
`ìs_foo()` function returning and checking our special value. Now `enum` can
just be implemented by calling this function in a loop.

```perl
sub enum {
    for my $name ( @_ ) {
        create_special($name);
    }
}
```

finally, we can write:

```perl
enum('red', 'green', 'blue');
```

The implementation of `print_colored` stays the same.

```perl
sub print_colored {
    for my $line ( @_ ) {
        print(color('red'))   && next if is_red($line);
        print(color('green')) && next if is_green($line);
        print(color('blue'))  && next if is_blue($line);

        print $line;
    }
}
```

But there can be one problem. When we now just try to write.

```perl
print_colored(green, "RED", "\n", blue, "GREEN", "\n", red, "BLUE", "\n");
```

we get the following compile-time error.

```
Bareword "green" not allowed while "strict subs" in use at ./metaprogramming.pl line xxx.
Bareword "blue" not allowed while "strict subs" in use at ./metaprogramming.pl line xxx.
Bareword "red" not allowed while "strict subs" in use at ./metaprogramming.pl line xxx.
Execution of ./metaprogramming.pl aborted due to compilation errors.
```

The reason for this is the following:

1. Perl has a compile-time and a runtime-stage.
2. The function `red`, `green`, ... are created at the runtime-stage.
3. When the compiler hits the `print_colored` line at the compile-time,
   the compiler don't know about the functions `red`, `green` and `blue`. Only
   functions that the compiler knows so far at the compile-time are allowed
   to be called without parenthesis. (This usually fetches a lot of errors).

There are two ways to fix this problem.

## First

We could add parenthesis. This way the compiler knows that we want to
call a function and those errors about the *bareword* goes away. That's also
the reason why we never got an error writing the `print_colored` function even
the fact we used the `is_red()`, ... functions there.

```perl
print_colored(green(), "RED", "\n", blue(), "GREEN", "\n", red(), "BLUE", "\n");
```

## Second

We can add `BEGIN { }` around our `enum` call.

```perl
BEGIN {
    enum('red', 'green', 'blue');
}
```

[BEGIN](https://perldoc.perl.org/perlmod#BEGIN%2C-UNITCHECK%2C-CHECK%2C-INIT-and-END) is a special block in Perl that tells the compiler to immediately run
the code before continuing compiling the rest of the program. This way the function
gets created at compile-time. When the compiler sees the line.

```perl
print_colored(green, "RED", "\n", blue, "GREEN", "\n", red, "BLUE", "\n");
```

it knows about the `green`, `red`, ... functions and don't complain about
*barewords* anymore.