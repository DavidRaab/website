---
layout:  post
title:   "Why I Like Perl's OO"
slug:    why-i-like-perls-oo
date:    2024-04-15T00:00:00
lastmod: 2024-04-15T00:00:00
tags:    [perl,oop]
description:
---

I started programming with QBasic and later C. While it was difficult to learn
those languages back then (with no internet and just some books) nothing felt
"out of place" for me.

For example, when I learned about arrays the first time, it took some time
to learn the concept of it, that it is a storage of multiple elements one
after another. How it is used and what you can do with it. But nothing
of it feelt like magic without knowing what happens.

But this was different to me when I started to learn object-orientation in C++.

The first time I learned C++ I was teached that every class is like a template
(no confusion with C++ template here) and whenever you use the `new` operator
an instantiation of it was created. You have some member variables that are somehow
unique to every object. You can write methods and those methods somehow access
those member variables that are somehow different to every object.

Then you get teached a lot of inheritance, and in my opinion just extremely bad
examples of it. Yeah I somehow did understand the concept, I could do some stuff with
it, but I often still wondered what really happens. Object-orientation almost
looks like magic and I had zero clue about what really happens under the hood.

This changed when I learned Perl object-orientation. It was one of my first moments
where I thought. Hey, now i really understand OO! And this is the reason why I like Perl
OO. It is sometimes criticized of being low-level. And while this might be true,
this low-level is what made me better understand it.

So here is some basic introductions to Perl's object-orientation and I hope it
will help you the same as it helped me understanding OO better.

# References

Different to most modern languages, references in Perl are explicit not implicit.
This comes directly from C. You have a data-structure/variable and you can create
a pointer to that data.

In Perl it is the same. Only difference is that you get some more level of security
compared to C, and automatic memory management.

For example this is an array in Perl.

```perl
my @data = (1, 2, 3, 4, 5);
```

You could call a function this way.

```perl
someFunc(@data)
```

but what happens here is that the array basically gets copied. Not only copied,
the array is expanded. In reality you basically call `someFunc` with 5 elements.
The above call is the same as calling.

```perl
someFunc(1, 2, 3, 4, 5);
```

This is a lot different to most other languages. In most other languages arrays
are pointer/references. For example in JavaScript you might write.

```js
var data = [1, 2, 3, 4, 5];
someFunc(data);
```

But in JavaScript `data` is already a reference. It is a reference pointing to
an array somewhere in the memory. That address is basically passed to `someFunc`
here.

`someFunc` in JavaScript basically receives only one value. A reference to an array.

You can achieve the same in Perl by creating a reference. You do that by adding
`\` in front of any variable.

```perl
my @data = (1,2,3,4,5);
someFunc(\@data);
```

Now we don't pass 5 values anymore, we pass a reference to `@data` to `someFunc`.

Perl also has a built-in way to directly create an array as a reference. It looks
the same as in JavaScript.

```perl
my $data = [1,2,3,4,5];
someFunc($data);
```

In the above case `$data` is a reference to an array, and you pass a reference
to `someFunc`. We also could have written.

```perl
my @data = (1,2,3,4,5);
my $data = \@data;
someFunc($data);
```

People coming to C, from other languages like C# or Java, often complainin that
they don't understand pointers. But the interesting part is often that they are
using pointers throughout the language without knowing. Whenever they create
an array, hashes or any kind of object they basically working with pointers.

But the interesting question is. Why do we pass pointers anyway?

Okay that question is a big one and i could write a lot about it. A quick answer
is that we don't want to pass copies. We want to pass a reference to data, and
a function is able to manipulate the data.

Or in other words. The reasons are for performance and mutability.

Here is a quick example.

```perl
my @data = (1,2,3,4,5);

sub add10_array(@array) {
    push @array, 10;
}

sub add10_ref($array) {
    push @$array, 10;
}
```

For example calling `add10_array(@data)` would basically do nothing. You add
`10` to the copy of `@data`.

But calling `add10_ref(\@data)` would really add `10` to `@data` that is defined
outside the scope of the function. Calling this function once changes `@data`
to `1, 2, 3, 4, 5, 10`.

The last behaviour is what you get by default from languages like JavaScript,
C#, Java and so on.

# dereferencing

As shown above there is a distinction between an array and a reference to an array
in Perl. (We also can create references to hashes, subroutines, variables or other
references again, ...)

A reference is basically just a pointer to some other memory address. But when
we want to access where it points to, we need to de-reference it. In Perl
there are multiple ways how to do that.

```perl
my $data = [1,2,3,4,5];

