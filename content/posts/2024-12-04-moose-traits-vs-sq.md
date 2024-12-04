---
layout:  post
title:   "Moose Traits vs. Sq"
slug:    moose-traits-vs-sq
date:    2024-12-04T00:00:00
lastmod: 2024-12-04T00:00:00
tags:    [perl,sq,oop]
description:
---

# Moose Traits vs. Sq (Explicit language)

Did you ever hear of the **Law of Demeter** in object-orientation? Here
is the idea behind it. Let's assume we have a class, and that class has
a property named `options`. This `options` is an array.

In procedural or functional programming we would just access that array
and do whatever we wanna do with it. But somehow in object-orientation
land people think this is bad. What you should instead do, that's what
they are telling, is you should create **methods** that modify that attribute.

Let's see some Perl Code to understand better. We have that simple class.

```perl
package Stuff;
use 5.036;

sub new($class) {
    return bless({options => []}, $class);
}

# getter
sub options($self) {
    return $self->{options};
}
```

With just that class we could do something like that.

```perl
my $stuff = Stuff->new;
push @{ $stuff->options }, 'whatever';
```

we could get a reference to the Array, then push values onto it. But
"**proper**" object-orientation and the **Law of Demeter** tells you, you
shouldn't do that. So how do we solve the issue to add a new option to
the `options` field? Of course we need to write another method that does
that.

```perl
sub add_option($self, @args) {
    push @{ $stuff->options }, @args;
}
```

Now with this method we finally can do.

```perl
$stuff->add_option('whatever', 'yes', 'no');
```

But this kind of code is actually kind of silly to say at least. It basically
just wraps our `push ...` code we anyway could write into a method.

You know what i think is funny? Object-orientation always tells you that it
somehow delivers re-usable code, but it doesn't. This is just one example.
Now for every array you have and you want to add a field you have to write
the same method again and again.

You have another field that is an array and you wanna be able to add something
to it? Yeah, write that method again. Just with another name.

```perl
sub add_flag($self, @args) {
    push @{ $stuff->flags }, @args;
}
```

How often do you wanna write crap code like that? Maybe after your thousands
object you created and thousands of such properties, maybe you finally will
release how idiotic this kind of coding becomes.

So, what is a better alternative? Well, i don't know if it is better, but in
Perl we easily can create the methods through code. For example we could
generate the following function, that returns such a method.

```perl
sub gen_push_array($field) {
    return sub($self, @args) {
        push @{ $self->{$field} }, @args;
    }
}
```

With such a function we now can write.

```
no warnings;
*add_option = gen_push_array('options');
*add_flag   = gen_push_array('flags');
```

So the function `gen_push_array` actually creates the method for us and
when Perl's compilation stage ends it creates the methods for us by installing
the methods to the symbol table.

It reduces the code we have to write a lot. We still have to write a single
line to add such a method, but this one line is better than always writing
the whole method again and again, or not?

When we are just pushing a value onto an array then it doesn't seems like
a great win. But this rapidly will change once you have more complex logic.

What is about sorting that array? Finding a value in that array? how about
getting the minimum or maximum value? How about summing all values, iterating
that array and so on.

Do you always want to re-write the same logic again and again? Don't you have
more important stuff todo as a programmer as writing those silly functions
again and again?

# Moose to the Rescue?

Now we could write our own library and put functions like `gen_push_array`
into it. Functions that when you call them create the apropiate methods for
you. But we already have those libraries. `Moose` is one of them.

But before we delve into that, here is one important question you can think
of. What is wrong about just having a utility library?

Let's assume we want the minimum value of an array, we just can write.

```perl
sub min($array) {
    my $min_so_far = undef;
    for my $x ( @$array ) {
        if ( defined $min_so_far ) {
            $min_so_far = $x if $x < $min_so_far;
        }
        else {
            $min_so_far = $x;
        }
    }
}
```

when we want the minimum value of an array i now can write.

```perl
my $min = min( $object->method_arrayrref );
```

