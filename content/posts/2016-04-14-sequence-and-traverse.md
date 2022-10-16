---
layout: post
title: "Sequence and Traverse"
slug: sequence-and-traverse
date: 2016-04-14
tags: [FSharp,functor,applicative,list,option]
description: Explains the sequence and traverse functions
keywords: f#, fsharp, list, applicative, sequence, traverse, functional, programming
---

One problem that appears from time to time is that we we have some kind of
collection (I use `list` here) and we want to `map` every element with a
*monadic function* `'a -> M<'b>`. This then returns a `list<M<'a>>`. But
often we want a `M<list<'a>>`.

To be more concrete. Let's assume we want to turn a list of strings into integers. We could
write a `tryParseInt` function that does `string -> option<int>`. But if we `map`
our function with a `list<string>` we get a `list<option<int>>` back.

Sometimes that is what we want, but very often it is not. Usually what we want is
a `option<list<int>>` instead. So we expect the types to be switched.

The idea behind it in this example is that when we use `map` over a list of strings it is
either successful as a whole and all elements are integers or as soon a single element
is not an integer we get `None` back as a whole.

We could just write a function that somehow does that kind of transformation just for
`tryParseInt`, but instead of doing that again and again for every function we try to
generalize that problem, so we can turn every `list<option<'a>>` into `option<list<'a>>`.
Not only that, we want to generalize the problem that it works for any type, not
just `option`.

This generalization is what we think of the `sequence` and `traverse` functions.

## Sequence

We first start with the *monadic* `tryParseInt` function we already mentioned.

```fsharp
let tryParseInt str =
    match System.Int32.TryParse(str) with
    | false,_ -> None
    | true,x  -> Some x
```

Next, we have some kind of input from a file, user or somewhere else.

```fsharp
let validInput   = ["1";"100";"12";"5789"]
let invalidInput = ["1";"100";"12";"foo"]
```

In our example we want to do the following:

1. Parse every `string` to an `int`
1. If all inputs are valid, we want to `sum` the results
1. If one input is invalid, we want to print an error message

The first step is easy, as we could just `map` the list.

```fsharp
let validInts = List.map tryParseInt validInput
```

We now have a list containing:

```fsharp
[Some 1; Some 100; Some 12; Some 5789]
```

The problem starts in how we determine that every element is valid. We sure could
use `fold` to loop through our list. Starting with a `bool` set to `true` and as soon
we encounter a `None` we set the `bool` to `false`.

But we already have `Option` for this kind of purpose. With `Option` we still can
return the idea of `true` (Some) and `false` (None), but additional we also can return
a value.

Instead of just getting a boolean flag like `true` and `false` we just return `Some`
with a new list that has all the `option` values stripped instead. For the valid
input case we just expect:

```fsharp
Some [1; 100; 12; 5789]
```

for the invalid input list we just expect:

```fsharp
None
```

As we need to loop through every element and build a new list, this is just a task for
`List.foldBack`.

1. Because we want a `option<list<'a>>` as a result, we start with `Some []`
1. Then we check if `acc` and `x` are both `Some`
1. If that is the case we add `x` to `acc`
1. Otherwise we return `None`

We name this operation `sequence`.

```fsharp
let sequenceFold listOfOptions =
    let folder x acc =
        match x,acc with
        | Some x, Some list -> Some (x :: list)
        | Some _, None _    -> None
        | None _, Some _    -> None
        | None _, None _    -> None
    List.foldBack folder listOfOptions (Some [])
```

When we test our function it returns the right result

```fsharp
List.map tryParseInt validInput |> sequenceFold
// Some [1; 100; 12; 5789]

List.map tryParseInt invalidInput |> sequenceFold
// None
```

Nice, it works! But this implementation still has a problem. Our `folder` function
is basically a duplicate of the `apply` function! The whole idea to check two
`option` and only execute some code if both are `Some` is exactly what `apply` does.
Let's look again at `apply`.

```fsharp
let apply fo xo =
    match fo,xo with
    | Some f, Some x -> Some (f x)
    | Some _, None _ -> None
    | None _, Some _ -> None
    | None _, None _ -> None

let (<*>) = apply
```

There is also another problem here. It doesn't matter which type we use. We always have
to lift an empty list. In this case we did `Some []` for the accumulator. But in a
`Async` case we just want an empty list inside an `Async`. So we always just want to
`return` a list for the context. It just means: We always can create a `sequence`
function as long our type provides a `return` and `apply` function. Or in other words,
our type is an [Applicative Functor]({{< ref 2016-03-31-applicative-functors >}}).

Let's think about how we can implement `sequence` with `return` and `apply`.

1. The code we executed when we had two `Some` values was `x :: list`
1. So we just create a function for this operation
1. And `apply` this function
1. Then we use this function as our `folder` function

```fsharp
let retn x = Some x

let sequence listOfOptions =
    let folder x xs = retn (fun x xs -> x :: xs) <*> x <*> xs
    List.foldBack folder listOfOptions (retn [])
```

We still get our expected results

```fsharp
List.map tryParseInt validInput |> sequence
// Some [1; 100; 12; 5789]

List.map tryParseInt invalidInput |> sequence
// None
```

