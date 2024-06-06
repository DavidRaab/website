---
layout:  post
title:   "Insanity of Object-Oriented Programming"
slug:    insanity-of-oo
date:    2024-03-16T00:00:00
lastmod: 2024-03-16T00:00:00
tags:    [perl,fsharp,oop,data]
description: Describes why object-orientation is insanity.
---

# The insanity of Object-Oriented Programming

I started programming back in the 1990s. My first language I learned
was QBasic and after 2 days of learning it, i switched to C. Back in these
days there was no Internet. Okay, it existed but was not so common as today.
Most people still didn't even had computers.

I picked and learned C because when you got to the programming section in a store
it was full of C books. Learning C was really hard, especially because all I got
was a single book and nobody to ask for (nobody else that i knew did programming,
and no internet). When I didn't understand something I had to redo it, until I
could explain it to myself. Usually with trial and error.

Some years after I learned C and did some programming with it, the book shelf's
changed. Now it was full of C++ books. So I started to learn C++. I picked up some
C++ books, Visual C++ and so on. Did some MFC-GUI Applications and so on.

The books all started the same. Somehow they explained that the old-style C
programming (procedural-programming) was dead, and now everything has to be
object-oriented. Object-Orientation was the new standard, everything not OO
must be bad.

But you know what. I had a hard time understanding OO. In my career as a programmer
I maybe had like six moments where I thought: Oh, now I finally understand OO.

Today i just think that OO is maybe the worst crap I learned just wasting many of
my years as a programmer that I could have spent learning something better. But
anyway while those C++ and later Java books was full of its classes and design
patterns I once again learned another language.

In the year 2003 I started a apprenticeship as a system-administrator. Mostly I
learned Linux. And the language I came into contact was Perl. Maybe the most
used language in that field at that time. Not only for system administration, parsing,
but also for work as database and web development. I worked around 10 years
with Perl as a web developer. The importance of Perl is interesting for many reasons.

First of, Perl was designed for C programmers, it still has a procedural background
not an object-oriented on.

Second, Perl is actually a high-level (functional) language. It has many concepts
you will find in functional languages like LISP. Also automatic memory management
was a blessing in my opinion.

But the most important one is that is has two built-in data-structures. The built-in
self expanding `Array` and a `Hash`. And the `Hash` is what was changing the way how
I programmed the most.

For example I can build the following data-structure in Perl.

```perl
my $album = {
    artist => 'Michael Jackson',
    title  => 'Thriller',
    tracks => [
        { name => 'Baby Be Mine',                duration => '04:20' },
        { name => 'The Girl Is Mine',            duration => '03:42' },
        { name => 'Thriller',                    duration => '05:57' },
        { name => 'Beat It',                     duration => '04:18' },
        { name => 'Billie Jean',                 duration => '04:54' },
        { name => 'Human Nature',                duration => '04:06' },
        { name => 'P.Y.T. (Pretty Young Thing)', duration => '03:59' },
        { name => 'The Lady in My Life',         duration => '05:00' },
    ],
};
```

What does it represent? Sure the Michael Jackson Thriller Album. I guess even
non-programmers can understand it. Back in the year 2003 this was awesome.
Perl programming was really about reading/parsing data, turning it into a
useful data-structure that you need, for whatever you are doing, and then
working with it.

Working this way is easy, understandable and you can get a lot of stuff done pretty
fast. It also works great for debugging and testing. There is nothing better than
just printing a data-structure. And when you can print it, you also can write
a test for it. I guess that's also the reason why testing was more common
in languages like Perl.

What you see above is probably what you have seen today in programming and
will remind you on JSON. Only difference is that it took another 15 years
for the *enterprise elite programmers* to know about something like that
(including the importance of testing).

But sadly, the object-orientation mess did not stay away from Perl. And sure,
we can make out some shortcoming with the above code. A hash is completely untyped.
Instead of `artist` as a key name someone also could provide a key-name with
`Artist` or maybe `atrist` without that the program throws any errors. Or at the
wrong places.

So for me as a Perl programmer it was totally normal to work with data-structures,
but because of these *problems* people started to turn every bit into its own class.

They did what programmers do that only know about classes and arrays, they write
classes!

So instead of a hash containing data, they write a `Album` class. And sure
it cannot just be an array of hashes. All must be an Array of `Track`. So
they create a `Track` class.