why the hell do think object-oriented programmers think that this `min`
function must be part as some method on every damn object?

```perl
my $min = $object->min_method
```

Just think clearly for a moment and forget all that OO superior bullshit you
are wrongly teached.

A procedural function like `min` would be re-usable and you only have
to implement it a single time. While in object-oriented land you basically
have to write such a min function for every damn array field in every damn
class you ever create.

And while you do that. They are lying into your face and telling you
how object-orientation helps in writing re-usable code. Re-usable my ass!

Okay, let's go to **Moose**. They way how **Moose** "fixes" our problem is
by giving you traits and creating delegate methods for you. It looks like
this.

```perl
package Stuff;
use Moose;

has 'options' => (
    traits  => ['Array'],
    is      => 'ro',
    isa     => 'ArrayRef[Str]',
    default => sub { [] },
    handles => {
        all_options    => 'elements',
        add_option     => 'push',
        map_options    => 'map',
        filter_options => 'grep',
        find_option    => 'first',
        get_option     => 'get',
        join_options   => 'join',
        count_options  => 'count',
        has_options    => 'count',
        has_no_options => 'is_empty',
        sorted_options => 'sort',
    },
);
```

So what this code does is the following. It creates a class with the field
`options` that by default is an empty array. It also does a type checking
of just accepting strings.

Additionally it creates methods like `add_option`, `map_options`, `filter_options`
and so on as methods on `Stuff`, so you don't have to write that crap again
and again.

You can use that `Stuff` class like this.

```perl
my $stuff = Stuff->new;

$stuff->add_option('whatever', 'foo');
my $foo   = $stuff->get_option(1);
my @fs    = $stuff->filter_options(sub { m/\Af/ });
my $count = $stuff->count_options;
```

Seems nice, now we don't need to at least rewrite that whole functions again
and again.

# Doing it the Sq way

So now let's look at how we work with `Sq`. First of, we don't think in
objects. We go back down at how Perl started, we think and divide things
into data and code. We have code that operate on data. Data itself
is just data. And because we are in Perl it's dynamic typed.

Dynamic typing surely also has some drawbacks, but it also has some advantages
as it let's us built data in a very clean way without thinking about the
structure too much. We also can do multiple transformation steps with
intermediate data representation until we reach our final representation,
but forget about it here, this post is not about dynamic typing or static
typing.

So that `Stuff` class will be represensetd like this in `Sq`.

```perl
my $stuff = sq {
    options => [],
};
```

and voila, that's all. The `sq` function recursively traverse the reference
you pass it to and adds the `Array` and `Hash` blessing to that data.

You are supposed to work with `$stuff` like any Hash or Array you are used in Perl.
But the added blessing allows you to call some additional methods on it.

The idea is simple. Let's go back to the `min` example. Sq provides
a function named `Array::min`, and you can call this function in this
way

```perl
my $opt_min = Array::min([1, 10, -3]); # Some(-3)
```

As you can see it is basically just a helper function in the procedural
way. `min` is implemented once, no need to re-write it a thousand times.

But Perls object orientation is unique that by adding a blessing to a package
we basically can call it as a method. Now consider the following. While it looks
object-oriented, it is not. To it's core `Sq` just uses Perl OO feature
as a convenient way to write all functions in a chaining syntax.

```perl
my $opt_min = Array->new(1, 10, -3)->min; # Some(-3)
my $opt_min = Array::min([1, 10, -3]);    # Some(-3)
```

It's just about having a better syntax that allows chaining operation. But
you still can use both ways. The second way has the benefit that it also
works with unblessed Arrays as any function in the `Array` package should do.

So let's go back to our `Stuff` example. Here look at the comparison again.
This is Moose.

```perl
my $stuff = Stuff->new;

$stuff->add_option('whatever', 'foo');
my $foo   = $stuff->get_option(1);                  # foo
my @fs    = $stuff->filter_options(sub { m/\Af/ });
my $count = $stuff->count_options;
```

This is Sq.

