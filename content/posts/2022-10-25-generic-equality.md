---
layout: post
title: "Optional Generic Equality on a Data-Type in F#"
slug: optional-generic-equality
date: 2022-10-25
tags: [FSharp,equality,generics]
description: "Implementing an optional Generic Equality"
keywords: f#, fsharp, equality, generic
---

Lately I implemented my own immutable Queue and came upon a problem
implementing equality for it. The problem goes like this. You want
a generic data-type that supports equality. But it only should support
equality if the generic type also supports equality.

Seems a little bit weird? But F# has this built-in probably without that
you realize this. At least i did not.

Here is an example with a list.

You can write the following in F#.

```fsharp
[1;2;3] = [1;2;3]
```

In F# you can compare two lists. The above statement will return `true` as the list itself
is equal. But equality depends on he types inside the list. Here it is `int`.

When you use F# this doesn't look surprising. But you probably also know that list can also
contain types that has no equality. This would be for example a function.

```fsharp
let double x = x * 2
[id;double] = [id;double]
```

When you write code like this, you get the compiler-error

    The type '('a -> 'a)' does not support the 'equality' constraint because it is a function type

Or in other words equality for a list depends on the generic type you use. Let's try to implement
our own `List` to see the problem more clear and how we solve this.

## Our own List

We start with

```fsharp
type MyList<'a> =
| Empty
| Cons  of 'a * MyList<'a>

let empty     = Empty
let cons x xs = Cons(x,xs)
```

implementing Equality is easy. As a function we could just write.

```fsharp
let rec equal xs ys =
    match xs,ys with
    | Empty,Empty           -> true
    | Empty,ys              -> false
    | xs,Empty              -> false
    | Cons(x,xs),Cons(y,ys) -> x = y && equal xs ys
```

And our own List type work as expected. Here is a small test.

```fsharp
let rec map f xs =
    match xs with
    | Empty      -> Empty
    | Cons(x,xs) -> (cons (f x) (map f xs))

let xs = (cons 1 (cons 2 (cons 3 empty)))
let ys = (cons 2 (cons 4 (cons 6 empty)))

if (equal (map (fun x -> x * 2) xs) (cons 2 (cons 4 (cons 6 empty))))
then printfn "Equal"
else printfn "Not Equal"
```

The above code will print `Equal` as expected. We also can create a
list that contains functions without a problem.

```fsharp
let double x = x * 2
let fs = (cons id (cons double empty))
```

but comparing them doesn't work

```fsharp
// The type '(int -> int)' does not support the 'equality' constraint because it is a function type
equal fs fs
```

But we now have a function named `equal` or in your code you probably have a module `MyList.equal`. But
you also want to overload equality. So we can use `=` for our own type instead of writing `equal`. And then
things start to get complicated.

## Overloading Equality

The first attempt would be

```fsharp {hl_lines=[2,6,14]}
[<CustomEquality;NoComparison>]
type MyList<'a> =
    | Empty
    | Cons  of 'a * MyList<'a>

    override this.Equals (obj:obj) : bool =
        match obj with
        | :? MyList<'a> as other ->
            let rec loop (xs:MyList<'a>) (ys:MyList<'a>) =
                match xs,ys with
                | Empty,Empty           -> true
                | Empty,ys              -> false
                | xs,Empty              -> false
                | Cons(x,xs),Cons(y,ys) -> x = y && loop xs ys
            loop this other
        | _ ->
            false

    override this.GetHashCode() : int =
        Unchecked.hash this
```

But the F# compiler will give the following errors for the highlighted lines.

    1    -> The signature and implementation are not compatible because the declaration
            of the type parameter 'a' requires a constraint of the form 'a: equality

    6,14 -> A type parameter is missing a constraint 'when 'a: equality'

You could fix this by adding the constraint.

```fsharp {hl_lines=[2]}
[<CustomEquality;NoComparison>]
type MyList<'a when 'a : equality> =
    | Empty
    | Cons  of 'a * MyList<'a>

    override this.Equals (obj:obj) : bool =
        match obj with
        | :? MyList<'a> as other ->
            let rec loop (xs:MyList<'a>) (ys:MyList<'a>) =
                match xs,ys with
                | Empty,Empty           -> true
                | Empty,ys              -> false
                | xs,Empty              -> false
                | Cons(x,xs),Cons(y,ys) -> x = y && loop xs ys
            loop this other
        | _ ->
            false

    override this.GetHashCode() : int =
        Unchecked.hash this
```

But then you lose the ability to use `MyList` with a generic types that has no
equality. Because now all types must have equality. So you cannot write.

```fsharp
let fs = (cons id (cons double empty))
```

The above code now returns the error

    The type '('a -> 'a)' does not support the 'equality' constraint because it is a function type

So what we need is either two types one with the generic constraint and one without or the ability to
say that equality depends on the generic type we pass. If the generic type has equality then our
own List implementation can be compared otherwise not.

Luckily F# already supports this.

## EqualityConditionalOn

F# has the attributes `EqualityConditionalOn` and there is also a `ComparisonConditionalOn`
that works the same way. But only adding those will still not solve the problem.

```fsharp {hl_lines=[2,6,14]}
[<CustomEquality;NoComparison>]
type MyList<[<EqualityConditionalOn>]'a> =
    | Empty
    | Cons  of 'a * MyList<'a>

    override this.Equals (obj:obj) : bool =
        match obj with
        | :? MyList<'a> as other ->
            let rec loop (xs:MyList<'a>) (ys:MyList<'a>) =
                match xs,ys with
                | Empty,Empty           -> true
                | Empty,ys              -> false
                | xs,Empty              -> false
                | Cons(x,xs),Cons(y,ys) -> x = y && loop xs ys
            loop this other
        | _ ->
            false

    override this.GetHashCode() : int =
        Unchecked.hash this
```

you still get the same errors. So what do we missing here?

The answer is that we need to replace the `=` with a **non-generic** version as
using `=` always expects the equal constraint. So the last piece we need to change is
use `Unchecked.equals` (or `Unchecked.compare` when you implement comparison).

```fsharp {hl_lines=[14]}
[<CustomEquality;NoComparison>]
type MyList<[<EqualityConditionalOn>]'a> =
    | Empty
    | Cons  of 'a * MyList<'a>

    override this.Equals (obj:obj) : bool =
        match obj with
        | :? MyList<'a> as other ->
            let rec loop (xs:MyList<'a>) (ys:MyList<'a>) =
                match xs,ys with
                | Empty,Empty           -> true
                | Empty,ys              -> false
                | xs,Empty              -> false
                | Cons(x,xs),Cons(y,ys) -> Unchecked.equals x y && loop xs ys
            loop this other
        | _ ->
            false

    override this.GetHashCode() : int =
        Unchecked.hash this
```

Now you have a list that can contain any generic type also those without equality and
on those with equality you also can use `=`.


```fsharp
if (map (fun x -> x * 2) xs) = ys
then printfn "Equal"
else printfn "Not Equal"

let double x = x * 2
let fs = (cons id (cons double empty))
```

## Summary

Sometimes it is interesting how much pain a *simple* (whatever...) feature like **overloading**
can cause.

If you want to see a full example with Comparision than you can
look at my [Immutable Queue](https://github.com/DavidRaab/Queue/blob/master/lib/Queue.fs) implementation.