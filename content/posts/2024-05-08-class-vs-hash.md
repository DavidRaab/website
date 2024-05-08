---
layout:  post
title:   "Class vs Hash"
slug:    class-vs-hash
date:    2024-05-08T00:00:00
lastmod: 2024-05-08T00:00:00
tags:    [perl,oop,metaprogramming]
description:
---

In my previous articles about object-orientation I explained how OO in
general works. I used Perl for the explanation, as it doesn't hide its itnernals
like most other languages do. That's [Why I like Perl's OO][Perl-OO].

Usually OO works that we define a *mutable* data-structure and pass it
to a function that mutates it. So in some sense calling `func($obj, $arg1)`
is the same as `$obj->func($arg1)`. It's just a different syntax and order
we write, but the concept is the same.

This is also how [object-orientation is used in C][C-OO], by using a `Struct`
as the mutable data.

Technically there is a big difference between a `Struct` in C and a `Hash` in
Perl, but the way we use them is similar.

In C we must define a `Struct` with its names and it types, because C is a
statically typed language. But in Perl we don't need to define the types first
because it is dynamically typed. We directly can use the names and values we
wanna use.

Let's see from an example. Consider I wanna save the name, birthday and weight
of a person. In C, I need to create a `Struct` first, with its types.

```C
struct Person {
    char   *name;
    struct tm birthday;
    float  weight;
};
```

Only after this definition I can create or use a `Person` struct. I don't go
into detail to create a variable from it. To annoying. But in C you need to
malloc the memory first. It's like creating an object with `new` in a
ypical *modern* object-oriented language like C#.

On the other hand, in a dynamically typed language like Perl, I can directly
create the data-type itself without prior definition of the type. I just can
write for example.

```perl
my $person = {
    name     => "Alice",
    birthday => DateTime->new(year => 1865, month => 7, day => 4),
    weight   => 40.8,
};
```

When I started to learn Perl back in 2003 that's also how I learned coding it,
and also how a lot of other code used it. But then the [Insanity of OO][insanity]
came to Perl. Suddenly this was not enough anymore. Because *good* object-orientation
told us to create prober classes. So we created them too in Perl. Suddenly
we needed to write this Perl code first, to be allowed to create a Person object/hash.

```perl
package Person;

sub new {
    my ($class, %args) = @_;
    return bless({
        name     => $args{name},
        birthday => $args{birthday},
        weight   => $args{weight},
    }, $class);
}

sub name {
    my ($self, $name) = @_;
    if ( @_ == 1 ) {
        return $self->{name};
    }
    else {
        $self->{name} = $name;
    }
}

sub birthday {
    my ($self, $name) = @_;
    if ( @_ == 1 ) {
        return $self->{birthday};
    }
    else {
        $self->{birthday} = $name;
    }
}

sub weight {
    my ($self, $name) = @_;
    if ( @_ == 1 ) {
        return $self->{weight};
    }
    else {
        $self->{weight} = $name;
    }
}

sub to_string {
    my ( $self ) = @_;
    return sprintf('Name=[%s] Birthday=[%s] Weight=[%s]', $self->name, $self->birthday, $self->weight);
}
```

You can tell me a lot, but I just hate to write this kind of code. And I am
not the only one in the Perl community who thinks so. Why do i don't like this?

After years of working with Perl I actually liked statically typed languages.
I think it is a good idea to have some types and a definition. Because you always
have some types. Dynamic typing does not mean *no typing* as some people sometimes
believe. It just means the concrete types are only known at runtime as already
known as compilation-time (before some code is running).

While dynamic typing allows some interesting stuff, I learned from practice that
in over probably 99% of coding you never need this stuff. The above code is even
an example to bring static typing into a dynamic typed language.

The only reason why we write those *packages* in Perl is, so we can add some
type-checking. For example in the `new` function we can test if the user passed
the required fields and correct types. For example we could add a checking
that `birthday` must be passed and it must be a `DateTime` object. Or maybe
some valid string that then can be turned into a `DateTime`.