```perl
my $stuff = {
    options => [],
};

$stuff->{options}->push('whatever', 'foo');
my $foo    = $stuff->{options}[1];
my $fs     = $stuff->{options}->filter(sub($str) { $str =~ m/\Af/ });
my $length = $stuff->{options}->length;
```

So what basically changes is

```perl
$stuff->add_option('whatever', 'foo');      # Moose
$stuff->{options}->push('whatever', 'foo'); # Sq
```

So all it does is basically a naming convention from `add_option` to
a `option_add` naming convention instead.

But, there is one big difference. You don't need to install those delegates.
Not only that, we don't even need to create a class. I mean to
work with `Stuff` and just the `option` field we had to create a package
and write nearly 20 lines of code just for the `option` field.

In `Sq` all of it is done in 3 lines of code and no package/class is needed. And not
only do you have the same advantages, it provides a lot more functions too!

Here are just some functions/methods that are currently written for the
Array package: **all, any, append, map, bind, cartesian, blit, copy, count, as_hash,
choose, distinct, sort, sort_by, min, max, filter, find, first, last, fold,
iter, intersperse, index, join, mapi, zip, push, pop, pick, repeat, reduce,
rev, skip, take, slice, split, sum, sum_by, windowed**

Those are not all functions, just to list the most important one. On top
it also provides a lot of functions that helps in creating arrays in various
ways. Those supposed to be called as constructors like `Array->new`.
We have: **new, init, unfold, range, range_step, replicate, concat**

# Validation?

One reason why we created methods in object-orientation was because of
validation. In some sense and for Perl this meant we can add type-checking.

And I don't think that fits Perl good. Look I absolutely like types in general,
as I like working with F#. They have they benefits and sometimes they are also
annoying.

With `Sq` i actually try to bring ideas from **F#** or in general **ML**
languages to Perl that heavily rely on types. But still I don't try to add
type-checking (everywhere) or the idea of fixed data-structures.

Why? Because if that is what you want, i guess you better use a language like
**F#**. Dynamic typing has it's benefits and this module tries to use those
benefits and not trying to work against them.

In Moose we add least had the ability to add some types, for example
`add_option` checks if everything we add is a `string`. But this has it's
price because this kind of type-checking happens whenever you call any
method and is a runtime type-check.

Consider that the typical type-checking a static typed language has happens
at compilation time, not at runtime. That is what **static** actually means.
So those languages have type-checking at the compilation step but those are
removed when the program runs. That's one reason why those languages are faster.

But you can see why adding runtime type checks into a language like Perl
that has no separated heavily optimized compiler steps or a Just-in-Time
compiler to optimize further is maybe not the best idea?

`Sq` in the future will provide a function based type-checking simply based
on predicates. That means you can create just a simple function that either
returns a boolean value if some kind of data-structure fullfills a type, or
don't. But this kind of type-checking leads to a different way on how you
work and build things.

You are supposed to work with data. Transform your data as much you want
and only after everything is done, you maybe check if whatever you created
is valid. You don't try to type-check every damn modification, especially
not when it implies a lot of cpu performance.

Further down the road `Sq` will provide an AOP (Aspect-Oriented-Programing)
mechanism to add type-checking to function calls. That means you can add
or remove type-checking at will.

For example as long as you develope you can add the type-checking and ensure your
code is correct and run your test-suite with it. But in production code when
speed matters those type-checks are removed. Very much like a **static** compiled
language.

# Another Example

Let's look at another example on why I think this programming model is superior.
Here is some data. Can you guess what it is just from seeing the data?

```perl
my $album = sq {
    artist => 'Michael Jackson',
    title  => 'Thriller',
    tracks => [
        {title => "Wanna Be Startin’ Somethin", duration => 363},
        {title => "Baby Be Mine",               duration => 260},
        {title => "The Girl Is Mine",           duration => 242},
        {title => "Thriller",                   duration => 357},
        {title => "Beat It",                    duration => 258},
        {title => "Billie Jean",                duration => 294},
        {title => "Human Nature",               duration => 246},
        {title => "P.Y.T.",                     duration => 239},
        {title => "The Lady in My Life",        duration => 300},
    ],
};
```