In the end they write something like that:

```perl
package Track;
use Moose;

has 'name'     => ( is => 'rw', isa => 'Str', required => 1 );
has 'duration' => ( is => 'rw', isa => 'Str', required => 1 );

package Album;
use Moose;

has 'artist' => ( is => 'rw', isa => 'Str', required => 1 );
has 'title'  => ( is => 'rw', isa => 'Str', required => 1 );
has 'tracks' => (
    is       => 'rw',
    isa      => 'ArrayRef[Track]',
    required => 0,
    default  => sub { [] },
);

sub add_track($self, $track) {
    push $self->tracks->@*, $track;
}
```

and with those two classes now you can re-create the Album like this:

```perl
my $album = Album->new(
    artist => 'Michael Jackson',
    title  => 'Thriller',
);

$album->add_track(Track->new(name => 'Baybe Be Mine',               duration => '04:20'));
$album->add_track(Track->new(name => 'The Girl Is Mine',            duration => '03:42'));
$album->add_track(Track->new(name => 'Thriller',                    duration => '05:57'));
$album->add_track(Track->new(name => 'Beat It',                     duration => '04:18'));
$album->add_track(Track->new(name => 'Billie Jean',                 duration => '04:54'));
$album->add_track(Track->new(name => 'Human Nature',                duration => '04:06'));
$album->add_track(Track->new(name => 'P.Y.T. (Pretty Young Thing)', duration => '03:59'));
$album->add_track(Track->new(name => 'The Lady In My Life',         duration => '05:00'));
```

I think it is worse because of multiple reasons.

But first the benefit. The only real benefit you get is a little bit more
*type-safety*. The classes created by **Moose** require the correct fields.
So you cannot forget to instantiate an *Album* without an `artist` or `title`.
This catches some typos.

Now to some disadvantages. This style of programming forces you to
turn every hash into a class. Maybe even the arrays are turned into classes
or forces you to create a method on the class who uses that array.

In my opinion it is just an awesome amount of boilerplate code you write again
and again. You just build wrappers around already built-in functionality that
exists.

You see this perfectly with the `add_track` method. The `add_track` method
is nothing else as just a wrapper around the `push` function to add an entry
to an array.

How about getting a Track by its index? Do you add an `get_track_by_index`,
that is just a wrapper around the indexing for an array like `$self->tracks->[$index]`?

And if you follow stupid laws like the `The Law of Demeter` you will create a
lot of those stupid methods. Consider you want the total duration of an Album.

With a pure data-structure I just go through the tracks and sum up the duration
of every track. In a "perfectly fine" (whatever that means) class where every
attribute is private (because people think it is dangerous to access data) they
must provide a `duration` method on the `Album`. And what do they do in that method? They
go through every track and sum the duration up.

In a halfway good language that should be so easy that you simple don't need to
provide such a method on a class.

But what do you do if it is not provided by the `Album` class? I hope the
`tracks` field is *public* and you can implement it yourself. But most of the
time it is *private* (at least in other languages like Perl).

But you maybe can re-implement it by using the `get_track_by_index` function.
And anyway what is the point of making the `tracks` field *private* in a class
when you provide a `get_track_by_index` method anyway?

What is the point of making data private if you anyway provide all kinds of methods
to access or change data?

With the idea of private data and turning every hash into a class the amount
of methods you have to provide and think about for your users just become
totally insane. All you do is re-write all kind of data-structures functions
again and again.

In a Album/Tracks relationship you write `add_track`, `remove_track`, `track_by_index`, ...

In a Hospital/Patient relationship you write `add_patient`, `remove_patient`, `patient_by_index`, ...

And don't let's talk about testing. In Perl I can easily test if two data-structures
are the same. Testing modules provide these functions. But I cannot test if two
obects are the same. Not unless I implemented an equal method on that class.

Readability and understand-ability is just worse. You cannot print or show a class.

I started with the whole data-structure that was understandable on its own. But
let's consider you get a pointer (that is what an object is) to an Album class
that has methods like `name`, `duration`, `tracks` and `add_track` on it. Do you
know what you got?

Of course not. You know you got an Album. But which Album? Who knows! Maybe you
want to implement a printing method that just prints the data in a class? Then
why do you not use just data in the first place?

