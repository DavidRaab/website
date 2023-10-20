---
layout: post
title: "Perl vs. JavaScript: Scalar and List Context"
slug: perl-vs-javascript-scalar-and-list-context
date: 2023-10-20
tags: [perl,javascript,perl-contexts,language-design]
description: How Perl's scalar and List Context translate into different languages.
---

Something I like in Perl, and also dislike sometimes, is Perl distinction between *Scalar-* and *List Context*. Something I haven't seen in any other language so far. In Perl any function call can happen in two different context. And depending on the context a function can return different things, or even behave differently. But it also effects how arguments are passed to a function.

Let's see an example. I want to write an easy `sum` function that sums up all numbers given to it. In Perl I would write it like this.

```perl
sub sum {
    my $sum = 0;
    for my $x ( @_ ) {
        $sum += $x;
    }
    return $sum;
}
```

Because Perl automatically has a *List Context* for all function parameters. You always can pass as many arguments as you want. In other languages this is often seen as *vararg* or something similar. All arguments end up in the special `@_` array. So you can just work with all arguments as if you have gotten an array.

As an example i now can call `my $sum = sum(1, 2, 3, 4, 5);` to get the result of `15`. But it also allows many other different calling styles.

```perl
my @arr1 = (1,2,3,4);          # array
my @arr2 = (5,6,7,8);
sub result1 { return 1,2,3,4 } # function returning multiple values
sub result2 { return 5,6,7,8 }

say sum(1, 2, 3, 4, 5);        # 15
say sum(@arr1);                # 10
say sum(@arr1, 5, 6, 7, 8);    # 36
say sum(@arr1, @arr2);         # 36
say sum(result1());            # 10
say sum(result1(), result2()); # 36
```

In some way you can think of it, that Perl doesn't pass an array as a data-structure to a function. Instead it will expand the array into all its values. So calling `sum(@arr1)` is really just the same as writing `sum($arr1[0], $arr1[1], $arr1[2], $arr1[3], $arr1[4])`. The beauty of it is that it becomes very flexible. You can easily call multiple functions and sum up the result of a multiple function calls. You can have multiple arrays, sum them up, add additional values to it, and so on.

The bad part is, you pay for it, with performance. As every array get expanded, passing it this way can be expensive. If you have an array with 1,000 elements then you really are calling `sum` and pass it 1,000 arguments. This is slower as just passing a whole array. Or as many other languages do, they just pass a reference to an array. Well, you can do the same in Perl. But you have to explicitly create a reference to an Array; Python developers will love Perl obviously!

Compared to JavaScript, C#, Java, Python, ... you have an explicit distinction between a value and a reference pointing to it. Like you can see in `C` or `C++`. In "modern" languages like JavaScript, C#, Java, Python, ... on the other hand you also have this distinction. In C# for example you must learn the distinction between a *Value Type* and a *Reference Type*. Java names it *Primitive Types* and so on. But that distinction is somehow hidden from you. Not visible in the source code or type you use.

Let's see the same in JavaScript

# JavaScript sum function

In JavaScript I would write the `sum()` function like this:

```js
function sumA(array) {
    var sum = 0;
    for (i=0; i < array.length; i++) {
        sum += array[i];
    }
    return sum;
}
```

Here it will take an array. Let's compare the different calling styles to Perl.

```js
var arr1 = [1,2,3,4];
var arr2 = [5,6,7,8];
function return1() { return [1,2,3,4]; }
function return2() { return [5,6,7,8]; }

console.info( sumA(1,2,3,4,5)   );          // 0
console.info( sumA([1,2,3,4,5]) );          // 15
console.info( sumA(arr1) );                 // 10
console.info( sumA(arr1, arr2) );           // 10
console.info( sumA(return1()) );            // 10
console.info( sumA(return1(), return2()) ); // 10
```

First of, functions in JavaScript cannot return multiple values. From the standpoint of Perl we can say, it only has *Scalar context*. It only can return a single value. To return multiple values we **must** return an array instead.

A call like `sumA(1,2,3,4,5)` doesn't work as intended. I executed the code with `Node.js` and it returns `0`. Actually I expected an error. But, who cares? Me not.

Obviously we need to pass an array instead, so at least call `sumA([1,2,3,4,5])`. All other functions just return `10` because we just iterate over the first argument (the array) and just sum those values up. Ignoring all other additional arguments/arrays. Actually, it even pisses me off that it is not an error to pass more than one argument. Yes, Perl do the same, but in Perl I also
don't define a function signature! And when I do (added as loadable feature with Perl 5.36) it will throw an error on invalid ariety.

But, we also could have chosen another implementation of the `sum` function. In Perl we had the special variable `@_` that is technically all arguments as a single array. JavaScript offers the same, it has a special `arguments` variable containing all passed arguments. So we can use this instead.

```js
function sumB() {
    var sum = 0;
    for (i=0; i < arguments.length; i++) {
        sum += arguments[i];
    }
    return sum;
}
```

But doing so, is in my opinion counterproductive in JavaScript. Now we get the following results.

```js
console.info( sumB(1,2,3,4,5)   );          // 15
console.info( sumB([1,2,3,4,5]) );          // 01,2,3,4,5
console.info( sumB(arr1) );                 // 01,2,3,4
console.info( sumB(arr1, arr2) );           // 01,2,3,45,6,7,8
console.info( sumB(return1()) );            // 01,2,3,4
console.info( sumB(return1(), return2()) ); // 01,2,3,45,6,7,8
```