Now let's assume i want to get the track with the maximum running time. I
would write something like that.

```perl
my $opt_longest_track = $album->{tracks}->max_by(sub($track) {
    $track->{duration}
});
```

This just gives me the longest track as an optional value. What is an
optional value you may ask? It's a value that contains the direct representation
of either having a value or don't, but with methods on it. See `Sq::Core::Option`
for further information. I now could do the following.

```perl
$opt_longest_track->match(
    None => sub {
        printf "No longest Track found. No Tracks defined?\n";
    },
    Some => sub($track) {
        printf "Longest track is %s with %d seconds",
            $track->{title},
            $track->{duration};
    },
);
```

Now consider the following. OO fanatics tells you that directly accessing `tracks`
is bad. So how would you get the longest Track from a album?

I hope that class already implements that logic for you to access it. If not,
maybe it implements iterating through that array? When you are the one
who writes the class. Well than you have to implement at least a function
that iterates through tracks or get the maximum. Maybe both.

But again. Why the hell do you wanna rewrite iterating through an array?
USE YOUR FUCKING LANGUAGE FEATURES!!!

So here is a next improvement. You know, in Perl we work a lot with hashes,
one thing we often need is to select a key from a hash. And because `Sq`
heavily works with lambda it provides a helper function creating such functions.
Here is what you also can write.

```perl
# without 'key' function
my $opt_longest_track = $album->{tracks}->max_by(sub($track) {
    $track->{duration}
});

# with 'key' function
my $opt_longest_track = $album->{tracks}->max_by(key 'duration');
```

In the above example I used `match`. Actually `match` is an expression
and returns whatever those Some/None functions return. But it is also fine to just
do some side-effects like printing. But let's assume you only want
to execute some piece of code when you had some value, otherwise nothing
should print. Then you can use `iter` instead. The `iter` here comes
from `Option::iter` as `max_by` returns an `Option` value.

```perl
$album->{tracks}->max_by(key 'duration')->iter(sub($track) {
    printf "Longest track is %s with %d seconds",
        $track->{title},
        $track->{duration};
});
```

This piece of code searches for the longest track, and if it found one,
usually this is the case as long the array is not empty, then it will print it.
Otherwise nothing will happen.

# Classes/Methods are bad

You see what the biggest problem on classes or methods are? By default
"proper good" OO tells you, you never should access data directly. It goes
by the name **encapsulation**. But this kind of thinking brings you into a
corner that you only can do things that the class implemented for you.

But what do you do if you want todo something that is not present/implemented
by the class?

You see, when you just have data, it is completly up to you whatever you
want todo with your data. Having a great utility belt with hundreds of function
on Hash/Array provides you with all kind of tasks you need todo.

Let's say for comparison someone would have returned me `Album` class
but didn't provided me a proper iteration for the `tracks` field nor a
`longest_track` method. Then how am i supposed to work with that object?

At least some kind of iteration would be good, but you know, i don't want to
write a `max` function to pick the longest track for the thousand time.

Let's assume I am the one who have written the `Album` class, usually I then
write code at least for my needs. So what happens if I share my code and
now someone needs a function for also getting the `min` track?

Do i really seriously have to write it again and again? Does it mean that
whenever i create a class with an array field i better have to implement
like 50 functions again and again for every damn field so hopefully everyone
using my class can do whatever they need todo?

Isn't that a real bad and completely inefficent way to write code? Can it
get any more worse?

By exposing data, and just data, and giving everyone full access to data you
can assume the following.

## Fast

Accessing just data without method overhead. Calling a method is not for free
even if that method literally does nothing. In the distribution there is
a **benchmarks/moose.pl** file you can look into and execute. The fastest
getter/setter in pure Perl i could come up still was three times slower
than just accessing a field in a hash directly.

```perl
$obj->title;
$obj->{title};
```

