---
layout: post
title: "Introduction to Functional Programming"
slug: introduction-to-functional-programming
date: 2016-05-11
tags: [FSharp,CSharp,intro,data,FPvsOO,currying,closure]
description: "Gives an introduction to functional programming and shows how the ideas relates to object-oriented programming"
keywords: f#, fsharp, introduction, functional, programming, oop, closures, currying, combinators
---

In this article I want to give a general introduction to some of the fundamental ideas of
functional programming. I just start with the idea of function as data, and explain
why functions are viewed as data and why it makes sense to pass functions as arguments.

When we understand this concept, I start explaining lambda expression,
currying, partial application and closures. All of this ideas built on each other.

But I don't stop at functional programming. Instead I will go back to OO programming
and show you, how you can translate all of these ideas into OO code. Probably
you will be surprised how similar functional and OO code is, and that most ideas
are things you already know.

Overall I show why functional programming and object-oriented programming are
orthogonal. I hope that by the end of the article you learned something about
functional programming, but also widen your view on object-oriented programming.

# Functional Programming

## Functions as data

One important concept in functional programming is the ability to use functions just
as data. This means you can create functions and store them in variables.
But that also means you can pass those functions to other functions as arguments
or retrieve a function from a function.

Sometimes people new to functional programming have some problems to understand
this idea and how it is useful, but in fact, when you do OO programming
you do that kind of idea basically all over the place. You do it even more
often as in a functional language.

But even if you don't see the connection at the moment, you still could ask
yourself if that idea really makes sense, or what useful thinks you can
do with that idea.

## What is a function?

Before we go deeper we have to ask ourself: What is a function anyway? Depending
on the language there are also multiple terms for the word function. Terms
like procedures, static methods or subroutines.

When I talk about functions I just mean the concept that you have some kind
of thing that you can pass some arguments, and it returns a result. As a simple
example we can think of a `square` function.

```fsharp
let square x = x * x
```

We can pass it several different values, and it will return a result.

```fsharp
square 0  // 0
square 1  // 1
square 2  // 4
square 3  // 9
```

Despite its simpleness. There are two ways how we can interpret `square`.

* A function is a series of commands that executes one by one returning some value.
* A function is a transformation of values. We pass some value in, and we get some value out.

Even if those definitions seems similar, the focus is different. The first definition is
often used by imperative languages. Functions are *just* a tool to get rid of
code-duplication. You have a series of commands? Put them in a function and you later
can call it again. What do you do if you want to understand
what a function does? Just explore the commands it executes step-by-step.

The second definition is how functional languages interpret functions. The focus lies on the
input and the output. A function is not just a series of commands, it transforms an
input to an output. You want to know what a function does? Examine the input and output
of a function. The best would be if the types of a function is already self-speaking enough.
Otherwise the function name itself should give us enough information what it does.

But we don't really care how a function work or how it exactly achieve the output
it returns. South Park teaches this thinking already:

![Underpants and Profit](south_park_profit.png)

A function is just something that takes some underpants, then do something, and
we get some profit out of it. We just have:

    Underpants -> Profit

What happens between those steps? We don't know, but we also don't care. The only
thing that matters is that we can somehow turn underpants into profit.

## Exploiting functions

The idea that only the input and output of a function matters is quite interesting.
We could take that idea further and for example rewrite our `square` function
into the following way:

```fsharp
let squareM x =
    let output =
        Map.ofList [
            (1, 1)
            (2, 4)
            (3, 9)
            (4, 16)
        ]
    defaultArg (Map.tryFind x output) 0

squareM 1  // 1
squareM 2  // 4
squareM 3  // 9
squareM 4  // 16
squareM 5  // 0
```

Okay, now you are probably saying that this is cheating and not the same. We also get
a wrong output when we pass it `5`. We only get correct output for the numbers one
to four. But actually the previous version was also not *correct*. When we do:

```fsharp
square 100000 // 1410065408
```

we also get a wrong result. The problem is that we have an integer overflow here.
`square` is also not correct, it only happens that our first `square` function returns
right result for a lot more arguments, but still not for all inputs.

But the more important idea is that we could replace a function just with a data-structure.
A functions just maps some input to its output. That is exactly what the `Map` data-structure
does.

<div class="info">
A <code>Map</code> is basically a key/value store. In F# it is immutable. In other
languages a <code>Map</code> is a Dictionary, Hash or Associative Array. In
non-functional languages they are most often mutable. Another example:
In JavaScript it is also called an object.

```js
    var squares = {
        1: 1,
        2: 4,
        3: 9,
        4: 16
    }

    squares[3] // 9
```
</div>

We could spent a lot of time filling out the remaining inputs in `squareM` to get it
*more correct*, but doing a task like this feels a little bit silly. But wait.

