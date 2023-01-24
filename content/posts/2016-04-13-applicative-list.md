---
layout: post
title: "Applicative: Lists"
slug: applicative-list
date: 2016-04-13
tags: [FSharp,applicative,list]
description: Building an Applicative Functor for the list type
keywords: f#, fsharp, list, applicative, functor, functional, programming
---

In [Applicative Functors]({{< ref 2016-03-31-applicative-functors >}}) I primarily
used the `Option` type to show how you implement and use an *Applicative Functor*.
But the concept also works for any other type. This time I want to show
you the idea of an *Applicative* with a list, what it means, what you can do
with it and how `apply` works.

# Implementing `apply`

Currently the `List` module don't offer a `apply` function. So we must write it on our own.
As we learned in [Understanding bind]({{< ref 2016-04-03-understanding-bind >}}) we
could implement `apply` with `bind`. Because `List.collect` is the `bind` function (you
can see that by inspecting the function-signature), we could implement `apply` like this.

```fsharp
let apply lf lx =
    lf |> List.collect (fun f ->
    lx |> List.collect (fun x ->
        [f x]
    ))
```

Although it is good to know this, this time we implement `apply` from scratch. So we can
better understand how `apply` works.

The general idea of `apply` is easy. We need to implement a function that expects a
function as it's first argument, and a value as the second argument. But both arguments
are *boxed* in our type. The only thing we must do is somehow call our function with
our value. So we need a function that can handle the following function signature:

    list<('a -> 'b)> -> list<'a> -> list<'b>

If it is unclear why we get a `list<'b>` as a result. We should remember
what `apply` does as a single argument function. It just takes a `A<('a -> 'b)>` and
transform it into a new function `A<'a> -> A<'b>`. Here `A` stands for any *Applicative*
type.

For a list it has the following meaning:

1. We get a list of functions as the first value
1. We get a list of values as the second argument
1. *Unboxing* a list means we just loop over the list
1. Then we just execute every function with every value

We can implement `apply` like this:

```fsharp
let apply lf lx = [
    for f in lf do
    for x in lx do
        yield f x
]

let (<*>) = apply
```

# Working with `apply`

We keep it easy, so we just create two to four arguments functions that just adds its
inputs together.

```fsharp
let add2 x y     = x + y
let add3 x y z   = x + y + z
let add4 x y z w = x + y + z + w
```

Usually we need a `return` function, but we can easily lift any values into a list by just
surrounding it with `[]`, so we will skip this one. The idea of `apply` means every argument
of a function can now be a boxed type. That means, instead of just passing two `int`
to `add2` we can now pass a `list<int>` as the first argument and a `list<int>`
as the second argument, and so on. We now can write something like this.

```fsharp
[add2] <*> [1;2;3] <*> [10;20]
[add3] <*> [1;2;3] <*> [10;20] <*> [5]
[add4] <*> [1;2;3] <*> [10;20] <*> [5] <*> [100;200]
```

Let's see what those function calls produces

    [11; 21; 12; 22; 13; 23]
    [16; 26; 17; 27; 18; 28]
    [116; 216; 126; 226; 117; 217; 127; 227; 118; 218; 128; 228]

What we get back is the result of every input combination. Our first call with `add2`
expands to:

```fsharp
[
    add2 1 10
    add2 1 20
    add2 2 10
    add2 2 20
    add2 3 10
    add2 3 20
]
```

# How `apply` works

At this point it is interesting to see how `apply` actually works to get a better understanding
why we get those results. First we should remember how the operator `<*>` works. Our
apply operator is just a infix function. It uses the the thing on the left-side as the
first argument, and the thing on the right-side as the second argument. Instead of

```fsharp
[f] <*> [1;2;3]
```

we also could write

```fsharp
apply [f] [1;2;3]
```

When we have a term like `[add2] <*> [1;2;3] <*> [10;20]` it means, first `[add2] <*> [1;2;3]`
is executed and it will return a result! This is exactly how a normal function call work.
Even a normal function call like `add2 1 10` basically works by first executing `add2 1`
returning a new function and then pass `10` to it. That's why we also can write. `(add2 1) 10`
and it produces the same result. With `apply` or `<*>` it is the same, our term is
basically interpreted as

```fsharp
( [add2] <*> [1;2;3] )     <*> [10;20]
```

At first, our `apply` function is called with `[add2]` and `[1;2;3]` as its arguments.
Our `apply` function just just loops over the functions and the values and call every
function with a value. After `[add2] <*> [1;2;3]` we get a new list back, containing:

```fsharp
[
    add2 1
    add2 2
    add2 3
]
```

At this point it probably becomes clear why we can view `apply` as some kind of
*Partial Application* for *boxed* functions. The only thing that `apply` does is take
a *boxed* function and a *boxed* value and execute it. But it only does it for the
next argument. The first `apply` call returns a new list with three *Partial Applied*
functions. We get:

```fsharp
[add2 1; add2 2; add2 3] <*> [10;20]
```

In other words, the new list is used as the first argument to the next `apply` call.
This time we have a list of functions that contains three functions and two values.
Once again we loop over the functions and call every function with every value. We get:

```fsharp
[
    add2 1 10
    add2 1 20
    add2 2 10
    add2 2 20
    add2 3 10
    add2 3 20
]
```

And this will result in

```fsharp
[11; 21; 12; 22; 13; 23]
```