Nice, everything works. So why is rewriting `sequence` in such a way better?
What we basically have here is a *Design Pattern* (or what every Design Pattern is --
A Copy & Paste Pattern).

The `sequence` operation for a `list` is *always* the same. It just depends solely that
a type supports a `retn` and a `apply` function. It probably opens up the question
that when the implementation is always the same if we cannot just have a single
implementation?

Yes and no. Currently in F# `retn` and `<*>` are not Polymorphic, they are specific functions
we define ourself. [We could fix it with Higher-Kinded Polymorphism]({{< ref 2016-03-24-higher-kinded-polymorphism >}})
but F# don't support this nicely, but there are ways around it. But i will not cover
this topic here.

## Traverse

So far we discussed `sequence` but in practice you will less likely implement `sequence`
at all. Instead we will implement `traverse`. So how is `traverse` different from `sequence`?

As you have seen so far. Even with `sequence` there is one *pattern* that is always the same.
You first `map` a list, then you use `sequence` on it. `traverse` is just the idea to combine
both operations into a single operation.

If that sounds complicated, it isn't at all! Just think for a moment. `map` just means we apply
a function to every element before we use `sequence`. So the only thing we need to
implement `traverse` is to make sure we call a function that transforms every element
before we pass it to our *lifted* function.

```fsharp
let traverse f list =
    let folder x xs = retn (fun x xs -> x :: xs) <*> f x <*> xs
    List.foldBack folder list (retn [])
```

The difference is so *minimal* that it can even be overlooked easily. We added the function
call between the `<*>` operators: `... <*> f x <*> xs`. Instead of `map` and then calling
`sequence`, we now just can use `traverse` instead.

```fsharp
traverse tryParseInt validInput
// Some [1; 100; 12; 5789]

traverse tryParseInt invalidInput
// None
```

If the *logic* seems still hard to follow. We just can think of `traverse` as a `map` function
for *monadic functions* that additionally *swaps* the layer when it finishes. When we use

```fsharp
List.map tryParseInt xs
```

We get a `list<option<'b>>`. But when we use

```fsharp
traverse tryParseInt xs
```

we get a `option<list<'b>>`.

## Sequence defined through traverse

The primary reason why you less likely implement `sequence` is because `traverse` is
basically the same implementation. You can enhance a `sequence` implementation easily
by just adding a function call to the element before you `apply` it.

Once you have a `traverse` function, you can very easily create `sequence` by just
using the `id` function with `traverse`.

```fsharp
let sequence xs = traverse id xs
```

You could even come to the conclusion to not provide a `sequence` implementation at all.

## Finishing the Example

With `traverse` we now can easily finish our example that we started.

```fsharp
let sum input =
    input
    |> traverse tryParseInt
    |> Option.map List.sum

let printSum opt =
    match opt with
    | None     -> printfn "Error: Some inputs were not numbers!"
    | Some sum -> printfn "Sum: %d" sum

printSum (sum validInput)
printSum (sum invalidInput)
```

This code now produces:

    Sum: 5902
    Error: Some inputs were not numbers!

## Not limited to Option

It is in general important to understand that this concept is not limited to `list`
and `option`. The only thing we need is a data-structure that has a `fold` function,
and another type that is an *Applicative Functor* (has `return` and a `apply` function).

After we have those the general idea is to swap both layers.
For example when we have a *monadic function* `download` that has the signature
`Uri -> Async<string>` we expect that we can use this function on a `list<Uri>`.
With `List.map` we would get a

    list<Async<string>>

But when we use `traverse` we get a

    Async<list<string>>

We don't need to write such a function for the `Async` type as we can use `Async.Parallel`.
`Async.Parallel` is basically the `sequence` function. It takes a `seq<Async<'a>>` and turns
it into an Async containing an array `Async<'b []>`. But anyway to see how it works we could
extend the `Async` module with the needed functions.

```fsharp
module Async =
    let retn x = async { return x }

    let apply af ax = async {
        // We start both async task in Parallel
        let! pf = Async.StartChild af
        let! px = Async.StartChild ax
        // We then wait that both async operations complete
        let! f = pf
        let! x = px
        // Finally we execute (f x)
        return f x
    }

    let (<*>)   = apply
    let map f x = retn f <*> x

    let traverse f list =
        let folder x xs = retn (fun x xs -> x :: xs) <*> f x <*> xs
        List.foldBack folder list (retn [])

(** We now can use `Async.traverse` in the following way *)

let uris = [Uri("http://www.google.com"); Uri("https://fsharpforfunandprofit.com")]
let download uri =
    use wc = new System.Net.WebClient()
    wc.AsyncDownloadString(uri)

let sizes =
    uris
    |> Async.traverse download
    |> Async.map (List.map (fun str -> str.Length))
    |> Async.RunSynchronously

for size in sizes do
    printfn "Content Length: %d" size
```

With `Async.traverse` we get a single Async that only completes once all Uris are downloaded.
As you can see from the implementation. `Async.traverse` is identical to the version that we
wrote for the `option` type. We just have to set `retn` and `<*>` to functions that work
with `Async`.

## Further Reading

 * [Understanding traverse and sequence](http://fsharpforfunandprofit.com/posts/elevated-world-4/)
 * [[Haskell] Traversable](https://en.wikibooks.org/wiki/Haskell/Traversable)