So you basically just get 1/3 of the performance so you don't need to write
those curly braces. Setting is even more worse.

```perl
$obj->title('whatever');
$obj->{title} = 'whatever';
```

So setting means we replaced curly braces with parenthesis! Awesome! But at
least Lisp people will like that! And everything just costs us 2/3 of
performance and we have to use a big library with all kinds of code
generation because otherwise we go insane writing the millionst getter/setter!

## Re-Use

By just exposing Array/Hash and providing methods/functions on it you get
real re-use of code. No need to re-write the same old boring array/hash
transformations or extraction functions you already have written a thousand
times or maybe even more.

## No Limitations

By exposing the data directly with all of it's functionality I let whoever
use that data do whatever they like to it. They can extract and transform
everything as they like. And i don't have to care a single bit anymore in
implementing all kinds of methods because Array/Hash already ships with
the implementation. Let those people just use the data as they want. It's
good for everyone!

# Disadvantages?

Everything has its advantages and disadvantages. Exposing data directly,
especially mutable data can have some problems.

I don't think having invalid data in itself is so bad as people always claim,
but this is maybe a topic on it's own. The Moose example had one advantage
that it also checked for a string when we called `add_option`. So only
strings get added.

By exposing data directly you don't have that ability anymore to do
type-checking. For example someone could do

```perl
$album->{tracks}->push({foo => 1});
```

Someone just can add a silly hash like `{foo => 1}` that doesn't really
represent a Track. Is that a problem?

Yes and no.

Yes i can write code, return data that represent an album and someone else
can put garbage in it. It's possible, but why should i care? If people wanna
do stupid things just let them do that. It's not my problem when they invalidate
that data!

Maybe it also can have some reason why they put that into it? I mean maybe after
I returned my data a user does some transformation on it. Should i prohibit
as a programmer that he can do that? Why should I?

As a user of your own system all you usually need is just a function that helps
you in the creation of your hashes. If you need type-checking, just add a function!

```perl
sub track($title, $duration) {
    return Hash->new(
        title    => $title,
        duration => $duration,
    );
}
```

When you want to add some more type checking. Fine! Do whatever you want.
You don't need more than a function to add this. But still consider that
`track` just returns a Hash that allows you to freely change or modify
its data. Why should it be a problem? It doesn't have to be a `Track` class
with poor functionality!

Another way in how it is usually resolved is to sticking to **immutable** data
instead. This means once data is created, you cannot modify it anymore. Any
modification returns new data.

In some sense `Sq` is written in that way, because most function you write
create new data instead of modifying it's data. Even when Array or Hash
are mutable you use them like you do in immutable programming. The idea
is that you built up data, even in a mutable way, and once done you usually
don't change them anymore.

But again. The above shows one problem. It's too much the way of object
thinking. Why even create a `track` function. Why not directly create a
`add_track` function that expects an `$album` as a whole?

```perl
sub add_track($album, $title, $duration) {
    $album->push(tracks => Hash->new(
        title    => $title,
        duration => $duration,
    ));
}

add_track($album, 'Bonus Track', 240);
```

Stop thinking in the small and trying to make every field its own object,
consider the whole data-structure, no matter how deep that structure is with
hashes/arrays, as a whole unit that gets modified.

This way you come up with useful functions that really saves you time.
Don't do

```perl
add_track($album, track('Bonus Track', 240));
```

Even if that is also totally fine sometimes! But here it doesn't really makes
sense. A single **Track** is not something that really exists on its own, its
part of an **Album**.

Maybe you need the total runtime of an album?

```perl
sub album_total_runtime($album) {
    return $album->{tracks}->sum_by(key 'duration');
}

my $total_time = album_total_runtime($album);
```

Sure `Sq` also can come with speed penality. Even when I think that it usually
will outperform any kind of OO module it still has some overhead. It
heavily rely on creating lambdas all over the place and also expecting
and calling lambda functions. Calling functions is not for free!

As you can see in the **moose.pl** benchmark you already loose 2/3 of performance
just because you use a getter/setter to access a hash field!