But the above code, does none of this. The above class is basically just
boilerplate-code doing absolutly nothing.

Consider. When we just have a hash. We can easily get and set a value.
It's built into the core language of Perl. Getting for example the birthday of
a hash would be.

```perl
$person->{birthday}
```

and setting would be

```perl
$person->{birthday} = DateTime->new(...);
```

with the above package/class we created, we just put some wrappers around those
abilities that is already present in the language itself. The only difference is
hat we need to write less curly braces.

getting becomes.

```perl
$person->birthday
```

and setting becomes.

```perl
$person->birthday(DateTime->new(...));
```

and just to avoid those two braces we need to write a 9 lines method like this.

```perl
sub birthday {
    my ($self, $name) = @_;
    if ( @_ == 1 ) {
        return $self->{birthday};
    }
    else {
        $self->{birthday} = $name;
    }
}
```

this code becomes so annoying to write. And it does not even do any kind
of type-checking yet! If you want this, you also need to code this!

But sure, [Perl allows some kind of meta-programming][meta-programming]. So instead
of always creating those damn getters and setters we can create a function creating
those methods. This looks like this.

```perl
sub getset($field) {
    no strict 'refs';
    *$field = sub {
        my ($self, $value) = @_;
        if ( @_ == 1 ) {
            return $self->{$field};
        }
        else {
            $self->{$field} = $value;
        }
    };
}
```

with such a function in place, I now can re-write the whole package like this.

```perl
package Person;

sub getset($field) {
    no strict 'refs';
    *$field = sub {
        my ($self, $value) = @_;
        if ( @_ == 1 ) {
            return $self->{$field};
        }
        else {
            $self->{$field} = $value;
        }
    };
}

sub new {
    my ($class, %args) = @_;
    return bless({
        name     => $args{name},
        birthday => $args{birthday},
        weight   => $args{weight},
    }, $class);
}

getset('name');
getset('birthday');
getset('weight');
```

