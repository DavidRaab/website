---
layout:  post
title:   "Go's Error Handling"
slug:    gos-error-handling
date:    2025-12-26T00:00:00
lastmod: 2025-12-26T00:00:00
tags:    []
description:
---

Recently i tried out some Go. There are some good parts on Go that I think more
languages should do, and some bad parts. Go's Error Handling is something i consider
better than in most other langauges, bad still bad enough looking on it from a
function-programmer perspective.

So let's dig deeper. Why do I think Go's Error handling is better, let's say
compared to C. At least Go's tries to be a system language competing with
C.

So when you program in C, there are no exceptions. Something that I consider
very very good after programming for decades with languages that have exceptions.

So, how do you do error-handling in C? Mostly functions must return an error-code
that you must check. Code often end's up like this.

```c
if ( someFunction1(a) ) {
    if ( someFunction2(b) ) {
        if ( someFunction3(c) ) {
             // Code when all calls are succesfull
        }
        // When someFunction3() failed
        else {
        }
    }
    // When someFunction2() failed
    else {
    }
}
// When someFunction1() failed
else {
}
```

There are also other error-mechanism. Like using goto to jump to an error,
that maybe is better. But anyway you can see, somehow writing/testing
for errors becomes annoying.

But, this is still necessary if you want correct code and no crashing programs.

In C you also just can avoid and bypass the error-handling.

```c
someFunction1(a)
someFunction2(b)
someFunction3(c)
```

nothing forces you to check the return value of a function! And a lot of people
maybe do. Exactly as people do when they have exceptions as an error mechansim.

The result are programs that crash often, have errors because of unhandled
errors, don't behave as they should. In C this often even can mean serious
security problems that can compromise a system.

But before we continue why Go makes this a little better, how do you return
values when every function just uses it's sinle return value for error-handling?

The answer is that C programmers usually pass pointers to struct, or other
storage, and the function writes it's result into that location. I am
not so deep anymore into C. So here is a C# example that does the same.

```csharp
int  number;
bool success = int.TryParse(value, out number);
if ( success ) {
}
else {
}
```

In C it almost looks the same. You just would write `&number` to pass a pointer
to `number`, so TryParse can write to that location. So how does **Go** improve
on that? It can return multiple values at once. So in **Go** the same
code looks like this.

```go
number, err := strconv.ParseFloat(yourString, 64)
if err == nil {
} else {
}
```

It almost looks the same. But being able to return two values at once makes it
better. Different to `C` you also cannot bypassing checking `err`. Because
in Go every unused variable turns into a compile-time error. So if you
didn't check for `err` your program won't compile. It forces you to write
code that don't fail.

This is overall an improvement, but, not enough in my opinion. Functional
languages just do better. So let me explain what functional languages do, and
i show you how to implement it in Go.

Just to start again, let's see the following example.

```go
numA, err := strconv.ParseFloat("123", 64)
if err == nil {
    numB, err := strconv.ParseFloat("abc", 64)
    if err == nil {
        var added = numA + numB
        fmt.Println("%d + %d = %d", numA, numB, added)
    } else {
        fmt.Println("Could not convert number")
    }
} else {
    fmt.Println("Could not convert number")
}
```

So what does that code does? It just parses two strings and adds them when
succesfull. Otherwise it prints an error. This is actually much code for
just parsing two numbers, on the other-side it is what you must write to
have correct code.

The problem of **Go** from a functional perspective is that it separates one
information into two different variables. Actually the idea of parsing is more
like an.

1. It is succesfull and has the values
2. It fails and there is an error.

The problem of **Go** and many other languages is that they cannot express
`OR Types`. The above is one information that tells you it is either this
or that. Instead it returns two pieces of information and you must handle
them yourself.

Actually what **Go** did here is, it made the problem *complex*. Returning
just one value would be *simple*. Go also doesn't support *Discrimanted Unions*,
*Tagged Unions* and so on how `OR Types` are usually named, but we can hack it
with a struct.

```go
type Result[T any] struct {
    Ok  T
    Err error
}
```