1. A function and a map data-structure are equivalent
1. It makes sense to pass a data-structure as an argument
1. Then it also must make sense to pass a function instead
1. If it feels silly to create all possible input/output combinations in
   a map data-structure to emulate a function, then passing a function
   makes more sense as passing a data-structure.

When you look at it then it indeed makes more sense. We actually can think
of a function just as a lazy-data-structure. Instead of generating all
possible input/output values that could exists and save them into a
data-structure. We just describe how every element can be computed, and
pass this description instead.

But lets at least once imagine our language don't support passing functions
as values. How could we create something similar to the `List.map` function?
This is the `List.map` function:

```fsharp
List.map square [1;2;3;4] // [1;4;9;16]
```

Without the ability to pass functions as values we just expect a `Map`
data-structure as the first argument instead.

```fsharp
// The map function with a `Map` data-structure
let rec map data list =
    match list with
    | []      -> []
    | x::list -> (Map.find x data) :: (map data list)

// We pre-compute the squares from 1 to 10000
let squares =
    Map.ofList [
        for x in 1..10000 do
            yield x, (square x)
    ]

// Now we call map with our pre-computed values
map squares [1;4;20] // [1;16;400]
```

Instead of writing all possible value combinations ourself we even could use the `square`
function to create the needed data-structure. This even shows more clearly why a function
and a `Map` data-structure is basically the same. Our own `map` function with a
data-structure is basically the same as the built-in `List.map`.

<div class="info">
The fact that our own <code>map</code> is not tail-recursive and fails with big input lists is not important
for this article. We could spent some more time in optimizing our own <code>map</code> and make it tail-recursive,
but the focus is not tail-recursion or performance, the focus is to understand that we can substitute
a function with a <code>Map</code> data-structure.
</div>

It is even interesting to see how similar both `map` functions are. When we look at both
definitions we see:

    List.map : ('a -> 'b) -> 'a list -> 'b list
    map      : Map<'a,'b> -> 'a list -> 'b list

`List.map` expects a function that maps `'a` values to `'b` values, while our own `map` expects
a `Map` data-structure that maps `'a` to `'b` values.

But once again, generating all possible input values before-hand feels a little bit silly.
We need to create a lot of possible values that we probably never need. It also costs more
memory as we need to save all possible combinations. That is why we pass a function instead.
We just pass the description how to compute the values instead.

<div class="info">
<p>
It is quite important to understand that passing a data-structure instead of a function is only
<em>theoretically</em> possible. <em>Practically</em> there exists a lot of cases where
such an approach will not work. When we for example try to create a data-structure that
maps any string to the length of a string, then it is practically impossible to create
a data-structure.
</p>

We actually would need to create a data-structure that contains any possible input string and returns
the length of it. Even when we just consider the stone-age view that they are only 128 characters
(aka ASCII) and we restrict us on a maximum length of 10 characters there already exists
1.180.591.620.717.411.303.424 possible input strings we need to handle.

Creating a data-structure that contains all possible input strings that maps it to the
length is theoretically possible. But already for 10 characters and just considering ASCII
we have such a large amount of possible input strings that it just exceeds the amount of memory
a single computer could have.

That doesn't mean passing functions doesn't make sense. It is quite the opposite, because the needed
data to represent a function is so huge, it is even more important that we can pass a function
that only calculates those things we truly need.
</div>

## Functions as return values

Let's consider the following function.

```fsharp
let generateAdd x =
    Map.ofList [
        for i in 1..5 do
            yield i, (i + x)
    ]

let add10 = generateAdd 10

add10.[1] // 11
add10.[2] // 12
add10.[3] // 13
```

We have a function that returns a new Map structure. When we call `generateAdd 10` we get
the following Map data-structure back.

```fsharp
[(1,11); (2,12); (3,13); (4,14); (5,15)]
```

But we already have seen this kind of code before. This was the way how we turned
our `square` function into a data-structure, so we could pass it to our own `map` function. But
this time we create a `Map` data-structure inside a function and return it from a function.
This is the same as creating a function inside a function and returning it.

When we look at `generateAdd` then we see the following. We loop over the number from `1` to `5`.
Those represents the inputs of a function we pre-calculate. With `yield` we return the mapping
`i` to `(i + x)`.

When we want to turn it into a function we just need to return the code `(i + x)` somehow.
So how do we return a function with that code? That is the purpose of *lambda*. *lambda*
is the idea of a function as a value. In F# we just use the `fun` keyword to create
a function/lambda.

```fsharp
let generateAdd x = fun i -> i + x
let add10         = generateAdd 10

add10 1   // 11
add10 2   // 12
add10 5   // 15
add10 100 // 120
```

Instead of pre-calculating `i + x` for a dozen of numbers, now we just return the whole
computation itself. When we call `generateAdd 10` we now get a new function back that
can turn any input `int` into an output `int` where we added `10` to it.

As you can see, both versions with a data-structure or the functions are quite similar.
But the last version still contains some interesting things that are worth to talk about.

