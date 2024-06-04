---
layout:  post
title:   "Anemic Domain Model: Data and Behaviour should be separated"
slug:    anemic-domain-model-data-and-behaviour-should-be-separated
date:    2024-06-04T00:00:00
lastmod: 2024-06-04T00:00:00
tags:    [design]
description:
---

In the object-oriented world there is often a debate around the so called
*Anemic Domain Model*. An *Anemic Domain Model* is when you clearly separate
your data and your code.

In object-orientation this is usually considered bad, but when you look at
*procedural* or *functional-programming* things change. They are seen as
good practice you should follow. So who is right?

# Rich Domain Model

The opposite of an Anemic Domain Model is usually called a *Rich Domain Model*
the idea of it goes as follow.

Usually all begins with data. So let's say we have an application to manage
some users. Could be for a webapplication like Wikipedia or any other web
service your are using. Also could be for any other service like an organization
that keeps track of it members. Or maybe just your Contact book in your Smartphone
that keeps track of its user, telephone numbers and addresses. Details
aren't so much important at the moment. So we just say we keep track of some
user data.

In a *Rich Domain Model* as you are used by *object-orientation* you usually
will start with a `User` class. Then you will add all kinds of *methods* to
that class for whatever stuff you wanna do with a `User`. This usually also
includes *validation* for additional constraints on data that are not
representable by its data. For example restriction an integer to a certain range
like -100 to 100, or maybe restriction a string that is not allowed to start or
end with whitespace. There are usually a dozens of such restriction you usally
can come up with.

We keep it *simple* (not really) so let's say we just have an User that
stores an unique id, a login name, a full name and the birthday of the user.

Then you go on and add all kinds of validation to the constructor of your class.
You decide unique id can never be smaller than 0, a username cannot contain
any whitespace at all, and so on.

You also implement the getter/setter of your class to only allow valid values.

Usually in most languages objects also have the idea of equality and comparison.
So you also start to implement equality so you know if two objects refer to
the same user or not. Sure you wanna comparison as when you show all users
they should be ordered.

This is called a *Rich Domain Model* and object-oriented programmer telling you
it is the only way you should do it. Everything else is bad.

# What's wrong with a Rich Domain Model

So what is the problem with the above? Seems okay, you don't wanna have invalid
data and it is cool to have all kind of abilities so you don't need to write them
yourself, right?

So let me ask you some question.

What do you do, if you wanna save a User to a relational database? Do you add a
`store_to_database` method to your `User` class? Usually that is a question
that most people will answer with *No*. Most *object-oriented* programmers
at least have heard about the problem of *dependencies*, they don't wanna add
a dependencie that a `User` somehow needs to know about *databases*. But hey,
do you see how this contradicts the idea of a *Rich Domain Model*?

And anyway, what kind of database? MySQL, MariaDB, PostgreSQL, Oracle or SQLite?
Maybe your application should support all of them? Maybe some?

Should it have methods like `store_to_sqlite` and `store_to_postgres`?

How about you provide a web-service for your users. You send the data across the
wire and some `JavaScript` should read the user. So what you gonna do? Add some
`to_json` to your `User` class?

Let's say you pick some support for some databases and you pick some serialization
(like JSON) and publish your code as a library someone else can use.

You go on a vacation and a week later some users where impressed by your library.
Some started to use it, but some have some feature Request.

Someone is developing a web-service but they use `XML` instead of JSON, so they
send you a pull-request for a `to_xml` method that is added to `User`.

Someone else uses MongoDB so they ask if also *MongoDB* could be supported. Someone
else wanna use *Cassandra* and someone else wants to use *CSV*.

You implemented a `to_string` that only showed the *User ID* of your objects because
that was all you needed for debugging and logging. But someone else wanna have
a Terminal TUI so he wants a better `to_string` implementation that shows
all data.

And anyway, why the hell are you so dump and implemented comparison by comparing the
*User Id*? Someone else wanna sort all users by its *Username*. Someone else
is asking by sorting it by the *Real User Name*. Somone else wants it sorted
by the *Username* and than by *Age*. I mean, you get it, right?

In some organization people under the age of 18 are not allowed, so they request
that validation should check for the age of 18. Maybe we can add a configuration
to configure the minimum age?

This is a *Rich Domain Model*. And I hope you know understand why a
*Rich Domain Model* is utterly crap.

They seems to be cool but it leads to exactly those kinds of problems that
*object-oriented* programmer always run into. Into [God objects](https://en.wikipedia.org/wiki/God_object)
that just do too much.

# A better alternative. The Anemic Domain Model

So how do we better? The answer is by doing less. First of you must forget
the idea that just having data is bad. Just having data is absolutely fine.
In the past maybe 20 years a lot of *object-oriented* programmers are telling
that it somehow bad (because it is procedural), but besides just claiming
and repeating it, it doesn't become true.

Separating data also doesn't mean that there is no code at all. It just means
you put it somewhere else, instead of attaching it to the class as a method.

Besides your data-structure that holds just data and maybe allows you to change
pieces of the data, you will not add any validation, equality or comparison or any
kind of other code to your data.

But you still can come up with some code that helps you create objects from it.
Maybe add some validation there to it. Maybe you think a Birthday cannot be in
the future. So you create your own library that adds some extra validation to it
to forbid this kind of thing.

But someone else that doing a *Warhammer 40K* website that keeps track of some
Warhammer characters maybe want the ability to create *birthdays* in the future.

He maybe can use some functions that you have written, some he don't use, some
he implement himself different.

The organization that wanted only to have people older than 18 could just
implement its own function to create users that disallows user that are too young.

In the world of an *Anemic Domain Model* you started to implement a useful
data-structure to manage *Users*. One year later you find a dozen of libraries
out there that uses your data-structure.

From `user_json`, `user_mysql`, `user_sqlite`, `user_warhammer`, `user_adult`, ...
you found around 100 libraries that have different validation, purposes and use
cases. You can load all of them, or just some of them you really need.

And the best is, you never created *God objects* or are faced with the problem
that your classes have to many *unenccesary dependencies* that is seen so often
in *object-oriented* code.

# Am I wrong?

Maybe I am wrong and you still wanna use a *Rich Domain Model*. It's not a
problem, but then at least I never wanna work with you and never wanna see code
from you.

# Related Posts

* [Object-Oriented Programming in C]({{< ref 2023-11-01-object-oriented-programming-in-c.md >}})
* [Why I Like Perl's OO]({{< ref 2024-04-15-why-i-like-perls-oo.md >}})