So what this type does is exactly what error functions in **Go** return. It just
stores two values into a single structure. The problem of this type compared
to a real union is that it allows invalid state. For example `Err` could be
set to an error, while `Ok` is a valid value. Or `Ok` is an invalid error,
Nil, and so on and `Err` is still not set. But I accept this now for granted.

So with that type in place, we now can write the following `wrap` function.

```go
func wrap[T any](x T, err error) Result[T] {
    if err == nil {
        return Result[T]{ Ok:x, Err:nil }
    } else {
        var zero T
        return Result[T]{ Ok:zero, Err:err }
    }
}
```

`wrap` is intended to be used for `Go` error functions. In another language with
better Generic support I also would made the `Err` case generic instead of
always being `error`, but so far i didn't see that this was practical.

Now, what we can write is the following.

```go
num1 := wrap(strconv.ParseFloat(strA, 64))
num2 := wrap(strconv.ParseFloat(strB, 64))
```

`num1` and `num2` are now of type `Result[float64]`. This type now says the following.
It might be a `float64` or it might be an `error`.

Next i also would improve that a little bit. I just create my own function to parse
a string.

```go
func pf64(str string) Result[float64] {
    return wrap(strconv.ParseFloat(str, 64))
}
```

now i just have to write.

```go
num1 := pf64(strA)
num2 := pf64(strB)
```

But how do we work with such a type? Well, for example we could create a `ror` function.
With the meaning of *result or*. We can pass it a `Result` type and it either returns
the value when there was no error. Or it just returns the passed default value.

```go
func ror[A any](res Result[A], or A) A {
    if ( res.Err == nil ) {
        return res.Ok
    }
    return or
}
```

Now, we can write the following.

```go
num1  := pf64(strA)  // Result[float64]
num2  := pf64(strB)
added := ror(num1, 0) + ror(num2, 0)  // float64
```

Here `added` is now just of type `float64`. In that example the code is even so
short, i would just inline the whole parsing.

```go
added := ror(pf64(strA), 0) + ror(pf64(strB), 0) // float64
```

so this one single line does parsing two string. And in the case one number
fails parsing, it just picks `0` as the default value. And it still has the
benefits that all error cases are handled. Not a single `err` was unchecked.
This code, compared to throwing exceptions will also never fail.

But usually `ror` is not the only function we create in functional programming.
Let's say you have a function that justs adds 100 to a float.

```go
func add100(num float64) float64 {
    return num + 100
}
```

then we have one problem. We cannot directly use that function with a function
returning a `Result[float64]`. But i guess no programmer has a problem to just
write code that accesses the `Result.Err` case and write an easy `if`
statement and either call `add100` or not. Right? We just need to put it into
a function so that code becomes a reusable *design-pattern* (How OO people
will tell you, but it is wrong, a design-pattern in OO is a piece of code
you copy & paste over and over because your language lacks abstraction). In
functional programming we name this a `map` function. A type that supports
`map` is named a **Functor**. Not that hard?

```go
func rmap[A any, B any](result Result[A], f func(A)B) Result[B] {
    if result.Err == nil {
        return Result[B]{ Ok:f(result.Ok), Err:nil }
    } else {
        var zero B
        return Result[B]{ Ok:zero, Err:result.Err }
    }
}
```

Now what we can do is the following.

```go
num1 := rmap(pf64("100"), add100)
num2 := rmap(pf64("abc"), add100)
num3 := rmap(pf64("100"), func(x float64) float64 { return x + 100 })
```

So here you see how to parse strings and pass a function or a lambda
to do something when parsing was correct. In all cases you still get an
`Result[float64]` not a `float64`.

`num1` will contain `200`, `num2` contains an error because parsing was wrong,
and `num3` will also contain `200`.

The idea that a `Result` value is not directly extracted allows you to write
code like this.

```go
x := failingFunc1(whatever1)
y := rmap(x, func2)
z := rmap(y, func3)
```

and so on. So we can just work with the value. It's like the advantage of
code that throws exceptions. You just see the *happy path*. Only at the very end
you check if there was an error. We for example could create another function
that helps us to handle both cases.

