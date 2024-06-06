---
layout:  post
title:   "Why map was not discovered in Object-Oriented Programming"
slug:    why-map-was-not-discovered-in-object-oriented-programming
date:    2024-06-05T00:00:00
lastmod: 2024-06-06T00:00:00
tags:    [design,map]
description:
---

When I started Perl programming back around 2003 one thing that Perl had,
what most language didn't, was the existence of the `map` function.

In today world of programming from JavaScript, C#, Java and so on, the concept
nowadays is so common that nearly every language has it, and usually every programmers
understands and use it.

That was not always the case. I can remember when LINQ was introduced in C# how
a lot of people was complaining how from know on C# would be a language that nobody
else never can read or understand anymore. The same happened in Java when
Java 8 introduced it's Stream API.

Also in the past I got a lot of complaining from other Perl programmers when
I used `map` instead of just using a `for` loop. "A `for` loop
is more readable", did those people say.

What they really meant to say was: I don't know the concept of `map` and because I
never learned and understand it, I think it is not readable.

But anyway, in this article I want you to show something different. Let's assume
Perl hadn't `map` built-in. Could we implement it ourself and would we do it?

# What `map` does

Let's say we want to go through an array `@nums` and just double every number in it.
But we also want to put the doubled number into a new array, usually we would write
code like this.

```perl
my @doubles;
for my $x ( @nums ) {
    push @doubles, $x * 2;
}
```

I guess its not too bad. Let's say we now wanna square each number.

```perl
my @squared;
for my $x ( @nums ) {
    push @doubles, $x * $x;
}
```

Yeah, it is short, but you know what. When you have written like 1000 of such
loops even that becomes too long. You wanna have it shorter. Most of the code
is anyway the same. As Perl supports [Anonymous Functions][Lambda] we could come
up with a very quick implementation. It could look like this.

```perl
sub mymap($f, @array) {
    my @new;
    for my $x ( @array ) {
        local $_ = $x;
        push @new, $f->();
    }
    return @new;
}
```

after that we could write the following code.

```perl
my @nums = 1 .. 10;

my @doubles = mymap sub { $_ * 2  }, @nums;
my @squared = mymap sub { $_ * $_ }, @nums;
```

we also could use a [Prototype][Prototype] in Perl.

```perl
sub mymap :prototype(&@) {
    my ($f, @array) = @_;
    my @new;
    for my $x ( @array ) {
        local $_ = $x;
        push @new, $f->();
    }
    return @new;
}
```

with this prototype we now can write it exactly like the Perl built-in `map`
function and dismiss the `sub` keyword.

```perl
my @nums = 1 .. 10;

my @doubles = mymap { $_ * 2  } @nums;
my @squared = mymap { $_ * $_ } @nums;
```

So would we come up with this kind of abstraction? I guess yes, at least me
I am always happy when I am able to achieve the same with less code. I
like it because the above `mymap` reduces the code to it's *Moving Parts*.

*Moving Parts* means that you only need to write the stuff that differs. When
you look at both previous `for`-loop solutions, you see that most code is actually
the same. When you do a diff on it than only small characters differs. Mainly just
what we did with `$x`. To create the `@doubled` array we wrote `$x * 2` and
in the squared version we wrote `$x * $x`.

With the `mymap` function on the other hand we can strip it down to only write
this piece of code. Or to it's core, we just write the *Moving Parts*. This
overall makes code easier to read and understand as you only need to read
and understand the *important parts*.

The whole idea of **go through an array, apply some code to it, and put the result
into a new array** now has become an abstraction known as `map`. Sure, if somebody
don't know what `map` does he don't understand it. But that doesn't make it
harder to understand. The same would happen if you don't know what a `for` loop
does and trying to read a `for`-loop. Or trying to read any foreign language
you don't know of.

So while that abstraction is useful, it doesn't mean we automatically would use
this abstraction. If we would use it or not depends on something different. It
usually depends on the fact if the abstraction leads to writing less/shorter
code.