So what do you do when you encounter performance problems? Well you always
can rewrite those functions in a pure perl version. In profiling
you measured that `album_total_runtime` causes most problems? No problem
just rewrite it!

```perl
sub album_total_runtime($album) {
    my $sum = 0;
    for my $track ( @{ $album->{tracks} } ) {
        $sum += $track->{duration};
    }
    return $sum;
}
```

The very fact that we threat `$album` just as pure data and we always have
access to everything is also the reason why we can do that kind of approach!
In an object-oriented codebase where we had some kind of objects we couldn't
do that! We at least had to open the maintainers code/class, maybe change
the implementation and then commit our changes hoping it maybe get accepted
or not.

While working together is nice. Why go through all of that trouble? Everybody
should be able to just write any kind of data-transformation function for some
piece of data without that any kind of collaberation is enforced!

It's also up to you how much `Sq` you wanna use. We also can write.

```perl
sub album_total_runtime($album) {
    return $album->get('tracks')->map(sub($tracks) {
        my $sum = 0;
        for my $track ( @$tracks ) {
            $sum += $track->{duration};
        }
        return $sum;
    })->or(0);
}
```

The `Hash::get` function we call on `$album` returns the field as an
optional. That means we either get **Some value** with the value in it,
or it returns **None** when the key `tracks` in the Hash does not exists
or is not defined.

It then calls `Option::map` on that result. Either executing our sum
function when we have `$tracks` or don't. The final `->or(0)` call
comes from `Option::or` and either returns the calculated sum or `0` when
we had no `tracks` field.

But the crucial loop that actually creates the most performance impact is
written without overhead. The above code translates to.

```perl
sub album_total_runtime($album) {
    my $sum = 0;
    if ( defined $album->{tracks} ) {
        for my $track ( @{ $album->{tracks} } ) {
            $sum += $track->{duration};
        }
    }
    return $sum;
}
```

# Getter/Setters are bad

But you know which lesson I have learned the most through all kinds of
languages and using typical OO code?

That all kinds of getters/setters are bad. Usually in OO they tell you
that it is good. When you have that getter/setter you can just do
that **one additional thing** while you get/set a value.

Oh gosh, and you know what? Whenever that happens and a getter/setter does
more than just getting or setting a variable, it ALWAYS CAUSES PROBLEMS! You
just think you retrieve that one thing, or set one kind of field, but
noooooo it does not.

Suddenly a callback is called, some kind of other hidden piece of data is changed.
Maybe you call a getter and think you just retrieve a field, but no, instead
it just calculates a whole value again and again whenever you call it
causing serious performance issues and you don't have a fucking clue that it
does that because you think it is just a stupid getter!

I mean look at `Moose`. When you create classes with `Moose` what do 99% of
your field do you create? I guess not much, you just set a field to either 'ro'
or 'rw' and maybe add a type constraint. That's what 99% of your fields look
like. Moose is just there because it's a pain to always write the full
`new` and all the getters/setters yourself!

But again, why bother with getter/setters? Most of the time classes are just
wrappers around hashes. And those getters/setters are just functions to get
and set a value in a hash, that's it. But it comes at a high price you pay so
you can maybe type two characters less. Remember.

```perl
my $time = $obj ->time;
my $time = $hash->{time};
```

One reason why we used classes in Perl is because its dynamic typing nature
of a Hash and how Perl works is prone to typing errors. Let's assume we
would have written. `ttime` instead of `time`.

```perl
my $time = $obj ->ttime;   # throws exception
my $time = $hash->{ttime}; # returns undef
```

Yeah, i get it. In that case it is really good that we get an exception,
because what we have here is basically a programming error. We try to
access a field that doesn't exists.

It's like not using `use strict` and suddenly every variable you not
explicitly define is not even an error anymore. Horrible. But Python
developers that have that mantra about explicit is better than implicit
will tell you that this is implicit creation of variables is not a problem.

