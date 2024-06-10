---
layout:  post
title:   "OOP is Partial Application"
slug:    oop-is-partial-application
date:    2024-06-10T00:00:00
lastmod: 2024-06-10T00:00:00
tags:    [design,fp,oop]
description:
---

In the past I spent a lot of time in different online forums, websites and so
on. As I am interested in general about language design and learnt a lot of
languages I also had many discussions with different programmers of different
knowledge.

As I started programming with QBasic and C I am also used to procedural
programming. The same goes for Perl. As of Perl 5 (1994) it also contains
object-oriented programming, but still some parts of it are procedural.

Nowadays there are a lot of people usually telling you that procedural
programming is somehow bad and was replaced by object-oriented programming. Most
of the time its by people who never worked, used or learned a procedural
programming language. But they sure know all about it.

So one discussion I had was the following. Someone explained to me why
procedural programming is bad. He explained it this way to me:

Let's consider you wanna design a `Person`. Let's say a person has a first
name, last name, an age and so on. In a procedural language, he explained to me,
you always must pass all those values to a function separated. So
you end up having functions like

```
C func1(string firstName, string lastname, Date birthday, ...)
```

If you do a lot of those functions, then this is quite annoying. Well, I think
I agree to that. So then he continued to explain to me that object-orientation
came to the rescue. Instead of always passing all those values to functions
now you can do a `Person` class.

```csharp
public class Person {
    public string firstName;
    public string lastName;
    public DateTime birthday;

    /* some addition constructors */

    C func1(...) {
        ...
    }
}
```

And this of course is better than those procedural programming.

Well, I would agree to that if procedural programming would really like that,
and you only have this limited knowledge of it. But the truth looks a little bit
different.

Even an old language like C already knows about creating custom data-types. Sure
it would be annoying if you couldn't create something like a `Person`, but
actually you can. In C that would be a [Struct][Struct] data-type. So your
person could look like this in C.

```c
struct Person {
    char* firstName;
    char* lastName;
    time_t birthday;
}
```

once you have defined such a structure, you also can create `Person` structs
in C and pass such whole structures to a function. Your functions then turns
into something like:

```c
C func1(struct Person)
```

Let's say you have person struct as a variable. Do you wanna know how you can
access those different members? It will look like this in C.

```c
char* firstName = person.firstName;
char* lastName  = person.lastName;
```

Do you feel comfortable with this kind of syntax? Do you maybe have seen
something similar like this as a *object-oriented* programmer?

The only real difference to a language like C# is that C differs between
a `struct` and a *pointer* to a struct. A `struct` in C is always a value type
and gets copied as a whole.

So if you call `func1` like above then `func1` gets a copy. But usually in C
you don't wanna pass copies, you wanna pass a pointer that points to the
original, so inside of `func1` you can mutate/change the content of a `Person`
struct instead of working on copies.

As a C# developer you should be familar with this. C# offers *structs* the same
as C does. And a pointer to a *struct* is named a `Class` or also refered to
being a *reference-type* instead of being a *value type*.

This is also how you can do [Object-Oriented Programming in C][OOPC].

The only real difference is a little bit of syntax. Instead of a typical `obj.func1(...)`
you basically do `func1(obj, ...)`. This conversion is also what [Perl does][PerlOO]
or many other languages work *under the hood*. In Perl or Python passing the object
is usually explicit. That's why methods usually start with `$self` as the
first argument. In C# this is usually hidden from you, but still is done. You can
access the object with the special `this` or you even can omit `this.` if you want
to access member variables. This makes it look special in an *object-oriented*
language but still is just nothing more than a little bit of syntatic sugar.

Now let's talk about *Partial Application* in a functional language. Let's
say I have the following function in F#.

```fsharp
let func1 firstName lastName birthday x y z =
    ...
```

