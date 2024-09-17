---
layout:  post
title:   "Data vs. Interfaces"
slug:    data-vs-interfaces
date:    2024-09-17T00:00:00
lastmod: 2024-09-17T00:00:00
tags:    [data,interfaces,oop,fp]
description:
---

# Intro

When we face a problem, then we must admit one thing, there are always different
kind of approaches how we can solve something. And on top of that, different kind of
approaches have different kind of advantages and disadvantages.

Whatever you do, you will never find a *perfect* solution in the sense that this
solution is best in everything.

Usually every advantage has its opposite disadvantage. For example for solving
a problem there are usually different algorithm out there. Let's say we
wanna sort an array, we could pick Quick-Sort as usually this is the fastest
sorting algorithm.

But it comes at a price. Quick-Sort can be very hard to implement and if wronlgy
implemented, even can be one of the slowest algorithm. It also has another problem
that sorting something by Quick-Sort is not Stable.

Merge-Sort don't have this problem. It's easier to implement, always has a good
performance and is stable. But in practice will be a little bit slower than
Quick-Sort.

This *problem* is not a computer only problem. You will find this kind of advantages
and disadvantages basically everywhere. As there is no *perfect* solution it leads
to the question on why you pick a certain solution and did you every consider
its *advantages* and *disadvantages* or do you just pick something because
**the majority** or a **\<insert famous person here\>** does so?

# What is the problem?

*interfaces* are usually a solution to a problem that is picked primarily in
object-orientet programming. But what actually is the problem?

I would wish that I could give you a good answer, but I can't at the moment,
so let's explore two different solutions to the same problem, maybe we get an
answer.

# The problem

Let's consider the following problem that a lot of programs have. Your program
offers some kind of configuration file. Your program should be able to
read and to store/update the configuration file. Or maybe just read it?

In an object-orientation solution you maybe come up with the idea of a `Config`
class. That `Config` class basically just offers you two methods. A `get`
method for reading a key. And a `set` method for setting a key.

Now we could just implement the `Config` directly, without any interface. Somewhere
in our program, usually at program-startup, we create a `Config` object, and
then we pass that `Config` object to everything that needs to read/write
the config.

Object-oriented programmers say that this is *dependency-injection*. A functional
programmer says that this is a *State Monad* or *Reader Monad* (if you just can
read from the config).

But solving the problem in this way can have some disadvantages. Consider that
we have some poorly implemented `Config` and whenever you call `get` or `set` it
will always read and write the config-file directly. So a lot of IO operations
happens instead of just reading the file once.

Or maybe is that an advantage? I mean whenever the config-file is changed even while
the program is running, the running program will always get the latest current
value?

But usually we made the assumption that a config-file does not change so often
that it matters, and even when it does, then we expect that the program just
must be notified for re-reading or even just be restarted. Here you can see
how we accept some disadvnatages and favor the advantages for better performance.

But still doing this has another disadvantage. How about we want to support
multiple config-files? This is usually the reason why it begins that people
tell you why you should do an interface out of it. By extracting the interface

```fsharp
type IConfig =
    abstract member Get : string          -> string // let name = obj.Get("name")
    abstract member Set : (string,string) -> unit   // obj.Set ("name","hello")
```

now with such an interface you are able to create an `ConfigJSON`, `ConfigXML`,
`ConfigINI` and so on.

While this sometimes is a valid reason, I don't think it is always a valid
reason. For example I absolutely see zero reasons why a program should support
multiple config-files. Except it maybe invented its own config file and you
need compatibility with the old one. As you can see there are reasons for that.

One thing that object-oriented programming leads to is, that programmers in
general, because of that, always starts to create interfaces, even when they
are never needed. But they still do because **maybe** it will be needed.

But there is still one other reason why you always want to create an interface.

## Side-effects

The biggest problem we face here is that `Config` and all of the other `Config*`
classes share is that all of them do side-effects. That means they need to read
or update/save a file on disk.

And all kind of side-effects have one big problem. They make testing your code
harder. Consider you want to test your `Config` classes or in general your
application with different kind of configs. Then for every config you have
you must create a config-file on disk.

