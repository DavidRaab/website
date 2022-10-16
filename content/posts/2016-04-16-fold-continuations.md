---
layout: post
title: "Continuations and foldBack"
slug: continuations-and-foldback
date: 2016-04-16
tags: [FSharp,fold,list,cps,recursion]
description: This article shows how you can implement List.foldBack with a continuation function
keywords: f#, fsharp, fold, continuation, foldback, fold, list
---

In [From mutable loops to immutable folds]({{< ref 2016-04-05-mutable-loops-to-immutability >}})
we implemented `foldBack` through `rev` and `fold`. In this post I show you how you can implement
`foldBack` by using a *continuation function*.

## Functions

Before we see how to implement `foldBack`, I want to give you analogy first. This analogy helped
me in a lot of cases. I hope that this analogy will also help you in better understanding
the further post, or probably even in other areas in programming in general.

### Back to the Future

One idea how you can view functions is, that you can do stuff now, with things you not
have yet, but you know you will get them in the future. Sounds weird? Let's go over an
example that you probably already know. Function composition.

Usually most definitions of a function that does function composition have three arguments.
But let's write it with only two arguments. You are given the task to write a function
that has two functions as the input, and now you should compose them. How do you do that?

Usually you start with something like

```fsharp
let compose f g =
    ...
```

But what comes next? Your task is to execute `f` and pass the result to `g`. But how do you execute
`f` if you don't a value to execute `f`? The answer is, you create a function inside `compose`. This
function gets the value to execute `f` sometime in the future.

```fsharp
let compose f g =
    let innerFunction x =
        let result = f x
        g result
    ...
```

But what do you now do with `innerFunction`? We still don't have a value to execute `innerFunction`.
Yes, that's right, because of this, your only option is to return `innerFunction` from our `compose`
function.

```fsharp
let compose f g =
    let innerFunction x =
        let result = f x
        g result
    innerFunction
```

This is exactly what `>>` does. It is only the fully expanded version of it. If you want to shorten it.
As you create a inner function and just return it. You also can just create a lambda directly and return
it.

```fsharp
let compose f g = fun x ->
    g (f x)
```

And because we have *Currying* and all functions are anyway just functions that returns function,
you just can add `x` as a function argument.

```fsharp
let compose f g x =
    g (f x)
```

It seems that i only explained *function composition* but the analogy I explained fits more nicely
into a lot more use-cases. It doesn't mean you only can execute some kind of functions with a value.
In general it means you already can work with a value you don't have yet! For example, you also
could write a function that first does something with `x` and then pass the result to it to `f`.

In general the analogy means. Whenever you need to somehow work with a value you don't have yet.
You just create a function. So you already can work with the value as if you had it. And you
can do whatever you want with it. But then instead of returning a result you return a function
instead. So you expect that someone else in the future gives you a value that starts your execution.

The general idea is. You already can compose behaviour or functions together without directly
executing anything. You first compose everything, and you delay the execution as far as possible
to the future.

## Rethinking the problem

With that idea in-place. Let's look again at `foldBack` and if we somehow can rethink that
problem. Let's first focus on the behaviour of `foldBack` and how it should work *logically*.

```fsharp
let f x     = x * x
let squared = List.foldBack (fun x acc -> f x :: acc) [1;2;3;4] []
```

The general *logic* is that we traverse a list backwards. Instead of starting with the initial
accumulator and fetching `1` from the last, we start by fetching `4` from the end. Then `3`, `2`
and last but not least `1`. The general logic how `foldBack` works is like that.

     list      | acc        | x | (f x) :: acc
    -----------+------------+---+--------------
     [1;2;3;4] | []         | 4 | [16]
       [1;2;3] | [16]       | 3 | [9;16]
         [1;2] | [9;16]     | 2 | [4;9;16]
           [1] | [4;9;16]   | 1 | [1;4;9;16]
            [] | [1;4;9;16] | return -> [1;4;9;16]

At this point let's focus on our `f` function that we pass `foldBack`. The `f` function
get the argument `x` and `acc`. What does those values represents at any point in time exactly?

As we basically just loop over a data-structure with `foldBack` it just means `x` is just
one element from our list. The interesting thing is what `acc` contains. `acc` contains
the *accumulation* of everything right from our current element. To be more concrete, let's
say we reached element `2`.

Then `x` contains `2`, but `acc` at this point in time contains `[9;16]`. It contains the result
of our accumulation that was right of `2`. Let's assume we have.

```fsharp
List.foldBack (fun x acc -> x + acc) [1;2;3;4] 0
```

In that case when we reach `2` our `x` is once again `2` but our `acc` contains `7`, because so
far it calculated `0 + 4 + 3`.

