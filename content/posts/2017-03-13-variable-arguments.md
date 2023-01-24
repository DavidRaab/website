---
layout: post
title: "Variable Arguments in F#"
slug: variable-arguments-in-fsharp
date: 2017-03-13
tags: [FSharp]
description: "This article explains how you can create functions with variable arguments, and why you should avoid them."
keywords: f#, fsharp, functions, variable arguments, paramarray
---

One question that appears in F# from time to time is: How do you create a
function that expects a variable amount of arguments?

**A short answer is:** You can't do that.

**A longer and correct answer:** You can do it with (static) methods.
But you probably don't want to use this and look for an alternative.

First we should look at the difference between an F# function and a (static)
method.

# F# Functions vs. (static) methods

Usually I don't distinguish between those two as both just execute
some code and return some value. But in this case we must differentiate
them. An F# function is any function defined with the `let` keyword.
F# functions are usually defined inside of modules or inside of other
functions.

A (static) method on the other hand is part of a class definition. The
definition is different, but using (static) methods or functions can look
the same. The biggest difference is that (static) methods often use
tupled-syntax while F# functions use currying. But you are not restricted
to the one or other.

You can use currying and a tupled syntax in F# functions.

```fsharp
module SomeModule =
    // Currying
    let funcC a b = ...

    // Tuple
    let funcT (a,b) = ...
```

You could call both functions like this:

```fsharp
SomeModule.funcC x y
SomeModule.funcT (x,y)
```

The second version looks a lot like calling a function in other languages
from C, C#, Java and so on and this is not an accident. But F# is
really consistent in its syntax. Whenever you see parenthesis
and values separated with commas then you really just define a tuple.
Because of this you also could write:

```fsharp
let args = (x,y)
SomeModule.funcT args
```

This is something you cannot do in C, C#, Java and so on. Calling a function
looks the same but the meaning is different. All of this is also possible
with (static) methods.

```fsharp
type SomeClass() =
    static member funcC a b = ...
    static member funcT (a,b) = ...
```

you can call it in the same way

```fsharp
SomeClass.funcC x y
SomeClass.funcT (x,y)
```

So, why is any of this important?

1. In F# you usually want to work with curried functions.
2. Variable Arguments are only supported with tupled (static) methods.

# Variable Arguments

First lets focus on the second point. So we only can use variable
arguments if we create a class, and use tupled syntax. As a light
example let's build a `max` function that returns the biggest
element from all arguments we pass to it.

```fsharp
type Util() =
    static member max([<System.ParamArray>] xs) = Array.reduce max xs

Util.max(1,2,3)         // 3
Util.max(1,10,3,20,4,6) // 20
Util.max(3,2,1)         // 3
```

The concept of a variable argument function is easy. You just use a normal
argument and add the attribute `[<System.ParamArray>]` to it. Only the
last argument can be flagged with the attribute. And finally, you receive all
arguments as an array.

# Why you should avoid variable arguments

Previously I said that when you use parenthesis and separate values with
comma it is a tuple. In fact this kind of consistency is broken with `ParamArray`.
You can see the difference in this extended example:

```fsharp
type Util() =
    static member max([<System.ParamArray>] xs) = Array.reduce max xs
    static member max4(a,b,c,d)                 = Array.reduce max [|a;b;c;d|]

let nums = (1,10,30,15)

Util.max4 (1,10,30,15) // 30
Util.max4 nums         // 30
Util.max  (1,10,30,15) // 30
Util.max  nums         // (1,10,30,15)
```

Both `Util.max4` calls return `30` because this function expects a tuple with
four arguments and we pass this to `Util.max4` in both cases.

But the `Util.max` calls completely differ. In the first example we really pass
four arguments, but in the second `Util.max` example we really pass a single
value, a tuple containing four elements.

`ParamArray` really is an inconsistency in the language. I wouldn't even say this
was a bad decision. If you use a variable arguments function defined in C# from F#
it absolutely makes sense to break this consistency. In fact this inconsistency can
even feel more consistent. A C# static method that you call from F# with four
arguments looks like:

```fsharp
Class.Func(a,b,c,d)
```

a static method with variable arguments that you also pass four arguments also looks like:

```fsharp
Class.Func(a,b,c,d)
```

So it is consistent or inconsistent depending from which way you look at it.