## There is only lambda

When we want to create an `int`. How do we do that? Well we just write it. For example `5` is
just `5`. We can work with `5` however we want. We can pass it to a function, use it in
calculations and so on.

This works with any number, but when we for example want to work with `587452198` then
always rewriting this number can become annoying and tedious. Instead of working with numbers
directly we can bind a number to a symbol and give it a name. We bind something to
a name with `let`.

```fsharp
let x = 587452198
```

Once we have written this kind of thing, we now can use `bigNumber`. But what happens exactly
when we write something like this:

```fsharp
let y = bigNumber + bigNumber
```

A process named *Substitution* starts. The language cannot do anything with `bigNumber` but
`bigNumber` was anyway just a name for something different. The language just replaces `bigNumber`
with the number it stands for. And now we see something like:

```fsharp
let y = 587452198 + 587452198
```

After the *substitution* we can calculate the result `1174904396` and bind the result to `y`.
Now also `y` can be used in other calculations. This substitution process is quite
important. It is basically the foundation of a function. Previously we defined `square`
like that.

```fsharp
let square x = x * x
```

In fact, the language cannot execute anything in this example, as there is nothing to
execute. `x` has no meaning at this point. To really calculate something we must substitute `x` with
something different. How do we substitute it? We do it when we write:

```fsharp
square 10
```

We then say that `x` in `square` should be substituted with `10`. Instead of `x * x` it
substitutes `x` with `10` and it calculates `10 * 10`. What we see here are three things:

1. We have a concrete value like `10`
1. We can bind a concrete value to a symbol with `let`
1. Symbols get substituted with a concrete value

The interesting part is. Technically a function definition does not exists. The only
way to create a function is through the `fun` keyword (also named: lambda expression). So when
we want to write a calculation with a symbol. We just write.

    (fun x -> x * x)

How do we execute the lambda? We just provide a value that should be substituted for `x`
after the lambda expression.

    (fun x -> x * x) 2 //  4
    (fun x -> x * x) 3 //  9
    (fun x -> x * x) 5 // 25

But this overall makes less sense. We also could have written `2 * 2` or `4 * 4` directly. And
overall we don't want to rewrite the function all over again and again because that is annoying.
But we already have seen what we do in such a case. We just bind it to a symbol with `let`!

```fsharp
let square = (fun x -> x * x)
```

How do we work with `square`? You already have done it before.

```fsharp
square 5  //  25
square 10 // 100
square 25 // 625
```

Actually what really happens. `square` is just a symbol and it is once again substituted by
`(fun x -> x * x)`. When we write `5` after it, then `x` in our function gets substituted by `5`.
But creating functions and binding it to a symbol happens so often that we just have a shortcut
for that.

```fsharp
let square x = x * x
```

This definition is the exact same as the `square` definition with an explicit `fun`. That is also
the reason why calling `square` looks exactly the same. We have two different ways to define
a function.

```fsharp
let squareA   = fun x -> x * x
let squareB x = x * x
```

But there is no difference in calling both functions.

```fsharp
squareA 5 // 25
squareB 5 // 25
```

It just shows that the second `let` definition that already includes the function arguments
is only a shortcut to the more explicit lambda expression.

## Currying

In fact the simplification don't stop here. We don't even have functions with more than one
argument. There only exists functions with one arguments. So what do you do when you for
example want to add two numbers?

```fsharp
let add =
    fun x ->
        fun y ->
            x + y

add 10 20 // 30
add 50 50 // 100
```

But creating such kind of nested functions is quite annoying. So there is a shortcut. You just can
use `fun` with multiple arguments and it creates the nesting for you.

```fsharp
let add' =
    fun x y ->
        x + y

add' 10 20 // 30
add' 50 50 // 100
```

And there is once again a shortcut by just using `let`

```fsharp
let add'' x y = x + y

add'' 10 20 // 30
add'' 50 50 // 100
```

In practice you will usually only use the last way of defining a function. But it is quite
important to understand that all three ways of writing a function are interchangeable
and mean the exact same.

How does those information help us? We can rewrite our `add10` function that we created
previously. Previously we wrote `add10` like this.

```fsharp
let generateAdd x = fun i -> i + x
let add10         = generateAdd 10
```

But this is the same as:

```fsharp
let add x y = x + y
let add10   = add 10

add10 5  // 15
add10 10 // 20
```

It is quite important to understand what happens here. We start with `add`, but this is
a function that expects a single value `x`. That is the reason why we just can call `add 10` with
one argument. This then returns a lambda like `(fun y -> 10 + y)`. That means
we substituted `x` with `10` and then returned a new function.

This new function is then bounded to the symbol `add10`. When we finally call `add10 5` then
we also substitute `y` and we get `10 + 5` and this evaluates to `15`.

Only passing some arguments is what we call *Partial Application*. But it is more important
to understand that we get *Partial Application* for free because we have Currying and all
functions are one-argument functions that return new functions.

