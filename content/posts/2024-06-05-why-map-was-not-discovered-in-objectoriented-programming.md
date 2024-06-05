---
layout:  post
title:   "Why map was not discovered in Object-Oriented Programming"
slug:    why-map-was-not-discovered-in-object-oriented-programming
date:    2024-06-05T00:00:00
lastmod: 2024-06-05T00:00:00
tags:    [design,map]
description:
---

When I started Perl programming back around 2003 one thing that perl had,
what most language didn't, already was the existence of the `map` function.

In today world of programming from JavaScript, C#, Java and so on, the concept
nowadays is so common that nearly every language has it, and usually every programmers
understands and use it.

That was not always the case. I can remember when LINQ was introduced in C# how
a lot of people complaining how from know on C# would be a language that nobody
else never can read or understand anymore. The same happened in Java when
Java 8 introduced its Stream API.

Not even that, also in the past I got a lot of complaining from other Perl
programmers when I used `map` instead of just using a `for` loop. "A `for` loop
is more readable", did those people say.

What they really said was: I don't know the concept of `map` and because I
never learned and understand it, I think it is not readable.

But anyway, in this article I want you to show something different. Let's assume
Perl hadn't `map` built-in. Could we implement it ourself, and would we do it?

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
function.

```perl
my @nums = 1 .. 10;

my @doubles = mymap { $_ * 2  } @nums;
my @squared = mymap { $_ * $_ } @nums;
```

So would we come up with this kind of abstraction? I guess yes, at least me
I am always happy when I know how to achieve the same solution by reducing the
code of its *Moving Parts*. That means i just want to write stuff that matters
so it is easier to see the difference between two pieces of code.

Compared to the `for` loop i gues 90% of all characters are usually just boilerplate
und you just copy & Paste them, again and again. `mymap` on the other hand reduced
it to only the the code that matters.

But another reason why we would come up with this solution is simply because the
result is *shorter* then the original was. And this is maybe the most important
point of it.

To understand this, let's see how an Object Oriented solution would look like.

# An Object Oriented map

So when I make the assumption of an object-oriented solution I just think about
the typical features a "pure" OO language like Java or C# was at the beginning
of its creation.

For some reason in those **everything is an object** languages where everything
is an object. Everything is an object, except a function. So you cannot pass
functions as arguments. Funny, heh?

So let's assume Perl hadn't the ability to pass a subroutine and we also couldn't
create a `sub { ... }` code reference on the fly. No problem we could come up with
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

So what I basically do is the following. I just create a class, and I always
expect that this class implements a method `invoke`. By creating that boilerplate
class i now can do.

```perl
my $double = Doubled->new();
```

I know can create an object that keeps hold of my single function. And because
it is an object, i finally can pass that as an argument to something else.

Finally a `mymap` would look like this.

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
that has an `invoke` method and call that one.

So would we come up with this kind of abstraction in an object-oriented
language?

No we wouldn't because in an object-oriented language this piece of code.

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

and still, people did this kind of stuff. Because sometimes this kind of code
still offers value. What you see here is basically just the [Strategy Pattern][SP]
a poor man design-pattern when your language don't support [lambdas][Lambda].

A pattern that at least in C# or Java should be mostly deprecated and never
be used anymore.

Sure in Java or C# you also need to add an interface definition because of its
static-typing, making the solution even more bloated.

And this is why `map` would basically never exist in an *object-oriented* language.

The language itself basically offers no solution in making the code shorter. It
only offers a way of doing the same with more lines of code. Something that is
usually not considered good by programmers.

# A Question

Sometimes people think that passing functions and creating them on the fly isn't
object oriented at all.

Well, do you know that whenever you pass an object you basically can think of it
as passing complex data and maybe a hundred of methods as a single object as a single
argument?

Is passing a single function as an object not object-oriented because it is
just a single function? Must an object at least have two, tree or six methods
to be object-oriented?

Isn't passing a single function exactly and the best fullfilment of the object-oriented
[Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle)?

# Related Posts

* [Functions are interfaces]({{< ref 2024-05-30-functions-are-interfaces.md >}})

[Lambda]: https://en.wikipedia.org/wiki/Anonymous_function
[Prototype]: https://perldoc.perl.org/perlsub#Prototypes
[SP]: https://en.wikipedia.org/wiki/Strategy_pattern