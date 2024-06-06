---
layout: post
title: "Map operates on functions!"
slug: map-operates-on-functions
date: 2022-10-27
tags: [FSharp,generics,functor]
description: "map is for functions not for data-types."
keywords: f#, fsharp, generics
---

`map` operates on functions not on data-types like `list`.

When I started learning F# I had some problems understanding the `map` function. Don't get me wrong. As
a Perl programmer `map` is built into the core language (long before other languages adopted the idea)
and I used and understanded it very well in Perl. This `map` is the same as `List.map` that I also
immediately understood.

But in F# you will encounter `Result.map`, `Option.map`, `SomethingElse.map`, ... and I didn't get what
those `map` function did for those types!

The problem was not really that I misunderstood `map`. I just understood a specific implementation
but not the more general concept. Or in other words: I didn't learned it properly. I learned
the definiton:

> `map` is a higher-order function that executes a function on every element on a list producing
> a new list this way.

but that definition is wrong. It just explains a specific `map` implementation or more general
what `List.map` does, but not the general concept behind `map`. A correct definition would be more like:

> `F.map` is a higher-order function that transforms a function into a new function working on type `F`.

The first thing you have to realize is. That `map` is actually not about a `list`, `option` and so on.
While you must implement it for the different types, it really is about the function you pass as the first
argument. `map` really is about transforming a function. So let's get started.

# `map` as a two argument function

The type of `List.map` looks like this

    ('a -> 'b) -> list<'a> -> list<'b>

When we look at `map` as a two argument function it does exactly as my first description. We interpet the function
this way.

    (1. Argument)   (2. Argument)   (Return Value)
    ('a -> 'b)   -> list<'a>     -> list<'b>

`map` takes a function `('a -> 'b)` and a `list<'a>` and applies the function to every element and returns the
`list<'b>`. But thinking this way makes it hard to understand `Async.map` or `Option.map` as those types don't
have a list of values. They usually only have one value. When I tried to implement those functions I always asked
myself: Are those implementation right? How do I know if the implementation i gave is right? What does *right*
or *wrong* even mean?

Maybe `option<'a>` is not so hard to understand, maybe like me you heard it before that you can think of `option<'a>`
as just a `list` with a single value. Maybe you can think of `Async` the same. But that doesn't really explain
the concept good enough to understand, and I think bad examples or analogies can even turn out to make things
even harder to understand.

I got enlightenment when I throw away what I already knew (I emptied my glass) and looked at `map` as a
one-argument function.

# `map` as a one-argument function

One important thing in F# is that every function is just a one argument function. There doesn't exists
functions with multiple arguments. We can think of the `map` function as a one-argument function that returns
a new function.

    (1. Argument)    (Return Value)
    ('a -> 'b)    -> (list<'a> -> list<'b>)

The additional parenthesis around `list<'a> -> list<'b>` are not needed here as the arrow `->` is right-associative
by default. But I added them for clarity. Now we have a `map` function taking a `'a -> 'b` function and it returns
us another new function `list<'a> -> list<'b>`. Think of every function just as a single input and output. Now
you can think of `List.map` this way.

    Input:       'a  ->      'b
    Output: list<'a> -> list<'b>

What `List.map` really does is just transform the input function. It somehow wraps whatever you have as
input and output and puts a `list` around it.

You have a function `async<url> -> async<option<string>>`? When you pass that to `List.map` you get a
new function with the signature: `list<async<url>> -> list<async<option<string>>>`.

    Input:       async<url>  ->      async<option<string>>
    Output  list<async<url>> -> list<async<option<string>>>

This is the same for any other `F.map` function. A `Result.map` for example turns a `'a -> 'b`
into a `Result<'a> -> Result<'b>` function.

    Input:         'a  ->        'b
    Output: Result<'a> -> Result<'b>

# map transformer

Because of this you always can think of `map` as a function transformer. So when you have a function
that squares an int.

```fsharp
// int -> int
let square x = x * x
```

you can turn this function into a new function that squares a list of ints.