To get a hang of it, let's once again go through the `add4` example and visualize
every step. We start with

```fsharp
[add4] <*> [1;2;3] <*> [10;20] <*> [5] <*> [100;200]
```

The first `apply` call produces:

```fsharp
[
    add4 1
    add4 2
    add4 3
]
```

We then use this result with `apply` and use `[10;20]` next.

```fsharp
[
    add4 1 10
    add4 1 20
    add4 2 10
    add4 2 20
    add4 3 10
    add4 3 20
]
```

Then we use this list of functions with `[5]` and we get

```fsharp
[
    add4 1 10 5
    add4 1 20 5
    add4 2 10 5
    add4 2 20 5
    add4 3 10 5
    add4 3 20 5
]
```

Finally we use `[100;200]` on this list, and we get.

```fsharp
[
    add4 1 10 5 100
    add4 1 10 5 200
    add4 1 20 5 100
    add4 1 20 5 200
    add4 2 10 5 100
    add4 2 10 5 200
    add4 2 20 5 100
    add4 2 20 5 200
    add4 3 10 5 100
    add4 3 10 5 200
    add4 3 20 5 100
    add4 3 20 5 200
]
```

The last call executes the functions, so we get the result.

```fsharp
[116; 216; 126; 226; 117; 217; 127; 227; 118; 218; 128; 228]
```

# Using `apply`

In general what we can do with an *Applicative* for a list is that we can get the result
of all possible input combinations for a function, no matter how many arguments that
function has.

We also can easily create [Cartesian Products](https://en.wikipedia.org/wiki/Cartesian_product)
for a set of data. For example we could create all possible Playing cards in a game this way.

```fsharp
type Suit =
    | Club | Diamond | Heart | Spade

type Rank =
    | Ace | Two | Three | Four | Five | Six | Seven | Eight | Nine | Ten
    | Jack | Queen | King

type Card = Card of Suit * Rank
```

We now can generate all possible Cards by using.

```fsharp
let suits = [Club;Diamond;Heart;Spade]
let ranks = [Ace;Two;Three;Four;Five;Six;Seven;Eight;Nine;Ten;Jack;Queen;King]

let cards = [fun s r -> Card(s,r)] <*> suits <*> ranks
```

We now get a list of all 52 cards as a result.

```fsharp
let cards = [
    Card (Club,Ace); Card (Club,Two); Card (Club,Three); Card (Club,Four);
    Card (Club,Five); Card (Club,Six); Card (Club,Seven); Card (Club,Eight);
    Card (Club,Nine); Card (Club,Ten); Card (Club,Jack); Card (Club,Queen);
    Card (Club,King); Card (Diamond,Ace); Card (Diamond,Two);
    Card (Diamond,Three); Card (Diamond,Four); Card (Diamond,Five);
    Card (Diamond,Six); Card (Diamond,Seven); Card (Diamond,Eight);
    Card (Diamond,Nine); Card (Diamond,Ten); Card (Diamond,Jack);
    Card (Diamond,Queen); Card (Diamond,King); Card (Heart,Ace);
    Card (Heart,Two); Card (Heart,Three); Card (Heart,Four);
    Card (Heart,Five); Card (Heart,Six); Card (Heart,Seven);
    Card (Heart,Eight); Card (Heart,Nine); Card (Heart,Ten);
    Card (Heart,Jack); Card (Heart,Queen); Card (Heart,King);
    Card (Spade,Ace); Card (Spade,Two); Card (Spade,Three);
    Card (Spade,Four); Card (Spade,Five); Card (Spade,Six);
    Card (Spade,Seven); Card (Spade,Eight); Card (Spade,Nine);
    Card (Spade,Ten); Card (Spade,Jack); Card (Spade,Queen);
    Card (Spade,King)
]
```

The *Cartesian Product* is also the idea how we view relational data. We could for example
create two lists that refers to each other, with `apply` we then can easily create the
*Cartesian Product* and filter those data.

```fsharp
type Person = {
    Id:   int
    Name: string
} with
    static member create id name = {Id=id; Name=name}

type Like = {
    PersonId: int
    Name:     string
} with
    static member create pid name = {PersonId=pid; Name=name}

let persons = [
    Person.create 1 "David"
    Person.create 2 "Markus"
    Person.create 3 "Björn"
]

let likes = [
    Like.create 1 "Pizza"
    Like.create 2 "Pizza"
    Like.create 3 "Pizza"
    Like.create 3 "Coffee"
    Like.create 1 "Tea"
    Like.create 2 "Tea"
]
```

We now can create the *Cartesian Product* of those Data. And afterwards filter it.

```fsharp
let likesTea =
    [fun p l -> p,l] <*> persons <*> likes
    |> List.filter (fun (person,like) -> person.Id = like.PersonId)
    |> List.filter (fun (person,like) -> like.Name = "Tea")
    |> List.map    (fun (person,like) -> person.Name)
```

This will return: ` ["David"; "Markus"]`

and resembles a SQL-Statement like:

```sql
SELECT p.Name
FROM   person p, likes l
WHERE  p.Id = l.PersonId
AND    l.Name = "Tea"
```

Sure, most stuff is basically *List-Processing* at this point, but `apply` is just another
functions in a tool-set that opens up some possibilities. And at this point you probably
even see the connection between functional list processing and SQL, or in general the
C# LINQ Feature.
