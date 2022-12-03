---
layout: post
title: "Are dynamic-typed languages really faster to develop in?"
slug: are-dynamic-typed-languages-really-faster-to-develop
date: 2022-12-02
tags: [dynamic-typed,static-typed]
description: ""
keywords: static typed, dynamic typed
---

On Reddit i came [about a discussion](https://www.reddit.com/r/functionalprogramming/comments/z3cbrm/the_case_for_dynamic_functional_programming/)
of dynamic typed (functional) languages. A user suggested that in a dynamic typed language it
might be easier to return different types for a function call and thus overall
the development can be faster. He concluded.

> Having not to deal with types in that way when you refactor or build a
> system makes you significantly faster. Combine that with a proper testing
> approach and you have a reason to use dynamically typed languages.

I disagree and I think that dynamic typing in this case makes you slower overall
not faster.

Yes, in a dynamic typed language it is very easy to return different types
for a function call. So let's assume you had a function that always returned
`string`. And you change that function to now sometimes also return a
`list` of `string`s. This has severall implications.

Doing this change is very fast in a dynamic typed language, maybe the
code you have written might still work. But does that really mean your code
is correct? **No**.

Once you changed a function to return two different types you also must change
every function invocation and check the result what is returned in each case.
If you don't do that on every invocation you just have a bug in your program
that you might miss.

But a dynamically typed language:

1. Gives you no help to find all invocations.
2. You still need some kind of *type-checking*.

(1) is the reason why it actually turns out to be slower to develop, not faster.

(2) is the reason why you also could use a statically typed language.

The author mentioned how hard it is to return different types in a statically
typed languages. But that is not quit right. In statically typed **functional** languages,
and those we were talking at, it is quite easy.

For example in F# a type would look like this.

```fsharp
type StringOrListResult =
    | Single of string
    | Many   of string list
```

You also could return `Choice1Of2` and `Choice2Of2` as F# already provides
types to return different things in a generic way instead of creating
a new type. But the above code shows that returning different types and creating
such a type is not a problem about static-typing. More about languages
with a bad static-typing system. Not all static-type systems
in languages are equal!

Let's assume you had written a `func` and now instead of `string` it now
would return a type of `StringOrListResult`. Now every invocation in your
program will fail at compile-time. Probably already your IDE will highlight
all cases where you call `func` and you at least must change the call to
`func` with some kind of pattern matching. So you will change your code
to something like this.

```fsharp
match func x with
| String x -> // Code when string returned
| List  xs -> // Code when string list is returned
```

This kind of pattern matching is basically what you also do in a dynamic typed language.
It just looks way more ugly. For example in JavaScript you might write.

```js
let result = func(x);
if ( Array.isArray(result) ) {
    // Code when array
}
else {
    // Code when hopefully string
}
```

in Perl you might write.

```perl
my $result = func($x);
if ( ref $result eq 'ARRAY' ) {
    # Code when Array
}
else {
    # Code when hopefully string
}
```

In basically every dynamic typed language you need some way to check the type
and do something in the different cases. Those languages also have the tools
to check for types but at least in my experience this kind of checking often
just feel awkward.

Sometimes it is even hard to distinguish different types in a dynamic typed
language. For example how do you test in JavaScript if you have a number or
a string?

I guess people will probably come up with 20 solutions how this can be done
in JavaScript where each solution has dozens of pros and cons. People
will probably even ask: Are strings containing a valid number also considered
a number, or not?

But when we compare the F# with the JavaScript solution we can
see that they are similar.

Actually both has some kind of *Pattern Matching*. You must check the return
type. But in a dynamically typed language you are not forced todo so. But
not doing just means you **will have a bug in your code**.

A statically typed language will **force** you todo so as otherwise your code will
not compile.

A dynamically typed language will not help you to find all wrong invocations.
And wrong doesn't mean it throws an error. It could be that your invocation only
rarely returns a `list` as it depends on the input. And most of the time
your input returns a `string`. So most invocations seems to be *right*. Your
program seems to still work, well ... most of the time.

But all of this doesn't mean it is faster. You might think it is faster and you
probably did your change in a dynamically typed language faster. But this is
an illusion. The time you miss to change every invocation or later try to find a bug because you missed to change
a line of code will probably consume more time as you ever *safed*.

Especially changing or re-factoring is **a lot** more safe in a statically
typed language. And no, writing additionaly test doesn't make it faster. It
just makes it slower again as you also need to write tests.

And once you changed code you also must re-write your test-suite. This
is just additional stuff you must do. Nothing against tests. They have there
purpose. But it is always better if you don't must write a test at all.