But rather arguing with consistency the important part I consider is that it
behaves differently and you cannot see that from the code. When I look at
code like `Class.func(a,b,c,d)` I could assume that it is a function that
expects four arguments. It isn't obvious that I can add a fifth argument
or probably use less arguments.

The biggest problem in my opinion is that most of the time you already have
a collection like a list and you just want to pass that list to a function.

```fsharp
let list = [20;14;37;16]

Util.max [20;14;37;16] // [20;14;37;16]
Util.max list          // [20;14;37;16]
```

If you already have a list, then variable arguments doesn't help you at all.
In fact you must write other code like:

```fsharp
List.reduce (fun acc x -> Util.max(acc,x)) list // 37
```

This is pretty much exactly how `Util.max()` itself is implemented! On top
it's a bit longer, because you cannot just write
`List.reduce Util.max numbers`. `List.reduce` expect a curried two argument
function, not a tupled two argument function!

The funny thing is: It works differently with an Array. Actually you can write
stuff like this:

```fsharp
let array = [|20;14;37;16|]

Util.max array               // 37
Util.max [|20;14;37;16|]     // 37
Util.max (Array.ofList list) // 37
```

So you can pass arrays, and arrays are not considered as passing one argument.
This also works with functions with fixed arguments.

```fsharp
type Util() =
    static member replicate(amount, [<System.ParamArray>] xs) = Array.collect (Array.replicate amount) xs
```

The idea is that you can pass a variable amount of elements, and the first argument
describes how often every element gets repeated.

```fsharp
Util.replicate(3,1,2,3)      // [|1; 1; 1; 2; 2; 2; 3; 3; 3|]
Util.replicate(3, [|1;2;3|]) // [|1; 1; 1; 2; 2; 2; 3; 3; 3|]
Util.replicate(3, [1;2;3])   // [|[1; 2; 3]; [1; 2; 3]; [1; 2; 3]|]
```

So the first and second function calls are the same, but the third one is different.
It works with arrays but not with lists. By the way here we see how `ParamArray`
could be implemented in F# from the beginning and still maintain consistency by
always expecting an `Array` and disallowing the first notation.

The Last reason why it might be a bad idea is because in F# everything is really
build around the concepts of currying. A tupled syntax like in `Util.replicate`
means we always must pass all arguments. We cannot just partial apply
only the first argument and write:

```fsharp
Util.replicate(3)
```

The reason why we want to write this is to allow a more sequence based approach.
As an example. We would want to write it like this:

```fsharp
// This doesn't work
[|1;2;3|]
|> Util.replicate 3
```

Okay, in this small example you gain not much from this kind of piping-style.
But you usually want functions that work well with piping. It's important to notice
that piping doesn't work because we have the `|>` operator. Piping really works
because we have curried function that we can call without passing all arguments!

Currying is the reason we can choose to write `func a b` or `b |> func a`.
With tupled-syntax we really lose this advantage, and we also must write
more parenthesis and commas.

# The Alternative

So, instead of variable arguments, what should we do instead? We should just
expect a collection as an argument! If you expect `Seq` then a user also can
pass an `Array` or `List` as an argument.

```fsharp
let max xs              = Seq.reduce max xs
let replicate amount xs = Seq.collect (Seq.replicate amount) xs

let numbers = [1;2;3]

max numbers        // 3
max [1;2;3]        // 3
max [|1;2;3|]      // 3
max (seq {1 .. 3}) // 3

replicate 3 numbers        // seq [1;1;1;2;2;2;3;3;3]
replicate 3 [1;2;3]        // seq [1;1;1;2;2;2;3;3;3]
replicate 3 [|1;2;3|]      // seq [1;1;1;2;2;2;3;3;3]
replicate 3 (seq {1 .. 3}) // seq [1;1;1;2;2;2;3;3;3]
```

# Summary

Instead of variable arguments you should just expect a `Seq` as an argument
to a function. It's very easy to create `List` or `Array` in F# and you can
directly inline those creations with a function call. You don't get problems
if you already have a collection. With currying you can use functions in a
piping-style and you can use partial application.

But more important, its easy to see that arguments are variable because you
pass a `List`, `Array` or `Seq` as an argument. My recommendation is just
simply: <strong>Avoid functions with variable arguments.</strong>