But do you know that Perl added something like **restricted hashes** since
Perl 5.8? You actually can restrict a Hash to certain kinds of keys. So
whenever you try to read/write a key that is not allowed it throws an exception.
In `Hash::Util` you will find those functions to restrict a hash. Also those
abilities are built into `Sq`. You can write.

```perl
my $hash = Hash->new(
    title    => 'whatever',
    duration => 1,
)->lock;

my $title = $hash->{tile}; # throws exception
```

The `Hash::lock` method restricts the Hash to its current fields. So only
getting or setting `title` or `duration` is now allowed. Everything else
turns into an exception.

# Dynamic Typing

Let's go quickly about some advantages you can do in a dynamic-typed language.
Something you can do when you just treat data as data. Consider our Album again.

Somewhere we have written code and we call

```perl
my $album = $whatever->search("Michael Jackson Thriller");
```

and the result we get is our data-structure we had above.

```perl
my $album = sq {
    artist => 'Michael Jackson',
    title  => 'Thriller',
    tracks => [
        {title => "Wanna Be Startin’ Somethin", duration => 363},
        {title => "Baby Be Mine",               duration => 260},
        {title => "The Girl Is Mine",           duration => 242},
        {title => "Thriller",                   duration => 357},
        {title => "Beat It",                    duration => 258},
        {title => "Billie Jean",                duration => 294},
        {title => "Human Nature",               duration => 246},
        {title => "P.Y.T.",                     duration => 239},
        {title => "The Lady in My Life",        duration => 300},
    ],
};
```

But you know, just working with the duration as seconds is not the best way.
Maybe you wanna have a better representation of that?

```perl
sub seconds_to_str($seconds) {
    my $minutes = int ($seconds / 60);
       $seconds = $seconds - ($minutes * 60);
    sprintf "%02d:%02d", $minutes, $seconds;
}

$album->on(tracks => sub($tracks) {
    for my $track ( @$tracks ) {
        my $dur = $track->{duration};
        $track->{duration} = sq {
            seconds   => $dur,
            as_string => seconds_to_str($dur),
        };
    }
});
```

This is something that is basically not possible in a static typed language.
We just iterate through the `tracks` array and changing the duration
field.

We replace every duration with a new Hash that now contains two pieces of
information. For example a `duration = 240` now turns into.

```perl
{ seconds => 240, as_string => '04:03' }
```

Okay, this example is a little bit made up. Because, why should you change
that value when you have a function like `seconds_to_str` to just compute
the value?

But there are cases where this is helpfull, like when we iterate a
certain kind of data-structure multiple times and while we do that we compute
the time string multiple times. Instead of always recomputing the value
we just can change our data-structure and compute the value once. So
any other logic accessing the time field now has become a lot faster.

What you see here is also the abilities `Sq` offers when working with Hashes.
In this case we mutated the Hash, but we also can easily get a whole
new `$album` value with all our changes applied without changing
`$album`.

```perl
my $update =
    $album->withf(tracks => sub($tracks) {
        $tracks->map(sub($track) {
            $track->withf(duration => sub($dur) {
                Hash->new(
                    seconds   => $dur,
                    as_string => seconds_to_str($dur),
                );
            });
        });
    });
```

`withf` is a function that allows us to create a copy of a hash with
some fields changed. On `$album` we select `tracks` field to change
and pass it a function that computes a new value from the current one.

Then we just call `$tracks->map` and create a new Array by using
the current `$track`. Here again on `$track` we select `duration`
with `withf` to just change the duration and create a new Hash.

Now `$update` has all changes applied without that `$album` got modified!
It is up to you when you want to create new values or mutate data.

Sure in a static typed language you could create a new type representing
your new data structure. And doing this kind of immutable transformation is
also very easy in a ML-like language like F#. But the direct in-place mutation
is not really possible. Consider that while you do a mutation
there is a time when some `duration` fields will contain just an `int`
while other are already updated with a `Hash`.

# Further Info

* [Sq Project Website](https://github.com/DavidRaab/Sq)