Only the first call now works. All other calls actually do garbage. I guess JavaScript is doing string concatenation when you do `+` on whole Arrays. Whoever thought this is useful in any way.

You still could *fix* all of the calls and make them work. And this is possible for both implementations of `sum()`. For example instead of calling `sumA(arr1, arr2)` we could first concatenate both arrays and then pass it to `sumA`. So instead of
`sumA(arr1, arr2)` that didn't worked, we need to write `sumA(arr1.concat(arr2))`. We need to concatenate all arrays into a single array before passing.

How do we make the `sumB` call work? There are two solutions. One solutions that is older, and i consider *more right* is:

```javascript
sumB.apply(null, [1,2,3,4])
```

Or since EcmaScript 6 we also can use array expansion.

```javascript
sumB(...arr1);
```

I guess calling it with `apply` compared to the array expansion is faster. But i don't know, also could be that those JavaScript compilers optimize it. Maybe you are asking why we should ever implement `sum` like in `sumB` or why we ever should do it this way?

# Math.max

This is just an example. But you see quite often that some APIs, functions or libraries chose a variable amount of arguments. A built-in example in JavaScript is `Math.max`. The JavaScript developers have chosen an API with variable arguments. So you call.

```javascript
Math.max(1,2,3,4) // 4
```

But at least when i write code. I usually have an array instead of passing multiple arguments. I guess you also probably have come across functions that you want to pass an array instead of multiple values. And `apply` fixes this.

```javascript
var someArray = /* from some function call */
Math.max.apply(null, someArray);
```

At least in JavaScript it is always bothersome if you have APIs that expect *variable arguments* because nearly 99% of the JavaScript APIs work with Arrays and return Arrays. At least, that is my perception, not something i can, or want to proof.

# Conclusion

Maybe you are asking why I write about it. First of, it isn't about which language is better or worse and so on. Here is how I think about it: As soon a language introduces the concept of *variable arguments*. It also makes some parts of the language more difficult or easier.

In Perl basically every function call is *variable arguments*. With it's *scalar-* and *list-context* it tries to cope with this feature. But how do you pass two distinct arrays in Perl? In Perl you must explicitly create references instead. So passing two distinct arrays in Perl becomes `sum(\@arr1, \@arr2)` instead of `sum(@arr1, @arr2)`. Perl have chosen that variable arguments and expansion of arrays is easy, while passing references is harder.

Most other languages do the opposite. They make passing references easier, as everything is a reference by default, and make expansion harder. But as soon this feature is added in a language, you will face one of the following problems:

1. You either have an array and want to pass it to a function with *variable arguments*.
2. You have a bunch of variables and are forced to pass it as an Array/List.

Because arrays are some kind of reference type in most languages, those languages need special treatment like the `apply` function in JavaScript. Or array expansion `...array`. It makes passing an array as variable arguments harder. (`apply` originates from [Lisp/Scheme](https://docs.racket-lang.org/guide/application.html#%28part._apply%29) btw.)

The second problem is not a real problem in some languages. In Perl and JavaScript you just write `[1,2,3,4]` to create arrays as references. So those languages offers a built-in way in the syntax to create arrays. But not every languages offers an easy way to create List/Arrays. `C` doesn't. Even as far as I know you must create a List/Array separately in `C#` and `C++` or it is more bothersome to create. But maybe it got added in those as well, I didn't track any language feature, `C++` is anyway crazy.

There also can be other reasons why you want to use `apply`. For example it happens a lot when you create wrapper functions that wraps other functions, and do something before and/or after another call. In this you often need a generic way to pass all arguments from one function to the next function call. All of that stuff becomes very is in Perl. In JavaScript it sometime a complete pain. But depending on which feature is easier in a given language, it also depends how people will use the language, or how different APIs look in different languages! Small features can have a great impact on the API design.

# My Opinion

After working with different styles I think that languages should just ditch *variable arguments* at all. Instead languages should just work with arrays/lists. Let's pick F# as an example. Sure it does support *variable arguments* because it is running on .Net and needs interoperability with other languages like C#. But this feature is restricted to the *Object-Oriented* part of the language. In the *functional* part of the language, you don't have variable arguments. For example `Math.max` becomes:

```fsharp
List.max [1;2;3;4]
```

It also somehow makes sense that the `max` function is part as a `List` function, and not part of the `Math` package if you think about it. There also could be different `max` functions for different data-structures or not?

If everything only takes list, and returns list, then the language becomes much more simpler. As always, adding more features to a language just makes a language more complex.

In F# it also allows another feature instead. *Partial application* in a language like F# is only possible because:

1. Every function only takes one argument.
2. Variable arguments are not possible.

You cannot have all features in a single programming language. Some language features are just incompatible. Did you ever questioned why a specific feature in a language exist, if it is even useful or makes other parts of the language just harder? Did you ever ask yourself if a language can get better by removing features? Instead of constantly adding one feature after another, like most languages do today.

A language is simple if it has as few features as possible. Not more or less as needed.

The problem is, everybody has a different opinion of what is needed, and what not.