The only problem we have. We cannot go backwards through a list. That was the whole reason why we
reversed the list before we looped/recurs through it. We only can go from left to right.
And that is bad. Because we have to start with `1` and we need to pass an `acc` to the `f`
function that we don't have at that point in time!

Uhm wait! Isn't that exactly what we discussed previously? What do we do if we expect to work
with a value that we don't have yet? We create a function!

## Functions as accumulator

The basic idea to solve our problem is by using a function that we use as our *accumulator*.
We use a function because we need to work with values we don't have yet. Because we have a
function I use `cont` (for Continuation) in our inner `loop` instead of `acc`. Let's
first write the basic template to loop through a list first.

```fsharp
let foldBack f list (acc:'State) =
    let rec loop cont list =
        match list with
        | []      -> ???
        | x::list ->
            let newCont = ???
            loop newCont list
    loop ??? list
```

So the question we have are.

1. What do we do for every element?
1. What do we do after we looped through our list?
1. With what kind of value do we start our loop?

### What do we do for every element?

At first, we reconsider what we are supposed to do. Our task is to execute the *folder*
function that the caller provided us as `f`. We pass this function the *current value* and the
*right-accumulator*. At this point in time we have the current value inside of `x`,
but we don't have the *right-accumulator*. What we now do, we just create a *inner function*
`newCont` that at some point in the future will get the *right-accumulator*, then we also can
call the `f` function.

```fsharp
let newCont racc =
    f x racc
```

But is that right? Let's think what would happen. In our first recursion we get `1` from
`[1;2;3;4]` and save it in `x`. We then create a function that basically does.

```fsharp
let newCont racc =
    f 1 racc
```

We pass that function as `cont` to the next iteration. In the next recursion we extract `2`
from the remaining list `[2;3;4]`, now we create a new `newCont` that basically does.

```fsharp
let newCont racc =
    f 2 racc
```

Then this new function is used as our `cont`. But this behaviour is wrong. We loop through
our list and on every step we just create a new function that will replace the old function.
The problem is, we cannot just throw the old `cont` away, we somehow have to use it. But how?

Let's look at our `f 2 racc` call. What does that function return when we call it? Or better yet,
which values do we have here exactly? Our `racc` is the *right-accumulator* that we have in the
future, in our example this will be `[9;16]`. When we call `f 2 racc` we actually call
`f 2 [9;16]`. What does our `f` function in our example? It squares `x` and add it to the `racc`.
So after we do `f 2 [9;16]` we will get `[4;9;16]` as a result.

But wait! Isn't that the *right-accumulator* that our previous `cont` needed? Yes, it is! And now
it becomes more obvious in how we need to use `cont`. We call `f x racc`, this then produces a
value that can be used to call our previous function that we passed as `cont`.

```fsharp
let newCont racc =
    let res = f x racc
    cont res
```

If that sound hard to follow. Just remember *function composition* again. What we have hear is
just this idea! In function composition we call a function `f` with a value we don't have yet, that
will produce a value that can be passed to `g`.

```fsharp
let compose f g = fun x ->
    let res = f x
    g res
```

Here we have the same. The only difference is that this time `cont` is a function we created
inside a function in a previous loop. Let's shorten the example a little bit. In function composition
we just ended with `let compose f g x = g (f x)`. We now shorten our call to:

```fsharp
let newCont racc = cont (f x racc)
```

Our `f` function takes two arguments, but otherwise it is the same idea. The whole *loop-body* part
in our example now looks like this:

```fsharp
| x::list ->
    let newCont racc = cont (f x racc)
    loop newCont list
```

But we are not done yet, we still have to fill out two missing parts.

### What do we do after we looped through the list?

One remaining part is the logic we have to do once we finished our looping. Let's actually try to
understand what we exactly build. In every step we create a new function
that expects the *right-accumulator* with this and the current `x` we calculate the
*right-accumulator* for the previous function. In our first loop we create.

```fsharp
let cont1 racc = initCont (f 1 racc)
```

In our second loop we basically do:

```fsharp
let cont2 racc = cont1 (f 2 racc)
```

Remember, we create a new function, but we call the previously `cont`. I used `cont1` to make
the relation more clear. Technically every new step now creates a new `cont` function that calls
the previous `cont` function. If we expand it fully we basically have.

```fsharp
let cont1 racc = initCont (f 1 racc)
let cont2 racc = cont1    (f 2 racc)
let cont3 racc = cont2    (f 3 racc)
let cont4 racc = cont3    (f 4 racc)
```

Every new function that we create contains the previous function as a closure. Actually let's
inline the whole functions step-by-step. First we inline `cont1` in our `cont2` function, and so on.

```fsharp
let cont2 racc = initCont (f 1 (f 2 racc))
```

then we inline `cont2` into `cont3`