What happens if your code changes a config-file? I mean it was a supported feature
right? Then you must create a test-system that maybe when it starts copies a bunch
of config files from one folder to another, do your testing, then maybe delete it
again.

It's annoying in multiple ways and makes some stuff harder. But hey, we have an
interface so why not create just an `ConfigInMemory` class?

That `ConfigInMemory` has the same interface, but it just holds all config data
in memory. It never reads and saves to a file. It's basically free of side-effects.

Such a class is also good for testing, because just in code we can create a dozens
of `ConfigInMemory` objects and just pass them around. We don't need to create
a dozens of files anymore or do some kind of test-setup anymore.

So *interfaces* are somehow awesome right? But, can we improve?

# Data approach

Yes, we can. So here is how I see the problem. First, when I see an interface
that just provides me a `Get` and `Set` method all I see is just a `Hash`,
`Dictionary` or however they are named in your favorite language.

A `ConfigInMemory` is basically just that. A data-structure holding data
just in memory. So here is another approach: Why instead of expecting an `IConfig`
interface with a `Get` and `Set` method, why do you not just expect a `Dictionary`
instead?

So instead of

```fsharp
let config = ConfigInMemory()
config.set(x,y) // multiple of those lines
obj.method(config, a, b)
```

you just do

```fsharp
let config = dict [
    x, y // multiple of those lines
]
obj.method(config, a, b)
```

I am used to Perl programming, and in such a dynamic typed language I would
usually expect to pass a hash.

```perl
my $config = {
    name   => "whatever",
    width  => 1200,
    height => 600,
}

$obj->method($config, $a, $b);
```

So, when you expect just a data-containe, how can we now read and write to
from a file?

Easy, how about just creating functions (static methods) that are able to read
and write from or to a file?

Could look like this

```fsharp
let config = Config.fromFileJSON("config/app.json")
let config = Config.fromFileXML ("config/app.xml")
let config = Config.fromFileINI ("config/app.ini")

Config.saveFileJSON(config, "config/app.json")
Config.saveFileXML (config, "config/app.xml")
Config.saveFileINI (config, "config/app.init")
```

So instead of expecting an interface with different kind of operation you
just expect the data-itself. By just accepting the data you can create different
functions that are able ro create your data. But once they are data, it doesn't
matter anymore if those data was read from a JSON, XML or INI file.

Because our `config` is just a dictionary, you anyway just can create them in-memory
and don't need to worry about side-effects.

This approach works great in a *dynamic-typed* language, because usually
every value of a key can be a different type. You can pick strings, numbers,
or even again more complex types like, arrays, hashes again.

In a *static-typed* language you maybe want todo a little change. Instead of
a *dictionary* you maybe want to expect a *type* instead. For example when
I expect to configure the title, width and size through a config file than
I create a type like.

```fsharp
type Config = {
    Title:  string
    Width:  int
    height: int
}
```

here is one interesting aspect. Because I am expecting data, I never need
interfaces. It doesn't even make sense to have *interfaces* for data.

Yes, you maybe want to read from different kind of config files, but I don't
need interfaces for this.

When you stick to a *Data* approach than all what you do is you basically start
to see your *side-effect-free-code* as your default approach.

I have seen a lot of bad programmers that somehow come up with stupid crap that
static methods can not be replaced. Seriously, somewhere in your code you
need some kind of dispatch that decides which function must be called.

And it doesn't matter if your code either calls a concrete static method (function)

```fsharp
let config = Config.readFromJSON("app.json")
```

or it does call a concrete class constructor

```fsharp
let config = ConfigJSON("app.json")
```

calling a class constructor is no kind of difference between calling **any**
other static method. You can even consider `ConfigJSON` just as a global
static method.

And somewhere you have todo that dispatch. Otherwise you cannot read your config.

You could for example create a `Config` class that automatically checks which
config file format is available. And when multiple are available which one
should be picked that than depending on this information either initializes you
a `ConfigINI` or `ConfigJSON` but just returns you an `IConfig`.

The same as you can create a `Config.read` function that does exactly the same
kind of logic and just returns you the above defined `Config` type.

You also can have a more complex config. Let's say I have a program that draws
lines and want the lines it draws to be configured. I could do.