<div class="info">
<p>
If you wonder about the apostrophes in <code>add'</code> or <code>add''</code>. They are
not special and just part of the function name. This is usual the way how
functional programmers say: This is another version/implementation of a function.
</p>

I also could just numbered the functions with <code>add2</code>, <code>add3</code> and so on.
But this creates conflict with Partial Applied functions. When i write <code>add3</code> i
would expect that it is a partial applied function with the first argument set to <code>3</code>.
</div>

## Closures

Previously I said that when you call a function then some kind of substitution happens.
When you call `add 10` in the last example then `x` gets substituted by `10` and it returns
a function `10 + y`. But this is not quite correct. What really happens is that the actual
variable `x` is just *remembered*.

It seems not like a big distinction, but the difference becomes more obvious when we use
a mutable variable.

```fsharp
let x       = ref 10
let add x y = !x + y
let add10   = add x

add10 5 // 15
x := 5
add10 5 // 10
```

Because the value of `x` changes between both calls we now get `15` and `10`. The
reason why we anyway get different values is because the returned function only refers
to `x` and does not do a real substitution of the value.

<div class="info">
<p>
Usually the difference between referring and substitution only becomes a problem with
mutable variables. Something we anyway avoid in functional programming. When we think
of referential transparency (this is how functional programming is usually defined)
we could even say that this code is not functional at all.
</p>

But i don't go further into this topic. Usually we don't use mutable variables, but
as F# is not a pure-functional language and it supports mutation, i still think it
is important to mention.
</div>

But there is an even more fundamental idea that emerges from that example. How long do
we need to keep `x` in memory? The answer is, as long we have some code that still refers
to it. In our case `add10` refers to `x`, we always must keep `x` in memory, as long
we have access to `add10`.

This is also the reason why the first Lisp compiler already provided automatic memory
management with garbage collection (invented 1962). More precisely I don't even know
of any functional language that don't provide automatic memory management. While it
might be theoretically possible to not provide automatic memory management. It
is probably not a good practical decision.

Whenever we *remember* a variable or refer to a variable from a lambda expression,
we call it a *Closure*.

## Example: Currying and Closures

I want to give a small example that shows functions as value and return values,
currying and closures all in action. In F# we have an option type. An option type can have
two states, `Some` or `None`. The `Some` state can carry an additional value with it.
Usually the option type is used for the idea of *No Value*, but it also can be used as
the idea of a *Success* or *Failure*, or how I use it as *Valid* or *Invalid*.

We could for example write two functions that check if a number is greater or smaller
than a limit. If the number is valid (smaller or greater) then we just return the number
unchanged, otherwise we return `None`. But it also could be that we already get a `None` as
input. In this case we just return `None`. We could write `smaller` and `greater` like this.

```fsharp
let smaller min x =
    match x with
    | None        -> None
    | Some number ->
        if   number < min
        then Some number
        else None

let greater max x =
    match x with
    | None        -> None
    | Some number ->
        if   number > max
        then Some number
        else None

smaller 10 (Some 3)  // Some 3
smaller 10 (Some 11) // None
greater 10 (Some 3)  // None
greater 10 (Some 11) // Some 11
```

But when we look closer, `smaller` and `greater` are nearly identical functions. The only
difference is the `if` check itself. But what does the `if` anyway? The `if` itself
just turns a `number` somehow into a boolean value.

Instead of hard-coding the `if` behaviour we also could call a function that we give the number
and returns us a boolean. This way we could get rid of the whole code-duplication. We just
abstract both functions into a new function that expects a function for the transformation
of `number -> Bool`.

With such a new function we then could easily rewrite `smaller` and `greater`.

```fsharp
let is predicate x =
    match x with
    | None        -> None
    | Some number ->
        if   predicate number
        then Some number
        else None

let smaller min x = x < min
let greater max x = x > max

is (smaller 10) (Some 3)  // Some 3
is (smaller 10) (Some 11) // None
is (greater 10) (Some 3)  // None
is (greater 10) (Some 11) // Some 11
```

Now the whole code is written with currying in mind. `is` expects a function that turns
a number into a `bool`. When we write `(smaller 10)` we just get back such a function.
We get back a new function that contains `10` as a closure. This function is then passed
as a value to the `is` function.

We also could nest the calls, for example when we want to do two checks at once. The second
argument to `is` must be an option value. But this is also what `is` returns. So we could
use the output of another `is` as the input for the first `is`. What we then get is very
Lisp-like code.

```fsharp
(is (greater 0)
  (is (smaller 10)
    (Some 3))) // Some 3
```

At least in F# most people would prefer the piping idiom. With pipe we can write
the last argument of a function in front of the function. With this idea we end up with a
more sequential way.

```fsharp
(Some 3)
|> is (greater 0)
|> is (smaller 10) // Some 3
```