Also creation of such objects is just worse. Data-structures are usually
complex. Complex in the sense that they are nested. They can have hashes with
field containing arrays containing hashes again. Perl, and many other languages,
provide an easy language built-in to create and access such structures.

But with classes you usually must create them *flat*. Like I showed above. You
first create an `Album` and then add a single `Track` to it. The more complex
your data-structure is, the worse it will get.

It will even get more worse if you have immutable objects, because then you must
create them inside-out. Meaning from the deepest level first. So you must
create all *Tracks* before you can create the *Album* as the last step.

# A way out

So what is a better approach to all that mess?

In my opinion there is nothing wrong in directly using a data-structure. Yes
it has some disadvantages as you can have some typos, but the disadvantages
are less a problem of that class mess above that creates an explosion of
boilerplate code.

But there is still one thing you can or should do. Consider functions that works
on whole data-structures, not just part of it. For example, you could create
an `album` and a `add_track` function like this.

```perl
sub album($artist, $name) {
    return {
        artist => $artist,
        name   => $name,
        tracks => [],
    };
}

sub add_track($album, $name, $duration) {
    push $album->{tracks}->@*, { name => $name, duration => $duration };
    return;
}
```

This allows you to give you some type-safety. The function that create the
data-structure creates it for you. In some sense it is still a lot comparable
to the OO way. You can create the same Album like this.

```perl
my $album = album('Michael Jackson', 'Thriller');
add_track($album, 'Baybe Be Mine',               '04:20');
add_track($album, 'The Girl Is Mine',            '03:42');
add_track($album, 'Thriller',                    '05:57');
add_track($album, 'Beat It',                     '04:18');
add_track($album, 'Billie Jean',                 '04:54');
add_track($album, 'Human Nature',                '04:06');
add_track($album, 'P.Y.T. (Pretty Young Thing)', '03:59');
add_track($album, 'The Lady In My Life',         '05:00');
```

but you are not forced to. You still can create the whole data-structure
itself as shown in the beginning. It is up to you if you want to use those functions
or bypass them.

In this style you usually only provide new functions that are more complex and
actually provide a real benefit of adding. I guess you maybe will not even
create a `add_track` function. In the class based style on the other hand you
must provide a method for basically any kind of primitive functionality.

You also can just add an `is_album` function that checks for the validity
of a data-structure of having all required fields. Something you maybe
want to add anyway when your data comes from external sources like parsing
from a file, JSON over a Socket, a database and so on.

# Static Typed Languages

Good static typed language also can help here. For example in F# i would write.

```fsharp
type Track = {
    Name     : string
    Duration : string
}

type Album = {
    Artist : string
    Title  : string
    Tracks : Track list
}
```

This kind of definition allows me to write.

```fsharp
let album = {
    Artist = "Michael Jackson"
    Title  = "Thriller"
    Tracks = [
        { Name = 'Baby Be Mine';                Duration = '04:20' }
        { Name = 'The Girl Is Mine';            Duration = '03:42' }
        { Name = 'Thriller';                    Duration = '05:57' }
        { Name = 'Beat It';                     Duration = '04:18' }
        { Name = 'Billie Jean';                 Duration = '04:54' }
        { Name = 'Human Nature';                Duration = '04:06' }
        { Name = 'P.Y.T. (Pretty Young Thing)'; Duration = '03:59' }
        { Name = 'The Lady in My Life';         Duration = '05:00' }
    ]
}
```

and it is completely type-safe. The difference is that F# programmers still
differentiate between data and code. The above are records comparable to hashes
(but immutable). Not objects that try to mix up data with all kind of methods
on it or trying to hide data (private fields) from it.

There are two things I would suggest.

1. Consider your data and code seperated.
2. Write functions that operates on complex-data as a whole. Do not dissect
   data into thousand small pieces of classes.

When you do that your code becomes less insane. But you maybe will lose
your job as the *elite enterprise programmer*. They need at least another 10
years to understand this.

# Related Posts

* [Why I Like Perl's OO]({{< ref 2024-04-15-why-i-like-perls-oo.md >}})
* [Class vs. Hash]({{< ref 2024-05-08-class-vs-hash.md >}})
* [Object-Oriented Programming in C]({{< ref 2023-11-01-object-oriented-programming-in-c.md >}})