```go
func rmatch[A any, B any](result Result[A], fok func(A)B, ferr func(error)B) B {
    if result.Err == nil {
        return fok(result.Ok)
    } else {
        return ferr(result.Err)
    }
}
```

with this we now can write at the end.

```go
x := failingFunc1(whatever1)
y := rmap(x, func2)
z := rmap(y, func3)

msg := rmatch(z,
    func(num float64) string { return strconv.FormatFloat(num, 'f', -1, 64) },
    func(err   error) string { return err.Error() })
```

`rmatch` expects two lambdas. The first one handles the case if the values was
`Ok` and the second handles the case if we had an `Err`.

The only problem of `Go` is it's lack of type-inference. Always writing
all the types in a lambda becomes pretty annoying. But still, in my opinion
this style of programming is even in `Go` much better than what you get by
default in `Go`.

You probably will add other functions. For example `map2` that can handle two
`Result` values.

```go
func rmap2[A any, B any, C any](res1 Result[A], res2 Result[B], f func(A,B)C) Result[C] {
    if res1.Err == nil && res2.Err == nil {
        return Result[C]{ Ok:f(res1.Ok, res2.Ok), Err:nil }
    } else {
        var zero C
        if ( res1.Err != nil ) {
            return Result[C]{ Ok:zero, Err:res1.Err }
        } else {
            return Result[C]{ Ok:zero, Err:res2.Err }
        }
    }
}
```

Consider how this `map2` is similar to the code we started with.

```go
numA, err := strconv.ParseFloat("123", 64)
if err == nil {
    numB, err := strconv.ParseFloat("abc", 64)
    if err == nil {
        var added = numA + numB
        fmt.Println("%d + %d = %d", numA, numB, added)
    } else {
        fmt.Println("Could not convert number")
    }
} else {
    fmt.Println("Could not convert number")
}
```

and it actually is! What we have done there was basically extracting/checking for
two values and executed some code when both cases was successful. With `map2`
we just put the whole stuff into a function so we don't have to re-write this
piece of shit a thousand times.

Here is another function. A function that trasnform the `Ok` value into a
`string` or just uses the `error` as a string.

```go
func rstring[A any](result Result[A], fok func(A)string) string {
    return rmatch(result,
        func(x       A) string { return fok(x)      },
        func(err error) string { return err.Error() })
}
```

finally, we can write.

```go
numA  := pf64("123")
numB  := pf64("abc")
added := rmap2(numA, numB, func(x float64, y float64) float64 { return x + y })
msg   := rstring(added,    func(num float64)string            { return strconv.FormatFloat(num, 'f', -1, 64) })
fmt.Println(msg)
```

This code will now print `strconv.ParseFloat: parsing "abc": invalid syntax`.

I guess because of the annoying lambda syntax in `Go` you maybe want to
extract the lambdas into it's own functions, so in the end you maybe end
up with code looking like this.

```go
numA  := pf64("123")
numB  := pf64("abc")
added := rmap2(numA, numB, add)
msg   := rstring(added, float_to_string)
fmt.Println(msg)
```

So this code does everything we want. Parse two strings. Still containing the
iformation if it was sucessfull or not. `rmap2` just adds those two numbers
together, but only when parsing of both were sucessfull. Then with `rstring`
we transform the result into a string being able to printing, or we
get the error message. Again everything also could be put
into a single line.

```go
msg := rstring(rmap2(pf64("123"), pf64("abc"), add), float_to_string)
fmt.Println(msg)
```

because there is no real reasons why we need those variables. But this can
be subjective if you are used to reading/writing functional code or not.

How good could `Go` be if they also had choosen to add something like
*Discrimanted Unions* and add such a Result-type instead of returning two values?

It maybe would be one of my favorite languages.

Another thing i dislike. For whatever reason Generics are not allowed when you
use that *OO emulation mode*, that's how i call it. Where it looks like you
call a method on a struct. Because then `Go` complains about that generics
are not allowed. An absolute pain. Because then we also could have chaining syntax
like

```go
numa := pf64(str).map(add100)
numb := rmap2(numa, pf64("abc"), add).rstring(float_to_string)
```

but i guess generics on methods are still to advanced for `Go`, maybe it takes
another 20 years until they add this feature.