`(Some 3)` is now the input to `is (greater 0)` the return value of this is then the input value
to `is (smaller 10)` and so on. But checking if a number is between some values is quite common.
Why not combine both things into a single function?

We already have `is` that does the handling of the option for us. But it only works
with a single predicate function. This is okay, but to be more flexible we should write
a way to combine two predicate functions into a single new predicate function.

<div class="info">
A predicate is a function that always returns a <code>bool</code>. Predicates are often used for
validation or filtering. For example:

```fsharp
[1;2;3;4;5;6] |> List.filter (fun x -> x % 2 = 0) // [2;4;6]
```
</div>

```fsharp
let combine f g x = (f x) && (g x)
```

The idea is simple, we have two functions, and both functions must return `true` for the given
input `x`. Only when both return `true` our `combine` function also returns `true`. Now we can
easily combine two predicates into a single new predicate.

```fsharp
let between min max = (combine (smaller max) (greater min))

(Some 0)  |> is (between 0 10) // None
(Some 3)  |> is (between 0 10) // Some 3
(Some 11) |> is (between 0 10) // None
```

Once again you can see how we use currying. We don't provide the second argument to `smaller` or
`greater`. That means both calls return predicates. And those two predicates are then provided
to `combine`.

But we also don't provide the third argument to `combine`. That means once again we get another
new predicate function back. This way we can create the `between` predicate. It doesn't look like
it, but `between` is a function that takes three arguments. `min` and `max` are the visible
arguments. But we only called `combine` with two arguments. That means it returns a lambda,
that makes `between` a three argument function (or a chain of three functions taking one-argument).

What do we do if we want to combine three, four or more predicate functions? `combine` currently
can turn two predicates into a single predicate. So we only need to `combine` all our predicates until
we end up with a single predicate. This task is already written for us and named `reduce`.
Let's create a `check` function that we can pass a list of predicates that combines it
into a single predicate.

```fsharp
let check predicates = List.reduce combine predicates
```

Now let's combine three predicates at once to create a new predicate.

```fsharp
let isEven x    = x % 2 = 0
let evenAnd1To9 = check [greater 0; smaller 10; isEven]

(Some 0)  |> is evenAnd1To9 // None
(Some 3)  |> is evenAnd1To9 // None
(Some 4)  |> is evenAnd1To9 // Some 4
(Some 6)  |> is evenAnd1To9 // Some 6
(Some 11) |> is evenAnd1To9 // None
```

This idea of writing small functions, or basically decomposing a task into small functions and
then composing them into bigger functions is the heart of functional programming. In functional
programming we directly work with functions and we even write our own combinators
to compose functions.

# Object-Oriented programming

At the start of the article i said we can achieve the same ideas in Object-Oriented programming
or that you probably already know these things. I want to show you some C# code to achieve the
same things. C# already has some functional features like lambda-expressions. But there is
really no point in showing C# code that uses functional features as an example that OO
and functional programming are orthogonal. Because of this i will only use classes.

## What is a class?

We start with the same idea. What is actually a class anyway? A class is actually some
kind of compose-able type. It always has at least one constructor, beyond that it can contain
multiple data in the form of public and private fields, additional it can contain multiple
functions operating on those data, often named *methods*. I make no distinction between
functions and methods.

But one important aspect is that there is no technical restriction to create classes with no
data or no functions. There is also no restriction that tells you how many data fields or
functions you must implement.

## Function as data

I started with the idea of functions as data. I created a `square` function and passed the function
around. We wrote our own `map` function that showed that we could replace a function by data.
But also the opposite that we could pass a function instead of a data-structure. Re-doing that
proof makes less sense as this idea doesn't invalidate just because we now use classes. But
you probably wonder how we pass a function.

In fact, when you do OO programming you pass functions all over the place in your code. You do
it even more often as in a functional language. Every object is a container for functions. So
when you pass an object you also pass functions around. In fact you not only pass a single
function, you usually pass dozens of functions including data as a single object around.

And because you usually return objects that include functions, you also return functions. In fact
in OO it is even hard to find a place where you don't do that. Functional programming
is more a simplification. We only pass a single function. And on top, we have lambda expression
to easily create a single function directly where we need it. That is why it looks different
but actually passing a single function or returning it shouldn't be hard to understand. You
already do that all over the place in an OO language.

<div class="info">

Some languages also have the ability to create whole classes/objects inline like you do with a lambda expression. In fact F# is such a language and supports
<a href="https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/object-expressions">Object expressions</a>
C# also supports <a href="https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/anonymous-types">Anonymous Types</a>,
but they are more limited. Java supports <a href="https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html">Anonymous classes</a>.
</div>

So, how do we create a `List.map` function in C# just with classes and how do
we pass it a function like `square`?

