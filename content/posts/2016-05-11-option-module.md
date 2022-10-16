---
layout: post
title: "The Option Module"
slug: option-module
date: 2016-05-11
tags: [FSharp,option]
description: Explains some lesser known functions in the Option Module
keywords: f#, fsharp, functional, programming, option
---

The Option type is a well known and often used type, but at least for me, most of the
time I just used `Option.map` and `Option.bind` and ignored functions like `Option.exists`,
`Option.filter`, `Option.fold` and so on. I spent some time with those functions to understand
when those are useful.

## defaultArg

The first function i look at is actually not in the Option module. It is the `defaultArg`
function. With `defaultArg` we can extract an option type and provide a default value
in the case we had no value.

```fsharp
let o1 = defaultArg (Some 10) 0 // 10
let o2 = defaultArg (None) 0    // 0
```

One think I dislike is the order of the arguments. Because the Option type is expected first
as an argument, `defaultArg` is unsuitable for piping or composition. That's why I most often
add a `orElse` function to the Option module myself.

```fsharp
module Option =
    let orElse x o = defaultArg o x

Some 10 |> Option.orElse 10 // 10
None    |> Option.orElse 0  // 0
```

## exists & forall

I must admit, i never looked closer at those functions. It is obvious that those functions are
*ported* from the List/Array/Seq module. But because an option never contains more than one element,
I never looked closer to those functions. The truth is, because we know that an option only contains
either no value or a single value, the meaning of those functions just change.

Let's look at some typical code with no option at all that you will sometimes have. You just
check a variable if some statement is true or false and you use that for branching.

```fsharp
let input = 5

if   input < 10
then printfn "input smaller 10"
else printfn "Input must be smaller than 10"

// prints: input smaller 10
```

What do you do, when x is an `option`? Then you can use `Option.exists`

```fsharp
let input = Some 5

if   input |> Option.exists (fun x -> x < 10)
then printfn "input smaller 10"
else printfn "Input must be smaller than 10"

// prints: input smaller 10
```

Generally speaking. With `Option.exists` you can check an option for a condition. `None`
is treated as `false`. Naming the function `check` or some other name than
`exists` would probably have been a better name. With some helper functions we can enhance
the validation process.

```fsharp
let smaller min x = x <= min
let greater max x = x >= max

let input = Some 5

if   input |> Option.exists (greater 0) && input |> Option.exists (smaller 10)
then printfn "input between 0 and 10"
else printfn "input not valid"

// prints: input between 0 and 10
```

`Option.forall` is basically the same, only that `None` is threaten as `true` instead of `false`.
But I must admit, I cannot come up with a useful example for `forall`.

## filter

In my last example I added a second check. While two checks are still somehow okay in terms of
readability it can become unhandy fast. Wouldn't it be better if we could chain the operations?

`filter` gives us exactly this ability. Instead of returning `true` or `false` it just returns
an option again. When the predicate we provided returns `true` we just get back the original value
unchanged. Otherwise we get `None`.

```fsharp
let isValid x = x |> Option.exists (fun _ -> true)
let isEven x  = x % 2 = 0

let input =
    Some 6
    |> Option.filter (greater 0)
    |> Option.filter (smaller 10)
    |> Option.filter isEven

if   input |> isValid
then printfn "input between 0 and 10 and even"
else printfn "invalid input"

// prints: input between 0 and 10 and even
```

We also can use `Option.filter` to easily turn a type into an option based on a predicate.

```fsharp
Some 1 |> Option.filter isEven // None
Some 2 |> Option.filter isEven // Some 2
```

## fold

I started with `defaultArg` and implemented `orElse`. But overall we could replace those with
`fold`. Besides the option itself, `fold` expects two additional arguments. A function and an
accumulator. `fold` either executes the function or it returns the accumulator as the default value.

```fsharp
let orElse def o = Option.fold (fun _ x -> x) def o
```

In general the idea of `fold` is that we can return any other type that we want. `fold` is
a general way to convert types. If that sounds a lot like `map`. The difference is that `map`
still only converts the wrapped type and we still get an option back. But with `fold` we
directly get the wrapped type back. It just means that whenever you use a `map` and then
`orElse`. You also could use `fold` instead.

```fsharp
let square x = x * x

Some 10 |> Option.map square |> Option.orElse 0 // 100
Some 10 |> Option.fold (fun _ x -> square x) 0  // 100
```

Up to this point I always ignored the accumulator argument, and just used the accumulator
as the default argument. But in general it means whenever you want to use a function
where one argument is an option you could probably use `fold`. In general `fold` works
nicely together with binary functions.

```fsharp
let swap f x y = f y x

Option.fold min 0 (Some 100) // 0
Option.fold max 0 (Some 100) // 100
Option.fold min 0 (None)     // 0
Option.fold max 0 (None)     // 0
Option.fold (+) 0 (Some 100) // 100
Option.fold (+) 0 (None)     // 0
Option.fold (swap String.replicate) "x" (Some 5) // "xxxxx"
Option.fold (swap String.replicate) "x" (None)   // "x"
Option.fold List.append [1;2;3] (Some [4;5;6])   // [1;2;3;4;5;6]
Option.fold List.append [1;2;3] (None)           // [1;2;3]
```

We either execute our function with two arguments, or if the second argument is a `None` we return
the first argument as the default value. The arguments itself don't need to be of the same types.

## Validation

With very few helper functions we could built a small validation framework that uses the option type.
Most of them are just other names instead of `map`, `filter` or `bind`.

```fsharp
// Monadic functions -- Converter
let toInt str =
    match Int32.TryParse str with
    | false,_ -> None
    | true,x  -> Some x

// Helper Functions
let orReturn x  = Option.fold (fun _ x -> x) x
let whenValid   = Option.map
let is          = Option.filter
let convert     = Option.bind
let combine f g = fun x -> Some x |> is f |> Option.exists g

// Validation Functions
let smaller min x   = x <= min
let greater max x   = x >= max
let between min max = (combine (greater min) (smaller max))
let even x          = x % 2 = 0

// Mapping functions
let square x = x * x

// Usage
let transformInput input =
    input
    |> convert toInt
    |> is (between 0 100)
    |> is even
    |> whenValid square
    |> orReturn 0

transformInput (Some "foo") // 0   -- not valid int
transformInput (Some "2")   // 4
transformInput (Some "5")   // 0   -- not even
transformInput (Some "10")  // 100
transformInput (Some "102") // 0   -- greater 100
```