```fsharp
let cont3 racc = initCont (f 1 (f 2 (f 3 racc)))
```

finally `cont3` into `cont4`

```fsharp
let cont4 racc = initCont (f 1 (f 2 (f 3 (f 4 racc))))
```

As we can see. With every loop iteration that we go forward, we build up a function. Every new
function we create builds the *right-accumulator* that is needed for the *previous-step*.
Once we looped through our whole data-structure, we ended up with a final function. The only thing
we need to do, is to start our function by passing it the final *right-accumulator*.

And at this point we have the final *right-accumulator*! At the end we have to pass the
*right-accumulator* of our last element. We only have the list `[1;2;3;4]` so what is the
*right-accumulator* of the last element `4`? The *initial-accumulator* that the user passed
to `foldBack`!

So here is our answer. Once we reached the end of our list we have a function and we need to pass it
the *initial-accumulator*. Our *empty-list case* now looks something like that:

```fsharp
| [] -> cont acc
```

### With what kind of value do we start our loop?

The last remaining part is how we start our loop. Or put in other words. What do we provide
as our `initCont` value? To understand this part better, let's assume we started our loop.
We already saw what kind of structure we build. In the end we just had

```fsharp
let cont4 racc = initCont (f 1 (f 2 (f 3 (f 4 racc))))
```

Once we reach the end of our data-structure in our example we provided the empty list `[]` as the
starting right accumulator. So we basically have.

```fsharp
let cont4 = initCont (f 1 (f 2 (f 3 (f 4 []))))
```

At this point, now our function starts its execution. First `f 4 []` is calculated and this will
return `[16]`

```fsharp
let cont4 = initCont (f 1 (f 2 (f 3 [16])))
```

Now `f 3 [16]` executes and we get

```fsharp
let cont4 = initCont (f 1 (f 2 [9;16]))
```

Now it is `f 2 [9;16]` turn

```fsharp
let cont4 = initCont (f 1 [4;9;16])
```

And finally we have

```fsharp
let cont4 = initCont [1;4;9;16]
```

Now `initCont` also must be a *function*. But what do we expect it to do? Well nothing!
We just expect it to return it's input as-is. In other words. Our *starting function* is `id`. So
we start our `loop` with `loop id list`.

### All pieces together

Now let's put all pieces together, our final `foldBack` function now looks like this.

```fsharp
let foldBack f list (acc:'State) =
    let rec loop cont list =
        match list with
        | []      -> cont acc
        | x::list ->
            let newCont racc = cont (f x racc)
            loop newCont list
    loop id list

foldBack (fun x acc -> (x*x) :: acc) [1;2;3;4] []
// [1; 4; 9; 16]

foldBack (fun x acc -> x + acc) [1..10] 0
// 55

foldBack (fun x acc -> (string x) + acc) [1..10] ""
// "1234568910"
```

## Conclusion

Continuation functions in general are powerful and we can do a lot with them, solving `foldBack`
without reversing a list first is just one example. But in this case i have to admit that
understanding it is quite hard. It has nothing to do that *Continuation functions* are itself hard to
understand, the problem why it is so hard is because we cannot traverse a list backwards. Using
a continuation function is just one possible way to solve the problem. I think it is in general
worth to know and understand this solution.

In general I also hope that the analogy that functions just works on a *future value*
helps. At least for me this analogy was quite helpful in a lot of cases in the past.

One question remains. Which solution of `foldBack` should we prefer, and which is better? Well, at
first how do you measure "better"? Is it memory-consumption? CPU-time? Understanding code?

If you really care for *performance* you should benchmark. But make sure to test *small*,
*medium* and *big* input lists. In the previous `foldBack` version it doesn't seem that
*reversing* a list is the best option, as you have to go through a list twice and create
a new intermediate list.

But using this kind of *continuation* style isn't quite better. We basically still traverse a
list twice. We build first a continuation function, and in the end, we have to execute
it that touches every element again. A *continuation function* can result in higher
memory consumption and could be slower, *reversing* in the end can be faster and easier
to understand. But if you really care, then do benchmarks on your own.

But I think we should focus more on *readability*. *Reversing* is just quite easier,
so try to use the obvious solution first if you design things.

But at this point I also want to mention *mutability*. In functional programming in general
we don't care how a function works. Functions are black-boxes. We only care for the input and
the output of a function. It doesn't matter how it works, as long we have a *pure* function
and it expect and returns *immutable-data* everything is fine. Even if a function uses
loops, mutation or other kind of stuff. This is absolutely fine!

Controlled or limited *mutation* inside a function is absolutely okay. As long as that function
is *pure* and only takes and returns *immutable-data*. That doesn't mean everything i said
is useless.

It just explains the overall concept that is in general good to know and is useable in
different scenarios.