```csharp
public interface IFunction<A,B> {
    B Call(A a);
}

public class Square : IFunction<int,int> {
    public int Call(int x) {
        return x * x;
    }
}

public static class List {
    public static List<B> map<A,B>(IFunction<A,B> func, List<A> values) {
        var newList = new List<B>();
        foreach (var x in values) {
            var mapping = func.Call(x);
            newList.Add(mapping);
        }
        return newList;
    }
}
```

At first, we need to create a `IFunction` interface. This interface just tells us that we have
a single method that takes some input and produces some output. We don't even care how that method
is named. I use `Call`, but we also could use `Run`, `Execute` or whatever you like.

Because we want to pass the square function as a value, we now must write a whole class and wrap
our function inside a class. Now we are able to create an object and pass that function around.

Our `List.map` just expects such a one-method interface (functional interface) and executes the
function for every element. We now can write something like this:

```csharp
var ints    = new List<int> { 1,2,3,4,5 };
var squared = List.map(new Square(), ints);
```

And `squared` is a new list with all elements squared. The `IFunction` interface is sure a thing
you only need to write once. Because we don't have currying in C#, we probably also want to
write a version with two or three arguments, but overall you only need to define those once.

Writing everything this way feels a little bit dumb. Mainly because most often the time there is a
(IMHO: dumb) rule that tells you to put every class into its own file. The above code leads to
an explosion of classes/files. But instead of criticizing the code, you should probably criticize
your rules and OO on why such a simple example is already so complex.

<div class="info">
Once again I want to highlight the importance that i ignore the functional features. You
sure could easily achieve the same with just static methods (functions). Using <code>Func<int,int></code>
in <code>List.map</code> and using a lambda <code>x => x * x</code> to call <code>List.map</code>.
But those are the functional features that were added with C# 3. There is no point in showing
functional concepts with functional features in an OO language to prove that they are orthogonal.
</div>

## Currying, Partial Application and Closures

All three things are somehow connected to each other. In the functional code I first introduced
currying, but I primarily used it to show Partial Application (only providing some arguments
to a function, not all) and explained why this needs the concept of a closure. I will first ignore
currying and only talk about the later two. So how can we create a function like `add` and partial apply
the first argument?

```csharp
public class Add : IFunction<int,int> {
    private int x;
    public Add(int x) {
        this.x = x;
    }
    public int Call(int y) {
        return x + y;
    }
}
```

We now can use it like that:

```csharp
var add10 = new Add(10);
add10.Call(5); // 15
```

What is Partial Application in OO really? It is just an argument to a constructor that is saved in
a private field. All other methods in your class then have access to this private field. The value
that was passed to the constructor is *remembered*.

This shows what a closure is. Our `add10` object just have some internal private fields and
all of those fields must remain in memory as long we have access to `add10`. This shows one
fundamental idea. A closure and an object is the same.

Whenever you create an object in OO programming. It is the same as calling a function that returns
a function. The returned function then has access to the input through a closure. In the functional
code I only returned a single function, but you also can return multiple functions or data-structure
like a *Tuple* or *Record* that contains those functions.

In C# we could for example create a class like:

```csharp
public class Counter {
    private int counter;
    public Counter(int init) {
        this.counter = init;
    }
    public int Current() {
        return this.counter;
    }
    public void Increment() {
        this.counter += 1;
    }
    public void Decrement() {
        this.counter -= 1;
    }
}
```

and use it like this:

```csharp
var counter = new Counter(10);
counter.Increment();
counter.Increment();
counter.Decrement();
Console.WriteLine(counter.Current()); // 11
```

In F# (without using classes in F#) we could achieve the same just with a closure

```fsharp
// A Record describing three functions
type Counter = {
    Current:   unit -> int
    Increment: unit -> unit
    Decrement: unit -> unit
}

// A function that has `counter` as a closure and returns a Counter Record
let counter init =
    let counter = ref init
    {
        Current   = (fun _ -> !counter)
        Increment = (fun _ -> counter := !counter + 1)
        Decrement = (fun _ -> counter := !counter - 1)
    }
```

And you use it like that:

```fsharp
let count = counter 10
count.Increment()
count.Increment()
count.Decrement()
printfn "%d" (count.Current()) // 11
```

An object is just a collection of functions that still has access to some hidden fields.
Objects and closures are the same. Probably you have heard that "Objects are poor man closures",
now you know why. But it is also the same reversed. "Closures are poor man objects". Why? A
class is basically an *optimization* of this use-case.

A class contains the definition, fields and so on in one unit. Instead of creating a record
definition, using `counter` as a closure and return a record, I also could just define
a class with the members (methods) and a private field. Defining a class is shorter.

So a class is a poor man closures because it did not add anything more useful as what a
function with closure already gives you (functional languages, lambdas, closures and so
on already existed before OO). But on the other hand, OO optimized this use-case in such
a way that Closures are really "poor man objects".

As F# also supports classes, if you really want to write something like this i would suggest you
also should create a class, and not use a record with functions and a closure. A class is just
much shorter.