I even can make a library for `getset`, so i don't have to create this function
again and again. Because of this easyness a lot of people already did this.
Like [fields](https://perldoc.perl.org/fields) or [Class::Struct](https://perldoc.perl.org/Class::Struct)
or maybe [Hash::Util::FieldHash](https://perldoc.perl.org/Hash::Util::FieldHash).

This is why we have millions of *object-oriented* modules in Perl. We have
[Class::Accessor](https://metacpan.org/pod/Class::Accessor), and sure
it most be faster in C, so we also have [Class::Accessor::Fast](https://metacpan.org/pod/Class::Accessor::Fast).

A lot of modules are primitive in functionality, they don't do more than the `getset`
I provided. Back in the year 2008, Steven Little implemented
a whole [Meta-Object Protocol](https://metacpan.org/pod/Class::MOP) in Perl that
is known from CLOS (Common LISP Object System). He implemnted it to create
[Moose](https://metacpan.org/pod/Moose), that somehow became probably the most
used/advanced OO module in Perl.

But it didn't ended here. Sure someone wouldn't like the overhead of that MOP, so
again someone created [Mouse](https://metacpan.org/pod/Mouse), because it is faster.
Then we had [Moo](https://metacpan.org/pod/Moo). And again we had so many *Moose*
modules with the same syntax, so isn't it better to just use the first one loaded?

So we got [Any::Moose](https://metacpan.org/pod/Any::Moose), but it is now deprecated.

Anyway, there are still a lot more of such Perl modules. Some modules
add some more functionality, some a little less. Some add type-checking some
don't.

But all they usually do is just add wrappers around setting and getting a key
from a hash. Those wrappers usually only makes sense when they add something,
like type checking. But then again, why not use a language that supports
defining types instead?

For example in F# I could define a type instead, and still have the easyiness of
Perl.

```fsharp
type Person = {
    Name:     string
    Birthday: DateTime
    Weight:   float
}

let alice = {
    Name     = "Alice"
    Birthday = DateTime(1865, 7, 4)
    Weight   = 40.8
}
```

and we can access it with.

```fsharp
let birthday = alice.birthday
```

or for example create a new structure with a changed field. In F# they
are immutable, that's why it creates a copy.

```fsharp
let alice2 = { alice with Birthday = DateTime(...) }
```

The same easiness, but everything is statically typed, faster and
typed-check at compilation time, instead of checking at runtime.

I don't want to make an advertisment for F#. I don't really care which programming language
you are using. Use whatever you like, but I hope you can see how annoying the lack
of static typing becomes, and how much performance we lose in Perl because of this.

And I often question why we even do that. Because all the reason why we create
those wrappers are usually extremely bad code.

Why do we create such getters/wrappers in OO? Besides type-cheking in Perl
it is because we can do some kind of horrible side-effects all over the place.

Examples are: Whenever you access a getter/setter some other field is manipulated,
a counter is raised, a callback is called, maybe some other object is changed,
some printing is done and all other crap all over the place happens that makes
code hard to understand. Typicall object-oriented mess people call *good object-oriented code*.

This is the essence of object-orientation. It always does more than you ever wanted. And
you don't even have a clue what it does *more*. Because it is all *encapsulated* and hidden from
you. The worst is that programmers even think that this is a *feature*. I call this a *bug*.

This kind of crap is what makes object-orientation a mess, hard to test and
raises complexity to infinity.

I cannot bear this anymore.

Please give me stupid data-structures. Data-structures that just holds data
and nothing more. The only thing I need is a little bit of type-checking, and
that's it.

Do you know that you also could restrict the keys a hash is allowed to have in Perl?
In [Hash::Util](https://metacpan.org/pod/Hash::Util) you have a `lock_keys` function
that restricts which keys are allowed. Setting a key that is not allowed throws
an exception.

```perl
sub person(%args) {
    my %obj;
    lock_keys(%obj, qw/name birthday weight/);
    for my $key ( keys %args ) {
        $obj{$key} = $args{$key};
    }
    return \%obj;
}

my $alice = person(
    name     => "Alice",
    birthday => "1900-01-01",
    weight   => 42,
);
```

If everything you wanted was to ensure correct field-names, then you also can
achieve it this way, without creating any kind of package/class.

When you don't use type-checking, can you see that you basically just copied the
whole functionality of a hash and you just can use a hash instead?

One advantage of a dynamic typed language is that we don't need to predefine
our data-structures. We just use them as needed. This advantage sure
has some drawbacks like typos or type-checking errors. But that's how it is.

When we add class/packages in Perl with type-checking all over the place, then
we lose the advantages of dynmaic typing, and still don't get the benefits of
static typing. To get those we had to change the language itself. We even lose
performance. So what is the point of creating so much classes/packages in Perl
with all of that overhead?

But it also translates to other languages. You maybe have seen this in Java
or C# too. People who get annoyed by writting those damn getters/setters all over
the place. Either they also use some libraries that creates those classes
at runtime through reflection or even use an editor (like Visual Studio) that
creates whole template classes.

Again, why not use a class with just public fields? This is what a hash technically
is in a statically typed OO language. It's a restricted hash that only has the defined
fields and types. The advanatages you think a getter/setter gives you, are no
advantages at all. They usually makes code worse.

Some people think that classes with public fields are bad, they call them
[Anemic Domain Model](https://en.wikipedia.org/wiki/Anemic_domain_model). Don't
be fooled, there is nothing wrong with an Anemic Domain Model. People that usually
tell you that they are bad are just close minded and also telling you that
everything not object-oriented must be bad.

[Perl-OO]:  {{< ref 2024-04-15-why-i-like-perls-oo.md >}}
[C-OO]:     {{< ref 2023-11-01-object-oriented-programming-in-c.md >}}
[insanity]: {{< ref 2024-03-16-insanity-of-oo.md >}}
[meta-programming]: {{< ref 2023-10-26-meta-programming-in-perl.md >}}