```fsharp
// list<int> -> list<int>
let squareList = List.map square
```

and when we pass `square` to `Option.map`? Then we just get a new function back that works
on an optional value.

```fsharp
// option<int> -> option<int>
let squareOption = Option.map square
```

Even when we pass all two arguments to `List.map` at once (without partial application)
we still can think of it as a function transformer. Let's say we write

```fsharp
let x = square 4
```

to square a `4`. But what happens if you have a list of values? We write:

```fsharp
let xs = List.map square [4;5;6]
```

Think of it the following way: You still execute `square` but the first argument of your function
now turns into a list because you put `List.map` in front of it. The return value will
also be a `list`.

I often look at those functions the following way.

```fsharp
let x =            square  4
let x = List.map   square [4;8]
let x = Option.map square (Some 4)
let x = Result.map square (Ok 4)
```

In all those cases we just execute the `square` function and the argument we pass to `square` now can
be of type `F` whichever `F` we choose with `F.map`.

This is one reason why I prefer to write

```fsharp
List.map   square  numbers
Option.map square (Some 4)
Result.map square (Ok 4)
```

instead of

```fsharp
value |> List.map   square
value |> Option.map square
value |> Result.map square
```

I call pipe the *Object-orientet invocation operator* and it hides what it does. It looks like `value` is the
important aspect as if we would call the method `map` on `value`. But the values or objects have
no importance even in OO code. When we read code we want to know what the code does. `value` is
not anything that *does* anything. We don't even care about the specific values inside of `value` because
`value` is a variable that can have any value depending n when the code is executed, but source code is
not executed when we look at it or write the code. Not even `map` is the most important aspect in those
lines. It is `square` that has the most importance, followed by `map` as it *augments* `sqaure` and
the `value` we want to `square` is of least importance.

Because `map` is really about the function you use, it also works well with any number of arguments.
This invocation:

```fsharp
let r = List.map6 func a b c d e f
```

for example just executes `func` with all six arguments and the return value being a `list`. This is what I mean
with **function transformators**. And it exactly resembles what you would write if `func` would already
expect `list` values. You would just delete the `List.map6`

```fsharp
let r = func a b c d e f
```

# map on inner-type

You also can think of `map` as a function that lets you work on the inner-type. This is more
appropiate with the two-argument version. For example when you have a `list<int>` and a
function that does something with an `int` then you can use `List.map` for this.

This goes hand in hand with other types. You got an `Async<int>` and you have a function that
does something with an `int` but not `Async<int>`? Just use `Async.map`!

You have a `Foo<int>` and want to work with the `int` inside the `Foo` type? Just use `Foo.map`!

This mind-model also works for `List.map2`, `List.map3` and so on. It lets you work on the inner
type but it let's you use multiple lists at once. As an example `String.replicate` expects
a `int` and a `string` as its arguments. So you can write.

```fsharp
// "BB"
String.replicate 2 "B"

// ["A"; "BB"; "CCC"]
List.map2 String.replicate [1;2;3] ["A";"B";"C"]
```

So the function you pass to `map` always just sees one of the inner type of `xs` and `ys`.

# Conclusion

Think of `F.map` as either **upgrading** a function that puts wrappers around the
input and output types of whatever `F` is.

Or think of this function as something that works on the inside of the type. This means, something is
unwrapped and wrapped again. Like a list is unwrapped (unfolded) and then wraped (folded) again. But
you don't have to know how that is done (even for other types). Just think of it as don't caring
for the wrapped type at all. Just do something of what's inside of the type.

But definitely never think of `map` as a function that iterates through every element of a list. This
is what you probably know because that's what all other languages usually provides. But you have to
abondon this idea because its wrong and just stands in your way to really understand what `map` is all
about.

# Related Posts

* [Functions are interfaces]({{< ref 2024-05-30-functions-are-interfaces.md >}})
* [Understaing map]({{< ref 2016-03-27-understanding-map.md >}})
* [Why map was not discovers in OOP]({{< ref 2024-06-05-why-map-was-not-discovered-in-objectoriented-programming.md >}})
