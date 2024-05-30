---
layout:  post
title:   "Functions are interfaces"
slug:    functions-are-interfaces
date:    2024-05-30T00:00:00
lastmod: 2024-05-30T00:00:00
tags:    [fp,interfaces,perl,csharp]
description:
---

When I started to learn C# around 2013 there were certainly new things I
came into contact. Some of them had todo with the fact that I primarily used
Perl, a dynamic typed language, but in some sense the language features of
Perl were also far ahead of C# back then, leading to different solutions
and how you use a language.

Back at that time I realised that a lot of programmers were into a "newly"
thing. They teached that every interface should have only one method at best.

Like most stuff in the OO world it seems a lot of extremist always take it to
the extrem and saw it at the only way to do it. While I agree that one-method
interfaces are pretty good and there are reasons to stick to it, there are
also use-cases to have interfaces with many methods.

But anyway let's focus on that one-method interfaces for the moment. For
example you could come to the following idea. Everything that should be
printable should have an interface called "IPrintable" with only one single
method on it: `print_me`.

In C# that interface would look like this.

```csharp
interface IPrintable {
    void print_me();
}
```

While I think that it is a very good idea to limit yourself to just a single
method, there is a big problem with this kind of interfaces. Those kind of
interfaces are actually useless as they are not needed at all.

Think about it. When you have code that expects an `IPrintable` object, what
are the possible things you can do with that?

Thw answer: You just can call that one single method or not. And that's it.

It doesn't even matter how that method is named. Above I called the method
`print_me`, but it wouldn't even matter if that method is named `CallMe`,
`Execute`, `Bark`, `Drive`, `Invoke` or whatever fancy name you come up with,
all you can do is to execute that one method or don't.

I could even go so far and actually use reflections, read that one method and
just execute that method, this is how unimportant and generic the name of the
method of such one-method interfaces are.

Or in other words: **All one-method interfaces are basically just representations
of functions**.

We could make a generic interface for the above. Luckeliy we don't need, as C#
already ships one. The above interface could be basically replaced by the
[Action][Action] type.

Coming from Perl where I was used to work with functions and passing functions
as arguments I found it sometimes shocking how inefficent typical C# or Java
solutions are. (It is even a bigger shock to realize that C already had
function-pointers and you had nothing comparable like that in early C#
or Java versions).

If you understand the above and you start to learn design-patterns like [Strategy Pattern][SP]
or [Command Pattern][CP] you see how redundant they are. They are basically the
same Pattern and just represent a function in a complicated way because those
language those patterns was invented for didn't know [lambdas][LAMBDA] back at
that time.

Perl is a dynamic-typed language so being able to just pass any kind of function
that the user can execute maybe can be seen as a dynamic typed feature, but
it isn't.

When you start learning F# then you will realize that every function has a
signature. But different to let's say C# the name of a function doesn't matter.

For example the whole `IPrintable` interface above would be represented as
`unit -> unit` in F#.

`unit` is like `void` in C#, a little bit different but the detail isn't so much
important at that moment. All the signature tells you is that it represents
a function (represented by an arrow `->`) that has an `unit` as its input
and an `unit` as its output.

Languages like F# (ML, Ocaml, Haskell, ...) useses functions as interfaces. You
are able to pass any function that has the same type-signature.

You can pass an existing function that is somewhere defined, but you also can
create a function on the fly by using a [lambda][LAMBDA].

So don't forget, whenever you create one-method interfaces you should
either use [Action][Action] or [Func][Func] or however a function is defined
in your language.

# Example

Luckily C# was extended a lot in the later years, but when I started learning it,
coming from a Perl background I wasn't pleased at all. An example was *Sorting*.

Perl has a `sort` built-in that works by specifying a function that let's you
define how you wanna sort. You wanna sort by numbers?

```perl
my @sorted = sort { $a <=> $b } @numbers;
```

you have some kind of objects that should be sorted by age?

```perl
my @sorted = sort { $a->age <=> $b->age } @objs;
```

In C# on the other hand when you wanted to sort objects by some other criterium
you needed to create an [IComparer][Compare]. `IComparer<T>` is basically
just a function `T -> T -> int`, but you had to create a whole class somewhere
and implement that single `int CompareTo<T>(T,T)` method. It drived me nuts
how complicated that crap was.

Luckeliy, today you just can pass a `delegate` to `Sort` or use [LINQ's OrderBy][OrderBy].

But imagine how bad C# still would be if we did everything just *object-oriented*
only (because it's the only true/best paradigm). Sadly there are still enough
people out there believing in that kind of thinking.

Btw. a typical procedural [qsort](https://stackoverflow.com/questions/1787996/c-library-function-to-perform-sort)
from the stdlib in C also expects you to pass a function-pointer making it less
painfully to use as early C#.

[Action]: https://learn.microsoft.com/en-us/dotnet/api/system.action
[Func]: https://learn.microsoft.com/en-us/dotnet/api/system.func-1
[SP]: https://en.wikipedia.org/wiki/Strategy_pattern
[CP]: https://en.wikipedia.org/wiki/Command_pattern
[LAMBDA]: https://en.wikipedia.org/wiki/Anonymous_function
[Compare]: https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.icomparer-1
[OrderBy]: https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.orderby