this is a definition of a function in F# taking six arguments. One *special feature*
(it isn't so epcial as you might think) is that you don't need to pass all arguments
to a function call in F#. For example you just could do.

```fsharp
let func2 = func1 "foo" "bar" (DateTime.Now ())
```

but this doesn't execute `func1`. Because we only passed three arguments we get
a new function back. This new function `func2` still expects the arguments `x`,
`y` and `z`.

This is an interesting aspect, because if you think about it, then *Object-orientation*
actually does nothing more than exactly the same, but over multiple methods.
Let's consider you have a class like.

```csharp
public class Whatever {
    public A a;
    public B b;
    public C c;

    /* Constructor */

    void method1(D d, E e);
    void method2(E e, F f, G g);
}
```

when you create an object from that class

```csharp
var whatever = new Whatever(new A(), new B(), new C())
```

and you later call the methods on that object.

```csharp
whatever.method1(d, e);
whatever.method2(d, f, g);
```

then actually it is the same as if you would have **static methods** and
call your functions like this.

```csharp
Whatever.func1(a, b, c, d, e);
Whatever.func2(a, b, c, d, f, g);
```

If you think about it, you always can turn any class with methods into a bunch
of *static methods* that just receive every member you passed to the constructor
directly to the function call. There is basically no difference.

But sure, always passing `a, b, c` to `func1` and `func2` would be annoying.
If the combination of `a, b, c` only makes sense together then always passing
those three values is annoying. But you always can create a new type/class and
puting those three values together.

This *putting together* and combining data into a new type is nothing special
you only see in an object-oriented language. Even a very old language like C
supports it. Functional language like F# even have better options to create
data-types that are more advanced then you will find in a typical object-oriented
language. The concept there is named [ADT - Algebraic Data Types][ADT]

So yes instead of.

```fsharp
let obj1Method1 = func1 "foo" "bar" (DateTime.Now ())
let obj1Method2 = func2 "foo" "bar" (DateTime.Now ())
```

to create two functions that somehow can be seen as methods operating on the same
data. You maybe wanna create a data-type first too in a functional language.

```fsharp
type Person = {
    FirstName : string
    LastName  : string
    Birthday  : DateTime
}
```

then you can create those compound data types and use it with your functions.

```fsharp
let obj1 = { FirstName = "foo"; LastName = "bar"; Birthday = (DateTime.Now ()) }

let ret1 = func1 obj1 x y z
let ret2 = func2 obj2 x y z
```

Isn't it interesting how similar all of this is? A class with a bunch of methods
always can be turned into a bunch of *static methods* receiving all of its
arguments that you pass to the constructor at once.

This is important in the sense when you only have one-method classes. Because
if you have, just turn that into a single function. Having this kind of stuff
is just crap.

```csharp
var obj1 = new Whatever(a, b, c)
obj1.method1(d, e, f)
```

just turn it into

```csharp
Whatever.func1(a, b, c, d, e, f)
```

there is no point in doing it in the OO way. If somehow it makes sense that `a, b, c`
is some kind of unit that always must be passed together. Then create a new type
out of `a`, `b` and `c`.

```csharp
var abc = new ABC(a, b, c)

Whatever.func1(abc, d, e, f)
Whatever.func2(abc, e, f, g)
```

In a functional language like F# there exists an operator like `|>`. That operator
actually isn't so special but widly used in F#. Here is the definition of it.

```fsharp
let (|>) x f = f x
```

So `|>` is basically just a function expecting two arguments. First an `x` and
a `f`. And what if does is just `f x`. Or in other words. It considers `f` a
*function* that is just called and passes it `x` as a value. It just revers
the typical order. so instead of

```fsharp
let r = func1 x
```

to call `func1` with `x` you also can do.

```fsharp
let r = x |> func1
```

Because `|>` is called as an operator whatever is on the left-side is considered
as the first argument to `|>` and whatever is on the right-side is considered
the second argument of `|>`.

Because of this operator and how a functional language is used, we also do
another order of the arguments. Instead of defining.

```fsharp
let func1 obj a b c =
    ...
```

we also put the `obj` as the last argument, not the first.

```fsharp
let func1 a b c obj =
    ...
```

now you can call it in two different ways

```fsharp
let r = func1 a b c obj
let r = obj |> func1 a b c
```

does the second call seems not familiar to you how it looks in a language like C#?

```csharp
var r = obj.func1(a, b, c)
```

You also can chain multiple `|>` calls.

```fsharp
let result =
    x
    |> func1 a b c
    |> func2 d e f
    |> func3 g
    |> func4 h i
```

this is a style you will see a lot in F#. Actually it is the same as

```fsharp
let result = (func4 h i (func3 g (func2 d e f (func1 a b c x))))
```

so instead of nesting the functions calls it allows you to chain them.
This is exactly how you maybe are used to it in an *object-oriented* language.
There it is often called *fluid syntax*.

```csharp
var result =
    x
    .func1(a, b, c)
    .func2(d, e, f)
    .func3(g)
    .func4(h, i)
```

so while being very similar. The way of how `F#` does it has a big advantage.

Because in an object-oriented language that `func1`, `func2`, ... always
must be part of the class itself. But if that class is *sealed*, *final*
or you cannot *inherit* it, you also cannot extend it by new functionality.

On the other hand doing it the F# way allows you to always just write your
own function. You can put that function wherever you like. Just create a
locally scoped function, create your own module. It doesn't matter where
that function is put, you always can create a chain from it.

```fsharp
let result =
    x
    |> List.map ...
    |> Lst.map2
    |> localFunc
    |> Whatever.listOp
```

To achieve the same in a language like C# you actually have to create an interface.
Use Extension methods or reach to default method interfaces, otherwise you cannot
really come up with this chaining. Nothing of that is needed in F#. You also don't
run into naming conflicts of your *methods* and can use the same function names
on different modules. A far superior way of programming and you even need to learn
and understand less.

Or you don't use the chaining, the nested way also isn't that bad if you are
used to it, and sometimes is far better understandable than chaining. Especially
if your code is not a chain of imperative steps of `doA` then `doB` then `doC`.

<div class="info">
In <a href="{{< ref 2016-09-25-function-application-and-composition >}}">Function Application and Composition</a>
I discuss this topic in more depth and give some thoughts about readability
of those different styles.
</div>

For example


```fsharp
listA |> List.append listB
```

does not append `listB` onto `listA`. It does the opposite. If you would call

```fsharp
let r = List.append [1;2;3] [4;5;6]
```

then it returns `[1..6]` what you maybe also expect. Calling it

```fsharp
let r = [4;5;6] |> List.append [1;2;3]
```

is the same function call and also returns `[1..6]`.

So in this post I hope to bring you closer and showed you how procedural, functional
and even object-oriented programming are more similar than you maybe expect. Those
language usually have different syntax and might look different but the concepts
are pretty much the same.

The big problem I see is that doing it the object-oriented way actually has led
a lot of programming into a bad corner. [Those encapsulation or that methods
have to be part of the class actually has not improved programming much][Anemic], I think
it made programming much more worse as it should be.

[Struct]: https://www.w3schools.com/c/c_structs.php
[OOPC]: {{< ref 2023-11-01-object-oriented-programming-in-c.md >}}
[PerlOO]: {{< ref 2024-04-15-why-i-like-perls-oo.md >}}
[ADT]: {{< ref 2016-04-26-algebraic-data-types.md >}}
[Anemic]: {{< ref 2024-06-04-anemic-domain-model-data-and-behaviour-should-be-separated.md >}}