```fsharp
type CounterClass(init:int) =
    let mutable counter = init
    member this.Current     = counter
    member this.Increment() = counter <- counter + 1
    member this.Decrement() = counter <- counter - 1
```

The usage of a class in F# is actually nearly the same as the code with the record:

```fsharp
let count = CounterClass 10
count.Increment()
count.Increment()
count.Decrement()
printfn "%d" count.Current
```

But as a final note. Writing things like this is anyway not idiomatic functional code. Functional
code uses immutability. With immutable data you don't really need a class at all. A class gives you
the ability to hide *mutable data* so you only have specific functions that can manipulate those.

## Currying

F# does automatic currying by default, but it doesn't mean just because your language doesn't do
currying you can't have it. While most modern functional languages usually do it by default some
older languages especially those based on Lisp also don't do it by default.

I give a short recap on what currying is, because it is still often miss-interpreted as
partial application.

Currying is the idea that every function only has one input and one output argument. A function
with multiple input arguments must then be converted into a function that returns another
function. This transformation process is what we name currying. Once a function is in curried
form we don't have to specify all arguments at once. Not providing all arguments at once is
partial application. But you don't need currying to use partial application.

Often beginners have a problem to differentiate both. We also can write a curry
function in F#. And probably that can help to demonstrate the distinction. First
we create a function that expects two arguments in tupled form.

```fsharp
let add (x,y) = x + y
```

Now `add` expects a single argument, a tuple that contains two variables. You can use it like that:

```fsharp
add(3,10)  // 13
add(10,10) // 20
```

Probably at this point you will notice how similar this is to a function in a C-style language.
It is probably not an accident that we use `,` for the creation of a tuple. The disadvantage
of a tuple is that we now must provide both values at once. We now could write a `curry`
function to turn a function expecting a tuple into a chain of functions.

```fsharp
// Extended version
let curry =
  fun f ->
    fun x ->
      fun y ->
        f (x,y)

// Short version
let curry f x y = f(x,y)
```

Using `curry` only with the first argument returns a curried version of `add`. After we have
a curried function we can partial apply it.

```fsharp
let addC   = curry add
let addC10 = addC 10
addC10 5 // 15
```

To be more formal. We started with a function signature that looked like this:

    (A,B) -> C

and `curry` turned it into a chain of functions

    A -> B -> C

When you are not used to reading functional signatures. The `->` stands for a function. The left
is the input and the right is the output. `-> ` is left-associative. So you can read it like

    A -> (B -> C)

A function that takes `A` and returns a new function that takes a `B` and returns `C`. When we
now go back to the OO world. We already defined a generic interface with one argument and output.

    IFunction<A,B>

How could the interface for a function with two arguments, without currying, look like?

    IFunction<A,B,C>

How does it look with currying?

    IFunction<A,IFunction<B,C>>

A `curry` function would take a `IFunction<A,B,C>` and return a `IFunction<A,IFunction<B,C>>`.
The problem is, without the ability to create a lambda or anonymous classes such a task is hard
up to impossible.

Instead of focusing on currying, we could focus on partial application instead. We already
have seen what partial application means in OO. It just means we already provide the arguments
to the constructor when we create a class. But it is really bad that we have to do that kind
of thing explicitly and manually.

Instead of creating classes that expects its value explicitly in the constructor, we should
write a generic version that can partial apply any two argument IFunction instead. We
could write it like that:

```csharp
public interface IFunction<A,B> {
    B Call(A a);
}

public interface IFunction<A,B,C> {
    C Call(A a, B b);
}

public class Partial<A,B,C> : IFunction<B,C> {
    private A a;
    private IFunction<A,B,C> func;
    public Partial(IFunction<A,B,C> func, A a) {
        this.a = a;
        this.func = func;
    }
    public C Call(B b) {
        return this.func.Call(this.a, b);
    }
}

public static class Func {
    public static IFunction<B,C> partial<A,B,C>(IFunction<A,B,C> func, A a) {
        return new Partial<A,B,C>(func, a);
    }
}
```

We don't need the static `Func` class, but without this helper function and by directly
using `new Partial()` we need to specify the generic values like `new Partial<int,int,int>()`
as otherwise the C# compiler cannot infer the types. The helper functions helps us here.
With such a setup we now can write our `Add` function like this.

```csharp
public class Add : IFunction<int,int,int> {
    public int Call(int x, int y) {
        return x + y;
    }
}
```

And now we are able to partial apply the first argument. We can work with it like this.

```csharp
var add   = new Add();
var add10 = Func.partial(add, 10);
Console.WriteLine(add10.Call(5)); // 15
```

It is a little bit boiler-plate to define everything, but we only need to define it once,
now we are able to partial apply any two argument function without that we explicitly
need to create a private field or think about which or how many arguments we want
to partial apply.

