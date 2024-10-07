---
layout:  post
title:   "Algorithms: Filtering an Array"
slug:    algorithms-filtering-an-array
date:    2024-10-07T00:00:00
lastmod: 2024-10-07T00:00:00
tags:    [algorithms,data-structures,array]
description:
---

Let's say we wanna filter an Array. In general that means we want to run
some constraint on every element. The goal is to keep only the elements
that fulfills the constraint.

This is a very easy task, and yet there are multiple ways on how todo that.
Each solution offers different advantages and disadvantages.

But first let's pick a concrete example. We just have an array of integers and
we wanna keep only those that are even.

Here is one solution I would choose in Perl.

```perl
my @evens = grep { $_ % 2 == 0 } @numbers;
```

But, let's pick a more C-style of programming. The line above roughly translates
to.

```perl
my @evens;
for my $num ( @numbers ) {
    if ( $num % 2 == 0 ) {
        push @evens, $num;
    }
}
```

So, the code works the following:

1. Create a new empty array
2. Go through every element and check if it is an even number.
3. When it is an even number, push it into the `@evens` array.

The disadvantage of this approach is that this code takes up more memory (compared
to other solutions). We create a new array and don't modify the existing one.

On the other hand it has several advantages in my opinion.

First, the code is very easy, making this piece of code wrong is actually very
hard todo.

From a runtime perspective it's also quite good. We anyway must check every element
so a solution to this problem never can be less than `O(n)`. We must check every
element, and we either push the element onto the new array or don't.

In the *best-case* scenario all elements would be uneven, so we need to check every
element, but never push any element onto the new array so we end up with an empty array.

In the *worst-case* scenario all elements would be even and we actually would make a whole
copy of the array taking twice as much memory.

Neverthless we would say that both version takes `O(n)` timing even if the
worst-case solution would take a little bit more time.

# Fully mutable approach

The interesting aspect of the above code is that it is actually not a fully
mutable version. At some point it reminds on how you program in a fully immutable
world. There you would create a full new Array or sometimes an **immutable list**
depending on which language is used.

But how would a fully mutable version work? Okay, I don't have the patience to
write the code for that version but here is what you need todo.

First, you would need a function that is able to delete one element from an array.
How do you delete an element? Usually by moving every following element one to
the left.

So let's say you want to delete index 10 of an array. Then you move index 11
to index 10. index 12 to index 11, index 13 to index 12 and so on. You keep doing
that until you reach the end of the array.

Once you have such a function you can go through an array by it's index, when
the condition you have is not matched you remove the current index. Then you
re-check the current index again. No, you don't go to the next index, think twice
why you need to re-check the current index.

So what are the advantages and disadvantages of this approach?

One advantage is that it consumes less memory. No new array is created, instead
the current array is re-used. Actually this also can be an disadvantage, sometimes
you want to keep the original array and not modify an array.

One disadvantage is that it's performance is actually pretty worse. Think about it.

Let's say you delete index 0 out of a 1,000 big array. Then you actually need to
copy 999 elements and move 999 elements to the left.

Then you repeat checking index 0, let's say it also needs to be deleted, then
you now have to move 998 elements on to the left.

In a *best-case* scenario no elements needs to be deleted, that means no deletion
is done and this algorithm runs in `O(n)` time. The code just needs to check every
element but never does something besides that.

In a *wors-case* scenarion every element needs to be deleted. So you end up deleting
every element, and it is done by moving every one to the left. So in an array
with 1,000 elements it ends up doing 999 + 998 + 997 + 996 + ... + 1 operation.

When we are precisly it translates to a running time of `O(n ^ 2 / 2)` but overall
we just say it's `O(n ^ 2)` or a quadratic runtime.

# Comparison

One thing I have heard a lot when it comes to programming with immutability
is that people blindly repeat stuff that programing with immutability would be
slower. Well, the truth is that this isn't quite so easy.

The first version of creating a new array and copying elements to a new array
will usually outperform a fully mutable version, and it is also a lot easier
to implement.

This example also shows a property that general exists in algorithms and
data-structures. Usually we can make things faster by the expense of using more
memory.

# Resizeable Array

One interesting aspect is that the above algorithm and how it works expects
a growing/resizeable array. In fact when you don't have that, runtime between
both solutions can become even worse, on both solutions.

For example in C (but also C#/.Net) an Array is of fixed-size. You cannot add or
remove one element. Adding one element or removing one element means
creating a new array with one element less and copy all elements except the one
you don't want.

I guess this kind of working is what people usually have in mind when they think
about *immutability*, and yes, this kind of working would be even horrible. But
you could do better.

For example you could scan an array twice. First you count how many elements
are even, then create/allocate the target array, and then copy every matching array
to the new one.

Here is an interesintg question I don't know the answer myself. Would it be faster
to scan the array twice, calculate the size of the new array or use an resizeable
array that sometimes grows itself when you add an element?

I don't know, and usually I wouldn't care because roughly both solutions translate
to an `O(n)` algorithm having nearly the same amount of performance, I would
just pick a solution with an resizeable array because this solution is
brainlessly easy. And most modern language anyway sheeps with an resizeable array.

# Immutability

One thing I have learned or use more and more even unintentionally in whenever
I write code is to threat some of my data as immutable even when they are mutable.

I usually differentiate between a *construction phase* and a *normale usage phase*.

Like above, I may have an array `@numbers` and even if that array is mutable
and can be changed I usually would create a new array when I want to transform
the data, not change `@numbers` itself.

When I create `@evens` I consider that there is a *construction phase* where I
create my data, usually by mutation. But once this creation is done, I usually
try to never change `@evens` again.

And hey, when this kind of style works in a slowish dynamic typed language like
Perl that has no JIT compiler at all, believe me, it also works in a language
with a modern JIT optimized runtime.

# Questions

1. I can imagine a scenario that runs in under `O(N)` timing. Can you imagine how?
2. Are you aware of other solutions?
3. Can you imagine other advantages and disadvantages I didn't mentioned?