my $first = @{ $data }[0];
my $first = @$data[0];
my $first = $data->[0];
```

The typical and most often used way is by using the `->` operator. Also called
the de-reference operator. It's an arrow that follows the address it is pointing
to. Somehow makes sense that it is this symbol (instead of a dot).

# Packages

Packages are just Perls way of *namespaces*. By default when you don't define a
package you are just in the **main** package. This is usually where you
write your main program.

Otherwise you always can switch to a different namespace by writing `package Foo;`

Every package can have its own functions, global variables and so on.

For example we can do the following.

```perl
package Foo;

sub hello {
    print "I am Foo::hello\n";
}

package Bar;

sub hello {
    print "I am Bar::hello\n";
}

package main;

Foo::hello();
Bar::hello();
```

Running this program would just print

```
I am Foo::hello
I am Bar::hello
```

it also shows how you call functions from other namespaces. Namespaces are
just seperated with two colons `::`. This is also how it is done in C++.

You als can import functions from packages. But technically this is not
a supported Perl language feature. We also call it exporting (not importing)
as the module usually exports its functions to other modules when it gets loaded.

But i don't cover this as it is unimportant for understanding the OO part of Perl.

# bless

Perl whole object-orientation is basically implemented by references, packages and
the function `bless`. `bless` is a function expecting two arguments.

1. Any kind of reference.
2. Any package name as a string.

`bless` then adds some information to the reference to which package it belongs.
By default this doesn't change much. You still can use the reference exactly
the same. For example.

```perl
my @data = (1,2,3,4,5);
my $ref  = \@data;

print $ref, "\n";        # Prints: ARRAY(0x5592bc17ddb0)
printf "%d\n", $ref->[0] # Prints: 1

bless($ref, 'Foo');

print $ref, "\n";        # Prints: Foo=ARRAY(0x5592bc17ddb0)
printf "%d\n", $ref->[0] # Prints: 1
```

As you can see here. `$ref` is always a reference to an array, and you always
can for example use `$ref->[0]` to access the first argument of that array.

But once `bless` is called it adds the information that method calls should
be searched in the package `Foo`.

Method calls are also done in Perl with the `->` operator. So with a blessed
reference now you can write.

```perl
$ref->hello();
```

What this does is the following.

1. In the reference it will look to which package it was blessed.
2. It then does a transformation.

In this example `$ref` was added the information of being part of the
package `Foo`. The transformation that then happens is that the full function
just get called, and the variable itself is passed as the first argument.

So the above code turns into.

```perl
Foo::hello($ref);
```

The only difference is that this kind of method lookup is done at runtime. Not
at compilation time. That's also one reason why object-orientation should
be dynamic typed in my opinion. This is even true for some other languages. And
it is also one of the reason why object-orientation is in general slower and has
some overhead.

This concept is also called a [Virtual Method Table][VMT]

I am not a fan of inheritance and avoid it at all. But for completeness. Every
package in Perl can have a global `@ISA` array. It contains a list of packages
that should be searched if a method was not defined in that package.

So for example if you do `$ref->hello()` and `Foo::hello` was not defined, but
`our @ISA = ('Bar')` was defined it tries to call `Bar::hello($ref)` next.

You see why this can be slow?

But at least in Perl some caching happens so not every method dispatch must be
found every time.

And yes, this kind of magic is also what happens in languages like C# or Java.
All just hidden from you.

This is also the reason why in Perl you always explicitly write `$self` as the
first argument of any method. Python programmers do the same. Because that is
what it is. The object, usually just some kind of data like a hash, array and so
on is passed as the first argument to every method.

In typically OO languages that kind of information is hidden, but you have a special
variable like `this` that somehow represent this first argument.

In the world of C, [object-orientation was once just a design-pattern][OODP] until
languages decided to make it a core feature of the language. Do you think that this
was a good decision?

[VMT]: https://en.wikipedia.org/wiki/Virtual_method_table
[OODP]: {{< ref 2023-11-01-object-oriented-programming-in-c.md >}}

# Related Posts

* [Why I Like Perl's OO]({{< ref 2024-04-15-why-i-like-perls-oo.md >}})
* [Class vs. Hash]({{< ref 2024-05-08-class-vs-hash.md >}})
* [Object-Oriented Programming in C]({{< ref 2023-11-01-object-oriented-programming-in-c.md >}})