As `mymap` turns 4 lines of code into just a single line, usually there is a
reason to use `map`. This is an important part, maybe the most important.

To understand this, let's see how an Object Oriented solution would look like.

# An Object Oriented map

So when I make the assumption of an object-oriented solution I just think about
the typical features a "pure" (whatever that means) OO language like
Java or C# was at the beginning of its creation.

For some reason in those **everything is an object** languages where everything
is an object. Everything is an object, except functions. So you cannot pass
functions as arguments. Funny, heh?

So let's assume Perl hadn't the ability to pass a subroutine and we also couldn't
create a `sub { ... }` code reference on the fly. No problem, we could come up with
a class. For example I could come up with a `Doubled` class.

```perl
package Doubled;
use 5.036;

sub new($class) {
    return bless([], $class)
}

sub invoke($class, $x) {
    return $x * 2;
}
```

Here I am creating a class and put my code into the `invoke` method. By
creating that class I now can do.

```perl
my $double = Doubled->new();
```

The only purpose of the `Doubled` class is actually that I can create an object
from it. Why? Because then I can pass that object as a value and with that
also pass my `invoke` method as an argument. In this case that class is completely
obsolete because it would actually never makes sense to create a second object from it.

In a class-based solution `mymap` would change to something like this.

```perl
sub mymap($f, @array) {
    my @new;
    for my $x ( @array ) {
        push @new, $f->invoke($x);
    }
    return @new;
}
```

`mymap` itself wouldn't be much of a difference. We just expect any object
that has an `invoke` method, and call that one.

So would we come up with this kind of abstraction in an object-oriented
language?

Maybe not, because in an object-oriented language this piece of code.

```perl
my @squared;
for my $x ( @nums ) {
    push @doubles, $x * $x;
}
```

would be a lot more shorter than writing this kind of code.

```perl
package Squared;
use 5.036;

sub new($class) {
    return bless([], $class)
}

sub invoke($class, $x) {
    return $x * $x;
}

package main;

my $square = Squared->new;
my @squared = mymap $square, 1 .. 10;
```

why should we ever want to turn 4 lines of code into 10+ lines of code? The
whole purpose of creating a class just so we can pass the code `$x * $x` as
a value is utterly absurd.

And still, people did this kind of stuff. Because sometimes this
still offers value. Sure it can be really helpful in a lot of cases that you can
pass code as values instead of just data. *functional-programmers* do this
*a lot*. I can come up with maybe a dozens of use-cases where this is helpfull.
Doing this kind of thing is named the [Strategy Pattern][SP] in object-oriented
programming. A poor man design-pattern when your language don't support
[lambdas][Lambda].

A pattern that at least in C# or Java have become obsolete and should never
be used anymore.

Sure in Java or C# you also need to add an interface definition because of its
static-typing, making the solution even more bloated.

And this is why `map` would basically never exist in an *object-oriented* language.

The language itself gives you no ability to abstract that 4 lines of for-each loop
into a shorter more readable code. It usually only can make it more complex
with more lines of code, but that is usually not considered as *good* among
programmers.

# A Question

Sometimes people think that passing functions and creating them on the fly isn't
object oriented at all.

Well, do you know that whenever you pass an object you basically can think of it
as passing complex data and maybe a hundred of methods as a single object as a single
argument?

So why is passing a single function as an object not object-oriented? Because it is
just a single function? How many methods must an object have to be considered an
*object*? Two? Three? Twelve?

Isn't passing a single function exactly and the best fullfilment of the object-oriented
[Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)?

# Related Posts

* [Functions are interfaces]({{< ref 2024-05-30-functions-are-interfaces.md >}})

[Lambda]: https://en.wikipedia.org/wiki/Anonymous_function
[Prototype]: https://perldoc.perl.org/perlsub#Prototypes
[SP]: https://en.wikipedia.org/wiki/Strategy_pattern