<div class="info">
When we don't restrict ourself and use all functional features of C#, that means lambdas,
and static methods (that we then can pass as arguments) then all of the examples become easier.
We also can easily define <code>curry</code> functions.

```csharp
public static class Lambda {
    // 2-args
    public static Func<A,Func<B,C>> Curry<A,B,C>(Func<A,B,C> func) {
        return a => b => func(a,b);
    }
    // 3-args
    public static Func<A,Func<B,Func<C,D>>> Curry<A,B,C,D>(Func<A,B,C,D> func) {
        return a => b => c => func(a,b,c);
    }
}

class MainClass {
    public static int Add(int x, int y) {
        return x + y;
    }

    public static void Main(string[] args) {
        // We must specify the generic-types when we pass a static method
        // to the Curry() function as an argument
        var add = Lambda.Curry<int,int,int>(Add);

        var x = add(10)(5);   // It is now a chained function
        Console.WriteLine(x); // 15

        var add10 = add(10);  // Easily Partial Application
        var y     = add10(5);
        Console.WriteLine(y); // 15
    }
}
```
</div>

## Exercise

Previously I provided a small exercise with validation, but I leave the task to implement it
in your favourite language. Up to this point you should know enough about currying and
partial application. Most languages today also support lambda statements. Not every language
has automatic currying, but I showed how to create a curry function in F# and C#.

Otherwise you still can use partial application instead of currying. The example is still
small and still heavily rely on currying or in general the ability to take functions as
arguments and return new functions from other functions. When you want a better understanding
of the concepts then there is no better way to somehow rewrite the given example.

As a full overview, here is the full F# code.

```fsharp
// Takes a predicate and a value and returns a valid/invalid value
let is f x =
    match x with
    | None        -> None
    | Some number ->
        if   f number
        then Some number
        else None

// Combinator
let combine f g x = (f x) && (g x)

// Predicates
let smaller min x   = x < min
let greater max x   = x > max
let even x          = x % 2 = 0
let between min max = (combine (smaller max) (greater min))
```

Some notes on the implementation. `Some` and `None` is an option type. In a language without
[Algebraic Data-Types]({{< ref 2016-04-26-algebraic-data-types >}}) you have some problems to
build this. But you can create a class that contains a bool and a data field. The bool contains
the information if the data-field is valid or not. In the case it is invalid, the data-field
is empty/null.

`combine` is a combinator, it expects two predicate functions as an argument and returns
a new predicate that applies both checks on a value. In a language without automatic
currying this should be a two-argument function (returning a function).

You can use the functions above like this:

```fsharp
(Some 3)
|> is (between 0 10)
|> is even // None

(Some 4)
|> is (between 0 10)
|> is even // Some 4
```

When you get stuck writing it in such a sequential way, first try to write it in a nested style.

```fsharp
(is (between 0 10)
  (is even
    (Some 3))) // None

(is (between 0 10)
  (is even
    (Some 4))) // Some 4
```

When you have problems to understand the nested-style. The only difference in C-Style code is
the position of the open-parenthesis. It is placed after the function name instead before.
And often `,` is used to separate the arguments.

```csharp
is(between(0,10),
    is(even,
    Some(4)))
```

Writing it in a sequential style is possible in every language that also supports lambdas. In C#
you want to look at *Extension Methods*, in Java look at *Default Methods*. In a language without
such features. The `is` function will be part of your validation class. Your final code should
then look similar to this:

```csharp
var x =
    new Validate(3)
    .is(between(0,10))
    .is(even);
x.IsValid() // False
```

But first try to write it only with functions/static methods. The predicates itself
like `smaller`, `greater` and so on should never be part of the validation class you create.

When you create a solution in your favourite language, you can leave a message in the Disqus
chat, or sent me a notification via Twitter, I add a link to your solution here.

# Summary

I started with the idea of functions as data and why it makes sense that we can pass functions
as arguments or return functions from other functions. A lambda expression is a way to
create a function on-the-fly, so we can easily pass functions as arguments or return them from
functions without explicitly defining them. Then we learned that a language like F# basically
treats all functions just as lambdas. We also have seen that multi-arguments functions didn't
exists, they are just a chain of one argument functions. This on the other hand means we can easily
partial apply any function. But not only that, it means every multi argument function is automatically
a function that can generate other functions.

When we looked at C# we basically re-implemented all the ideas and we have seen how those
ideas translate to OO. You also should now know why Functional Programming is Orthogonal
to Object-Oriented programming and why closures and objects are the same.

I overall hope that this introduction helped you not only in understanding functional programming
better, but also widen your view on object-oriented programming.

# Further Reading

* [What's Functional Programming All About?](http://www.lihaoyi.com/post/WhatsFunctionalProgrammingAllAbout.html)
* [So You Want to be a Functional Programmer (Part 1)](https://medium.com/@cscalfani/so-you-want-to-be-a-functional-programmer-part-1-1f15e387e536)

