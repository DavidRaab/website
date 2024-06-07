---
layout:  post
title:   "Example: F# List is Anemic"
slug:    example-f-list-is-anemic
date:    2024-06-06T00:00:00
lastmod: 2024-06-07T00:00:00
tags:    [design,fsharp]
description:
---

In my previous article [Anemic Domain Model: Data and behaviour should be separated][Anemic]
I explain why you should stick to an Anemic Domain Model and which benefits it offers.

In this article we look into F# to see how it applies there. F# has many built-in
data-structures like many languages. One of the most used data-structure
is the `List` data-structure.

When you follow a typical object-oriented solution then maybe you would end up
with a `List` class, maybe you will consider some data as private, because the
way how to manage a list should be managed by the class. Usually this is done
for different reasons. Mostly for corectness, safety or some other reasons I
hear often.

Usually the promises are never held, because then we would have bug-free programs
without any security flaws in it. I have never seen a bug free program, especially
not in object-oriented code, it usually is the most hardest code I see to understand
and debug because of all that mutation and side-effects that appear everywhere.

When you use F# then there is a clear separation between the definition of
a List and its functions. This is not exactly the code how a List is defined
in F# but it would probably look something like this.

```fsharp
type list<'a> =
    | []
    | (::) of 'a * list<'a>
```

The definition you see here tells you the following. A list can be two different
things. Either it is `[]` or namely an *empty list* or it can be `::` usually
named a *cons* cell. A *cons* cell contains an element of type `'a` (a generic
type, that could for example be `int`) and the second element is another
`list` element of the same type `'a`. So that second element either can be
a *cons* cell again, or it is empty. This definition makes a list a
recurive data-structure.

The problem with the above definition is that you cannot create such a type
yourself in F#. Because F# has special syntax that is only allowed for this
built-in type. But you could come up with something similar like.

```fsharp
type mylist<'a> =
    | Empty
    | Cons of 'a * mylist<'a>
```

But anyway, the definition of the F# list is *open* in the sense that the definition
is not private or you need a special constructor, class or anything else to create
a list. Not only does it not restrict you in the creation, you also always can
travers a list yourself, create any function you like yourself or modify an
existing list.

Here is an interesting question. Can you create an invalid list with this definition?

The answer is. No, you can't. The definition of the `list` is choosen in such a
way that it only allows for valid lists. But sure it can be invalid if you wanna
create a list that only should contain numbers in a specific range. In *functional
programming* there is a saying.

> Make invalid states unrepresentable

and here you can see that approach. Already try to build your data in such a way
that it cannot be invalid. Sure there are limits to this approach or when
it becomes to complicated, but at least you should think more deeply
about your data.

<div class="info">
In the YouTube video <a href="https://www.youtube.com/watch?v=-J8YyfrSwTk">Effective ML</a>
this topic is discussed more deeply. Scott Wlaschin <a href="https://www.youtube.com/watch?v=2JB1_e5wZmU">Domain Modelling Made Functional</a> also is a good candidate.
</div>

But just having a List like above wouldn't be awesome, it would mean you have
to create a lot of useful functions yourself to work with a `List`. And creating
those can be tedious. Because of this, F# already ships with a `List` module
that provides ~100 functions to either create, modify or transform lists.

You can use them, but you don't have too. For example I can use the built-in
`map`.

```fsharp
let nums    = [1 .. 10]
let doubled = List.map (fun x -> x * 2) nums
let squared = List.map (fun x -> x * x) nums
```

Here you see the example I used in [Why map was not discovered in OOP][map-oo].
But you always can create your own functions if you like. Because the
data-structure is open and always traversable you just can create your own
`map` function.

```fsharp
let rec map f xs =
    match xs with
    | []        -> []
    | x :: rest -> (f x) :: (map f rest)
```