```fsharp
type Line = {
    Start: Vector2
    Stop:  Vector2
}

type Config = {
    Title:  string
    Width:  int
    Height: int
    Lines:  Line array
}
```

I could create this data-structure in Memory like this.

```fsharp
let config = {
    Title  = "Line Drawing"
    Width  = 1200
    Height = 600
    Lines  = [
        { Start = vec2 -1f 0f; Stop = vec2  0f 1f }
        { Start = vec2  0f 1f; Stop = vec2  1f 0f }
        { Start = vec2  1f 0f; Stop = vec2 -1f 0f }
    ]
}
```

how you exactly map that to JSON, XML or INI is another topic, but you
could for example pick the following JSON representation.

```js
{
    title:  "Line Drawing",
    width:  1200,
    height:  600,
    lines: [
        -1,0 ,  0,1,
         0,1 ,  1,0,
         1,0 , -1,0
    ]
}
```

or choose a dozens of other ways how to represent a line. Put for values
in an array, also expect an js object and so on. How you map that exactly isn't
the point of this article, the important aspect is the following.

Whenever you interact with the *outside-world* relative to your application.
That means usually doing side-effect by reading data from input devices,
files, sockets, data-bases and so on. You always have one single-representation
of whatever you need as data-format.

Once you have that data-format you can create as much reader or writer to
and from that data-format. But your application only uses that data-format,
and your data is passed around.

This way you eleminate the need for interfaces and avoid side-effects. You get
to a state where your main application logic is just a series of functions
transforming just data.

At the edges (outer shell) of your program your program actually follows the
following principle.

1. Read stuff from external stuff, and create your data (IO)
2. Do stuff on Data (pure) usually creating new data.
3. Do something with new data, for example, print, write (IO)

This kind of working is a common *pattern* in functional programming that has
given the name [functional core, imperative shell][fcis], but at some
degree this is also just called *procedural programming*.

# Verdict

So what kind of problem did we try to solve with *interfaces*?

We had the problem that our code actually needed different implementation, like
reading from JSON, XML or JSON. And the other problem is that side-effects
in a system causes problem.

So when you stick to just data. Then.

1. Data never has side-effects.
2. When you separate data from behaviour then it is up to you to create as
   much functions that operate on that data as you wish.

The same approach I have used here in F# also work in C# or any other
object-oriented language. What you just need todo is just create a `Config`
class that besides having public fields or simple getter/setter does absolutely
zero additional things. Usually from bad programmers called an *Anemic Domain Model*
considered as purely evil.

In the approach above you just would create a simple `Line` class holding your
start and stop point. The problem why purely object-oriented programmers don't
come up to this kind of solutions is because they believe in complex things like
an object that holds data and code instead of making it simple by separating
those two things from each other.

So separating your behaviour into an *interface* just to provide different
implementations is just a poor way of solving the problem of having different
similar, but still different, solutions.

Back to sorting. When I want to sort an Array I create `quick_sort`, `merge_sort`
and `insert_sort` functions that expects an `array` as its data. I don't come up
with the idea to create an `ISortableArray` that has a `Sort` method and three
different classes `QuickSortArray`, `MergeSortArray` and `InsertionSortArray`
just to implement three different sorting algorithm.

When you write code and for example you want to `sort` something but want to give
the user the freedom to choose which `sort` method should be choosen. Then
you expect that this function is passed as an argument. See my previous
article about [interfaces in functional programming]({{< ref 2024-09-04-interfaces-in-functional-programming >}})
about this topic.

# Relative Topics

* [interfaces in functional programming]({{< ref 2024-09-04-interfaces-in-functional-programming >}})
* [Anemic Domain Model]({{< ref 2024-06-04-anemic-domain-model-data-and-behaviour-should-be-separated >}})
* [functional core, imperative shell][fcis]
* [Rich Hickey - Simple Made Easy](https://www.youtube.com/watch?v=SxdOUGdseq4)
* [Rich Hickey - The Value of Values](https://www.youtube.com/watch?v=-I-VpPMzG7c)

[fcis]:https://github.com/kbilsted/Functional-core-imperative-shell/blob/master/README.md
