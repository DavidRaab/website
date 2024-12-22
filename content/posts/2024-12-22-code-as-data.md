---
layout:  post
title:   "Code as Data"
slug:    code-as-data
date:    2024-12-22T00:00:00
lastmod: 2024-12-22T00:00:00
tags:    [perl,sq,lisp]
description:
---

I have the following type definition.

```perl
my $album =
    t_hash(
        t_with_keys(qw/artist title tracks/),
        t_keys(
            artist => t_str(t_min 1), # string must have at least 1 char
            title  => t_str(t_min 1),
            tracks =>
                t_array(
                    t_min(1),         # Array must have at least 1 entry
                    t_of(
                        t_hash(       # All entries must be hashes
                            t_with_keys(qw/name duration/),
                            t_keys(
                                name     => t_str,
                                duration => $duration))))));
```

this is code that builds a function that you can run and type check against
any value. `$album` here is actually a function that you can execute and run
multiple times. What it does?

In words spoken. It checks if there is a hash, with keys artist, title
and tracks, the artist field must be a string, title is a string, both
with at least 1 character, tracks must be an array containing one element,
and it contains hashes with the key name and duration. name is of type
string and `$duration` is

```perl
my $duration = t_matchf(qr/\A(\d\d):(\d\d)\z/, sub($min,$sec) {
    return if $min >= 60;
    return if $sec >= 60;
    return 1;
});
```

I programmed it and heavily use **Combinators**. A technique i really fell in
love with. Code is usually very short, you can do a lot of things fast with
it, and the performance is quite "good enough". When more is needed you still
can transfer it to a full data-approach only, but it costs more development time.

But anyway, let's go deeper into some LISP land. The above is actually code
that executes. But do you know how you can make any function call lazy?

By transforming it into an array/data.

Let's say you call

    my $ret = function($x, $y, $z);

then you can transform it to.

    my $lazy = ['function', $x, $y, $z];

at least in a dynamic-typed language it is that easy. In a static-typed OO language
like C#, Java you probably must cast all things to `object` to make a dynmaic-typed
language out of it.

Once you have that you can write a function that takes such an array/data and executes
it back again. In Perl it's easy.

```perl
sub run($data) {
    my ($func,@args) = @$data;
    no strict 'refs';
    return *{$func}->(@args);
}

sub hello($arg) {
    say "Hello, $arg!";
}

my $data = ['hello', "World"];
run $data; # prints: "Hello, World!"
```

But instead of calling the global installed functions that are installed
in Perl, here `hello`, we also could change `run` to pass it a so called
**Dispatch-Table**.

```perl
sub run($dispatch, $data) {
    my ($func,@args) = @$data;
    my $fn = $dispatch->{$func};
    if ( defined $fn ) {
        return $fn->(@args);
    }
    Carp::croak "Unknown function: $func\n";
}

my $dispatch = {
    'hello' => sub ($arg) {
        say "Hello, $arg!";
    },
};

my $data = ['hello', "World"];
run $dispatch, $data;
```

now we have a small evaluator for executing all kind of data. To install new
functions we just need to install them into the hash-table.

But never mind, let's look what happens when i apply just the transformation
from code to data to the whole code i started with. Then i get.

```perl
my $album =
    ['t_hash',
        ['t_with_keys', qw/artist title tracks/],
        ['t_keys',
            artist => ['t_str', t_min 1],
            title  => ['t_str', t_min 1],
            tracks =>
                ['t_array',
                    ['t_min', 1],
                    ['t_of',
                        ['t_hash',
                            ['t_with_keys', qw/name duration/],
                            ['t_keys',
                                name     => ['t_str'],
                                duration => $duration]]]]]];
```

now it's just an array of arrays in any kind of depth. This is also called
an AST (Abstract Syntax Tree).

Now let's assume we just create a magic language. We just cleanup the code
for some useless characters, we just assume it's a new language. First we
remove all `,` because they have no additional meaning over an empty
whitespace we anyway must have.

```
my $album =
    ['t_hash'
        ['t_with_keys' qw/artist title tracks/]
        ['t_keys'
            artist => ['t_str' t_min 1]
            title  => ['t_str' t_min 1]
            tracks =>
                ['t_array'
                    ['t_min' 1]
                    ['t_of'
                        ['t_hash'
                            ['t_with_keys' qw/name duration/]
                            ['t_keys'
                                'name'     => ['t_str']
                                'duration' => $duration]]]]]];
```

also the `;` is not needed. The last closing `]` tells it all. On top
we remove `=>` because with the full string quote they also do nothing.
Consider that `=>` is anyway much the same as `,`. Now we get.

```
my $album =
    ['t_hash'
        ['t_with_keys' qw/artist title tracks/]
        ['t_keys'
            artist ['t_str' [t_min 1]]
            title  ['t_str' [t_min 1]]
            tracks
                ['t_array'
                    ['t_min' 1]
                    ['t_of'
                        ['t_hash'
                            ['t_with_keys' qw/name duration/]
                            ['t_keys'
                                'name'     ['t_str']
                                'duration' $duration]]]]]]
```

Next, we remove the `qw//`. We can create so called **atoms**. Those are words
that stands on their own. They are not strings. Just a word that is only equal
exactly to itself. We get those by just adding a `'` in front of them.

```
my $album =
    [t_hash
        [t_with_keys 'artist 'title 'tracks]
        [t_keys
            'artist [t_str [t_min 1]]
            'title  [t_str [t_min 1]]
            'tracks
                [t_array
                    [t_min 1]
                    [t_of
                        [t_hash
                            [t_with_keys 'name 'duration]
                            [t_keys
                                'name     [t_str]
                                'duration $duration]]]]]]
```

when we then finally replace the curly braces with parenthesis

```
my $album =
    (t_hash
        (t_with_keys 'artist 'title 'tracks)
        (t_keys
            'artist (t_str (t_min 1))
            'title  (t_str (t_min 1))
            'tracks
                (t_array
                    (t_min 1)
                    (t_of
                        (t_hash
                            (t_with_keys 'name 'duration)
                            (t_keys
                                'name     (t_str)
                                'duration $duration))))))
```

we have valid Lisp code. Except the first line `my $album =`. This is exactly
code like the Perl code we started with. It executes and creates something.
The interesting part on Lisp is the way how you turn this to data. Exactly like
in Perl we can turn it to a data representation. Lisp solves it that you just can
add a tick `'` in front of everything and the whole structure turns into data.

```racket
(define album
    '(t_hash
        (t_with_keys artist title tracks)
        (t_keys
            artist (t_str (t_min 1))
            title  (t_str (t_min 1))
            tracks
                (t_array
                    (t_min 1)
                    (t_of
                        (t_hash
                            (t_with_keys name duration)
                            (t_keys
                                name     (t_str)
                                duration duration)))))))
```

Much easier compared to Perl.

We also only need a single `'` now because anyway any **bareword** (how Perl
developer name it) now turns into an *atom*.

You also now what is intersting. When we have data there is no name clashing
of functions. Because now they are just names without a real code reference behind
it. The evaluator you choose decides in the end which code for every
name is executed. Because of this we also could rename the function and remove
the namespacing `t_`.

```racket
(define album
  '(hash
    (with_keys artist title tracks)
      (keys
        artist (str (min 1))
        title  (str (min 1))
        tracks
          (array
            (min 1)
            (of
              (hash
                (with_keys name duration)
                (keys
                  name     (str)
                  duration duration)))))))
```

Now i have a clean data-structure that actually represents code that you can
run. But you also can query/change that data before you execute it. For example
do some optimization and so on.