This function is not [Tail Recursive](https://en.wikipedia.org/wiki/Tail_call) but
this is not an article about proper, correct or fast implementation of `map`.

But needless to say is, that it is a good idea to try to re-implement all functions
that `List` offers by yourself. If you do, you will actually learn a lot of
functional programming, recursion and immutable data.

So here is an interesting thing you maybe know or not, depending whether you
used F# before or not. F# is not a pure functional language. It also offers OO
(otherwise interacting with .Net would be hard) but also allows side-effects
or throws exceptions.

Especially the later is something I don't like. For example there is `List.map2`
in F#. `List.map2` allows you to traverses through two lists at once and you
somehow can combine those values you want.

```fsharp
let sum =
    List.map2
        (fun x y -> x + y)
        [1;   2;  3;  4;  5]
        [10; 10; 10; 10; 10]
```

For example the code above creates the list `[11; 12; 13; 14; 15]`. It just adds
the numbers for the same index together. This is helpful, but what happens
when you provide two lists of different size to `List.map2`?

Or let me ask it differently. What do you expect or should happen in that case?
Because there are different answers and no answer is the right one.

F# decided to throw an exception in that case. So if you have the following code.

```fsharp
let sum =
    List.map2
        (fun x y -> x + y)
        [1;   2;  3;  4;  5]
        [10; 10; 10; 10; 10; 10]
```

then your code crashes. Because the first list only contains `5` elements
and the second list contains `6` elements.

I actually hate this behaviour. What I want to have is that it just works and
does so much work that is possible. If the end of one of the list is reached,
then it just stops returning what was computed so far.

So what can I do in F#? Sure I always can create my own module and just implement
my own `map2` that works this way. It looks like this.

```fsharp
module Lst =
    let rec map2 f xs ys =
        match xs,ys with
        | [],[]        -> [] // both lists are empty
        | [],_         -> [] // xs is empty
        | _,[]         -> [] // ys is empty
        | x::xs, y::ys -> (f x y) :: (map2 f xs ys)
```

Now instead of using `List.map2` i just can use my own module.

```fsharp
let sum =
    Lst.map2
        (fun x y -> x + y)
        [1;   2;  3;  4;  5]
        [10; 10; 10; 10; 10; 10]
```

and now it finally would return `[11; 12; 13; 14; 15]` instead of a program that
crashes. Again, that function is not [Tail Recursive](https://en.wikipedia.org/wiki/Tail_call).

Why can I do this? The reason for this is that the functions operating on a list
don't have to be part of the `list` definition itself. The functions are just
found in a Module named `List` and I can use them. Or don't.

When I write my own functions I always can create my own module. I also could
use some of `List` functions to implement my `Lst.map2` function.

Heck, if I wanted I also could create a `PositiveIntList` module that would only
allow positive integers as valid values. Every list created by `PositiveIntList`
would than be a valid `list<int>` but it doesn't mean that every `list<'a>` or
`list<int>` is a valid `PositiveIntList`.

By implementing `PositiveIntList` I also could use a lot of the functions found
in `List`.

All of that is possible because the definition of the data-structure and the
functions that create or work with a list are separated from each other and
I have full access to the data definition myself.

# A rant

The reason that I am not forced to add my own functions as a method, need to
inherit from a class, implement an interface, do some default interface
implementation, extension methods or all other crap makes it the most flexible
and easiest solution.

In fact all of the above usually was added because classes and methods are actually
to restrictive. A problem that for example doesn't even exist in C. Yeah, good old
C invented 1972 that is now over 50+ years old had basically the same abilities
that were introduced in C# 4 (Extension Methods) or C# 8 (Default Method Interfaces)
by just using structs and procedures operating on structs.

People always fear that when data is accessible and not *encapsulated*, usually
the reason why people stick to object-orientation, then everything breaks and it
would contain bugs. Sure it can cause malicious code, but nothing can prevent
bad code and bugs, not even trying to encapsulate data. Otherwise we wouldn't
had bugs in object-oriented code. But doing that *encapsulation* stuff makes working,
extending and pretty much *everything* else so much more complicated that
it isn't worth it. It caused more problems as it ever solved.

C# basically needed to invented dozens of features in the last 20 years just
to get the features back you lost by that whole idea of proper encapsulation.

I did write Perl code for nearly 15 years in web-applications, one thing
that Perl OO is often critizied is that this encapsulation is missing
and you cannot make data private.

Do you wanna know how often I encountered bugs because data was not private?
Not a single time in 15 years of Perl development.

Don't invent problems so you then can be proud of solving them.

# Related Posts

* [Object-Oriented Programming in C]({{< ref 2023-11-01-object-oriented-programming-in-c.md >}})
* [Anemic Domain Model][Anemic]
* [Why map was not discovered in OOP][map-oo]

[Anemic]: {{< ref 2024-06-04-anemic-domain-model-data-and-behaviour-should-be-separated.md >}}
[map-oo]: {{< ref 2024-06-05-why-map-was-not-discovered-in-objectoriented-programming.md >}}
