---
layout: post
title: "What is Dependency injection?"
slug: what-is-dependency-injection
date: 2023-01-11
tags: [patterns,dependency-injection]
description: ""
keywords: patterns, dependency injection
---

[On Reddit i came to a question](https://www.reddit.com/r/csharp/comments/108unij/quick_question_dependency_injection_is_just/) about someone who had problems understanding
Dependency Injection. He provided the following code.

```csharp
public class Wheel {
    public Wheel() {}
    public void moveleft() {}
}

public class Car {
    private Wheel wheel;

    public Car (Wheel wheel) {
        this.wheel = wheel;
    }
}
```

The question is: Is this usage of `Wheel` in `Car` **Dependency injection**?
But the person wasn't sure, as a **Is it that simple or I misunderstand something?** followed.

First, the answer is **No**. It's not dependency injection. And still a lot of people
saying **yes** believing it would be. This is one of the reason the author is being confused.
Because if this would be **Dependency injection** as many people believe, then the
only classes without **Dependency injection** would be those with an empty constructor.
No wonder the following **Is it that simple?** is asked. Because then **Dependency injection**
would have no meaning at all.

I guess the whole misunderstanding of **Dependendy injection** happens because
a lot of explanation (I see this often with design patterns) are just extremely bad.
They usually don't start with a problem and just explain a pattern out of the blue.

But this is not how Design Patterns, or any kind of a good explanation should work
in general. Design Pattern are called **Design Patterns** because the are solution
to common problems.

And sure, just passing some arguments to a class is obviously not a problem at
all. That's what basically every language offers. Otherwise programming with
objects would be completely pointless.

To understanding any **Design Pattern** we first must understand what kind of
problem a **Design Pattern** tries to solve. So we must understand the
problem first.

And this is what i'm doing now. I introduce you to a problem that is common
to software development.

# The problem

Consider you are writing an application. The application should have a common
configuration file. In your first attempt you just consider to read
a JSON file. So you create a `Config` class, that just does that. Reads the
config, does the parsing and offers you an interface to query your config.

How this interface exactly looks and how it is implemented doesn't matter
in this article.

Your program get bigger and suddenly you decide to support another configuration
file. Maybe you create your own format, or you use XML. Maybe you want to
read the config from a database. Or maybe you are writing a test-suite and
need different configurations in your test-suite, so it would be nice to define
the config just as a `string`.

The question that arise: How do you solve this problem? How can you provide this
flexibility without changing all of the classes that depends on `Config`?

The answer is with **Dependency injection**. **Dependency injection** is that
you have a **dependency** in this case for a Configugration file. But you want
multiple different ways to configure your application. So instead of
a single `Config` class you will have many different **Config** classes like
`ConfigJSON`, `ConfigXML`, `ConfigDatabase`, `ConfigString`, ...

Other classes that you write could now directly instantiate `ConfigJSON` or `ConfigString`
and so on. But this would have disadvantages. The short is, you usually
don't want to pick a concrete implementation if there exists more than one,
actually your application anyway wouldn't work this way.

<div class="info">
The <strong>more than one</strong> actually is really important. Today a lot of (at least
OO) programmers think it's in general a bad idea to instantiate a specific class.
This often leads to code where every class has an interface, wether or not
there exists multiple implementations. In my opinion an extremely bad
practice that is common today.
</div>

At the early stage when your program starts, you usually want to pick
one of those, and then use this throughout the whole program. For example
you program could check if a file like `config.json` exists, if so then
it uses `ConfigJSON`, otherwise it would test for `config.xml` and use
`ConfigXML`, ...

Once you have decided which config to use you would pass this configuration
to all of your classes. One way to do so is by **Dependency injection**. **Dependency injection**
means by either providing the dependency when you create the class (**Constructor injection**)
or by passing it later through a setter (**Method/Property injection**).

```csharp
var foo = new Foo(config=new ConfigJson("file")); // Constructor injection

var foo = new Foo();
foo.setConfig(new ConfigJson("file")); // Property/Method injection
```

And this is also why people get confused what **Dependency injection** really is.
When you are never introduced to the problem first then there is no difference
to any call to a constructor of a class or any other setter.

No wonder why there is so much confusion around this!

**Dependency injection** tries to solve the problem that rises when you have
one dependency, like a config, but many different ways how to create such an
dependency.

When your application uses only one way to configure your application, and
you never feel the urge to do it any different, then you would not have a
problem to begin with. You still can pass the config as an argument as passing
variables always makes things more flexible but you will never run into a problem
with this.

One reason where people feel the urge to use this pattern, and probably the most
common, is when your class is doing side-effects. Exactly what happens when you
read a config file. That's also why I picked this example.

Without side-effects and multiple different implementations there usually
is no need for this pattern as you don't have a problem to begin with.

# Static Typing

The only thing we need to consider is **Static Typing**. In a dynamic typed
language there would be no problem to create different classes like `ConfigJson`
and so on, and use them wherever you want. As long all those classes share the same
interface it will work.

But in a **static typed** language you must add some glue that this is possible.

You either must:

1. Create an abstract base class `Config` with your common interface and expect
   every implementation to derive from `Config`.
2. Create an interface like `IConfig` and expect `IConfig` everywhere you would
   expect to get a config.

This way it is possible to write.

```csharp
var config = new ConfigJson("config.json");
var config = new ConfigXML("config.xml");
```

and later use any of the `config` variable.

```csharp
var other = new Other(config=config)
```

It doesn't matter if your `config` will be `ConfigJson` or `ConfigXML`. And all
of this is **Dependency injection**.

Whenever you have just a *normal* (whatever this means) class with just some fields,
than it doesn't become **Dependency injection** just because you constructor expects
some variables.

```csharp
public class Person {
    public string Name;
    public Person(string name) {
        this.name = name;
    }
}
```

If we would say `string` would be a dependency to `Person` and it would be
**Dependency injection** then we could just forget everything about the Pattern.
The pattern would otherwise be so much simplified that it became complex. As
any meaning to the Pattern was lost. It would have no purpose to talk about
**Dependency injection**.

# Conclusion

A pattern always solves a problem and the code to implement it doesn't stand
for itself. Just because some other code basically looks the same doesn't
mean it is this pattern.

Ignoring the problem just causes confusion to understand patterns in general.
Sadly this is exactly what i see in most OO tutorials that explain design
patterns.

I also must say i am generally against most of the design patterns that people
think are usefull, they often have the exact same problem you see here. Some
(not all) are so identical that it becomes hard to distinguish them. Some
patterns are basically the same with different names (Strategy & Command pattern
as an example).

On top, there are also other ways how to solve the problem. You can solve the problem
of different configurations from JSON, XML or an in-memory string with a single `Config`
class, without inheritance and without using interfaces at all.

But this is another story ...
