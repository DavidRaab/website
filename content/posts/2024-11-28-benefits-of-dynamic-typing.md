---
layout:  post
title:   "Benefits of Dynamic Typing"
slug:    benefits-of-dynamic-typing
date:    2024-11-28T00:00:00
lastmod: 2024-11-28T00:00:00
tags:    [perl,dynamic-typed]
description: Advantages of using a dynamic typed language as a dynamic typed language
---

This is not a post about static typing vs. dynamic typing. In the several
past years I have worked with both and see advantages and disadvantages in
both dynamic and static typing. While this might be an interesting topic
on its own, this is maybe for a future post.

No matter what you like or dislike more, some languages are built from
the core being either static or dynamic typed. Perl, including other languages
like Python, Ruby, JavaScript, Lisp, Lua, ... are built as a dynamic
typed language. But instead of embracing dynamic typing and using it's
advantages many programers actually work with a dynamic typed language
as if it would be static-typed and adding type-checking either through
libraries or through obsessive use of classes.

In Perl this is no exception. In this post I will show you what happens
when you use a dynamic typed language like a dynamic typed language, not
like a static typed language with missing type-checking.

# Creating a Point

For discussion we pick a ~~simple~~ easy example. We just want to create
a Point that contains an X and Y value. Sometimes also called a `Vector2`.

One way, and probably one of the most picked way in many languages today
is to create a class out of it. In Perl this looks like this.

```perl
package PointP;
use 5.036;

sub new($class, $x, $y) {
    return bless({x => $x, y => $y}, $class);
}

sub x($self, $x=undef) {
    if ( defined $x ) {
        $self->{x} = $x;
    }
    return $self->{x};
}

sub y($self, $y=undef) {
    if ( defined $y ) {
        $self->{y} = $y;
    }
    return $self->{y};
}

sub to_string($self) {
    sprintf "Point(%f,%f)", $self->x, $self->y;
}
```

Here we define a class `Point` with methods `x` and `y` that act as a getter
and setter at the same time. When you have written many many classes like
this, like me, then maybe you gona go crazy of how stupid that code is
and how much boilerplate it is to always write those getter/setters. One
answer to that in Perl was `Moose`. So we also can write the class like
this in Moose.

```perl
package PointM;
use 5.036;
use Moose;

has 'x' => (is => 'rw', isa => 'Num' );
has 'y' => (is => 'rw', isa => 'Num' );

sub to_string($self) {
    sprintf "Point(%f,%f)", $self->x, $self->y;
}

__PACKAGE__->meta->make_immutable;
```

Here is how both classes are used.

```perl
my $p1 = PointP->new(1, 2);
print $p1->to_string, "\n";

my $p2 = PointM->new(x => 1, y => 2);
print $p2->to_string, "\n";
```

So what is wrong about this two examples you maybe ask? Well, a lot, so let's
begin how a truly dynmic typed version looks. Here is the code you usually
would write when you embrace dynamic-typing.

```perl
my $point = { x => 1, y => 2 };
```

You see what the problem is? That whole classes stuff usually creates just
a lot of boilerplate mess that aren't really needed. I mean consider the
following.

All that those classes do is basically just wrapping a Hash. You write those
getter/setters and what do they do? Well they just set a field in a Hash
or get you a field from a Hash. But what is wrong just using a Hash instead?

I mean let's look at how to get a field. Here is a class version compared
to the direct version using a Hash.

```perl
my $x = $point->x;
my $x = $point->{x};
```

and here is setting a value.

```perl
my $x = $point->x;
my $x = $point->{x};
```

So all you get from those 10-20 lines class implementation is that you can
omit those brace `{}` for either getting or setting a value?

Now there is the next problem. Did you ever Benchmarked how much performance
you will lose using a class instead of directly using a Hash? Here is
a benchmark for initalization.

```perl
cmpthese(-1, {
    perl => sub {
        for ( 1 .. 1_000 ) {
            my $p = PointP->new(1, 2);
        }
    },
    moose => sub {
        for ( 1 .. 1_000 ) {
            my $p = PointM->new(x => 1, y => 2);
        }
    },
    hash => sub {
        for ( 1 .. 1_000 ) {
            my $h = { x => 1, y => 2 };
        }
    }
});
```

Here are the results.

```perl
moose  640/s    --  -80%  -92%
perl  3139/s  391%    --  -60%
hash  7802/s 1119%  149%    --
```

So the pure perl version is roughly 5 times faster compared to moose. And
just the Perl class to just using a hash is 2.5 times the performance.
How about getting and setting a field?

Here is the Benchmark code.

```perl
my $pp = PointP->new(1,2);
my $pm = PointM->new(x => 1, y => 2);
my $h  = { x => 1, y => 2 };

printf "get\n";
cmpthese(-1, {
    perl => sub {
        for ( 1 .. 1_000 ) {
            my $x = $pp->x;
        }
    },
    moose => sub {
        for ( 1 .. 1_000 ) {
            my $x = $pm->x;
        }
    },
    hash => sub {
        for ( 1 .. 1_000 ) {
            my $x = $h->{x};
        }
    }
});

printf "set\n";
cmpthese(-1, {
    perl => sub {
        for ( 1 .. 1_000 ) {
            $pp->x(1);
        }
    },
    moose => sub {
        for ( 1 .. 1_000 ) {
            $pm->x(1);
        }
    },
    hash => sub {
        for ( 1 .. 1_000 ) {
            $h->{x} = 1;
        }
    }
});
```

and here are the results.

```
get
         Rate  perl moose  hash
perl   6981/s    --  -19%  -75%
moose  8650/s   24%    --  -69%
hash  28183/s  304%  226%    --

set
         Rate moose  perl  hash
moose  2739/s    --  -61%  -92%
perl   7045/s  157%    --  -79%
hash  34133/s 1146%  385%    --
```

Setting in Moose is so much more worse compare to the pure perl version because
it additionally does a type-check for number. And of course that happens
at runtime. That's what **dynamic typing** actually means. But compared
to directly getting/setting a value in a hash both version are just
extraordinaly bad.

So at this point I say this solutions have already two problems.

1. Too much boilerplate code.
2. Too much of a performance hit.

# Can we improve?

There are reasons why people start using classes and I go into that further
now. But let's see how creating a point is usually solved for example in typical
Scheme.

```scheme
(define (make-point x y)
  (cons x y))
(define (point-x p)
  (car p))
(define (point-y p)
  (cdr p))
```

When you are unfamiliar with Scheme. Here is how that code translates to Perl.

```perl
sub make_point($x,$y) {
    return [$x,$y];
}
sub point_x($point) {
    return $point->[0];
}
sub point_y($point) {
    return $point->[1];
}
```

In some sense this is very familiar to the class/object-oriented versions.
Only with the difference that it is by far a lot less code. Instead of so
called *methods* you just create namespaced functions. What is interesting
is the following.

A function like `make_point()` creates a Point. But the structure it creates
isn't *taged/flaged* as a point. It's structure is actually just an array/list
with two fields, and that's it.

You as a programmer have to remember if that is a point or not and if that
value is valid or not. But also the function `point_x` and `point_y`
are in some sense *special* as they don't really check it's structure or check
if they are valid points. They just expect an array and either return
the first or second value. And that's what dynamic typing actually is about.

At least, let's check how fast those versions are.

```perl
sub point_set_x($point, $x) {
    $point->[0] = $x;
}

# Scheme version
my $p = make_point(1,2);
cmpthese(-1, {
    init => sub { my $p = make_point(1,2) },
    get  => sub { my $x = point_x($p)     },
    set  => sub { point_set_x($p)         },
});
```

```
        Rate init  set  get
init  8296/s   -- -20% -43%
set  10383/s  25%   -- -28%
get  14491/s  75%  40%   --
```

Here we can see that initalization is around the same, while getting/setting
is actually faster. But most of those performance benefits are because we
use an Array instead of an Hash. Let's look what happens when we change
the implemention to a Hash (I omit the code here).

```
        Rate init  set  get
init  5493/s   -- -45% -59%
set   9998/s  82%   -- -25%
get  13398/s 144%  34%   --
```

Everything at least goes up. Initialization is nearly two times faster
compared to creating an object. But also getting/setting is faster. It's
usually faster because calling functions is usually faster as calling methods
on objects and dividing a getter/setter into two parts can make the code faster as
they are no branching logic that decides what todo or always do both things.

So, here is the question. Why not stick to such a function based solution?
It usually is by far more less code and it also is faster.

But let's go a little bit deeper to see some more difference.

# Adding behaviour

Let's assume the following. We want to add a function `add` that takes two
points and adds them together. I like immutable programming, so even if
the data-structure (Array,Hash) is mutable it means instead of mutating
on of its input, we return the result as something new.

Now it starts to become interesting, because we can pick two solutions
to implement it. Here is the first solution of the Scheme like version.

```perl
sub point_add_1($p1, $p2) {
    make_point(
        point_x($p1) + point_x($p2),
        point_y($p1) + point_y($p2),
    );
}
```

This solution is similar as the following solution in the object-oriented code.

```perl
sub add1($self, $other) {
    return PointP->new(
        $self->x + $other->x,
        $self->y + $other->y,
    );
}
```

So what we are doing in both cases is to not directly access the hash
to extract `x` and `y`, instead we are used to use *methods* to access those
fields. Well, methods that aren't that fast to begin with. So we now have
four of those getter methods we call to just get 4 fields.

Here is another way how to implement `add` instead.

```perl
sub point_add_2($p1, $p2) {
    make_point(
        $p1->{x} + $p2->{x},
        $p1->{y} + $p2->{y},
    );
}

sub add2($self, $other) {
    return PointP->new(
        $self->{x} + $other->{x},
        $self->{y} + $other->{y},
    );
}
```

So instead of using functions/getters we directly use the hash. Just let's
compare how fast those different implementations are.

```perl
my $pp1 = PointP->new(1,2);
my $pp2 = PointP->new(2,1);

my $sp1 = make_point(1,2);
my $sp2 = make_point(2,1);

cmpthese(-1, {
    class_1 => sub { $pp1->add1($pp2)        for 1 .. 1000 },
    class_2 => sub { $pp1->add2($pp2)        for 1 .. 1000 },
    hash_1  => sub { point_add_1($sp1, $sp2) for 1 .. 1000 },
    hash_2  => sub { point_add_2($sp1, $sp2) for 1 .. 1000 },
});
```

```
class_1 1047/s      --    -34%    -49%    -63%
hash_1  1585/s     51%      --    -23%    -44%
class_2 2055/s     96%     30%      --    -28%
hash_2  2844/s    172%     79%     38%      --
```

As expected. The implementations that uses a function/method to access `x/y`
are the slowest. But still amazing that a function based solution is already
50% faster than an object-orientet solution.

Otherwise directly accessing the hash fields makes the code around two times
faster compared to it's similar solution. `class_2` is around two times
faster as `class_1`. Same goes from `hash_1` to `hash_2`. From the slowest
to the fastest version we have a 2.7x performance improvement.

Here is btw. the Moose solution.

```perl
sub add($self, $other) {
    return PointM->new(
        x => $self->x + $other->x,
        y => $self->y + $other->y,
    );
}
```

and the benchmark.

```perl
my $pm1 = PointM->new(x => 1, y => 2);
my $pm2 = PointM->new(x => 1, y => 2);

timethis(-3, sub {
    $pm1->add($pm2) for 1 .. 1000;
});
```

this prints

```perl
timethis for 3:  3 wallclock secs ( 3.20 usr +  0.00 sys =  3.20 CPU) @ 462.19/s (n=1479)
```

So with roughly `460` calls per seconds we nearly have half of the speed compared to the *slowest* version `class_1` while `hash_2` is around **6 times faster**.

How about a faster `Moose` version? Well in Moose it is usually discouraged
to directly access the fields. I guess *proper* object-orientation is the reason
for that. And sure you want those checks and behaviour that `Moose` typically
add. So no fast version for you!

Some call it *prober object-orientation*. I just call it bullshit.

# Advantages and Disadvantages

It's something i cannot stress often enough. Everything has it's advantages
and disadvantages. The same goes for which solution you pick.

When you use function/methods to access the fields of `x` and `y` then
there is one advantage. You actually can change the internal structure
on how that object is represented. For example you change the hash version
to an Array. Because Array access are faster. Doing that change would
mean the `add1` solutions would still continue to work. While the `add2`
solutions will break.

So while the `add1` solution has an advantage when you change the internal
representation it comes at a big performance hit. So those `add2` solutions
are faster but when you change the internal representation you also
must change those functions to reflect the lastest changes.

But here is the great **Buuuuut**. How often do you fuckingly change the internal
representation? Are you stupid or something like that?

You usually pick one representation and then you stick with. I guess once you
settled to one representation you maybe will stick for it for the rest of your
life. So there is no point in *optimizing* a case that usually never will
happen but causing you big performance hits for your code.

And even when you change the internal representation maybe every 10 years,
yeah you also must change `add`. That's not a big deal.

# Extending Point

Object-orientation is often teached that you can use crappy inheritance
to extend your objects. So let's do that. We create a `Point3D` that
we extend from `PointP`.

```perl
package Point3D;
use 5.036;
our @ISA = 'PointP';

sub new($class, $x, $y, $z) {
    return bless({ x => $x, y => $y, z => $z }, $class);
}

sub z($self, $z=undef) {
    if ( defined $z ) {
        $self->{z} = $z;
    }
    return $self->{z};
}

sub add($self, $other) {
    return Point3D->new(
        $self->{x} + $other->{x},
        $self->{y} + $other->{y},
        $self->{z} + $other->{z},
    );
}
```

and now we can do:

```perl
my $p1 = Point3D->new(1,2,3);
my $p2 = Point3D->new(3,2,1);

my $p3 = $p1->add($p2);
printf "P3{%f,%f,%f}", $p3->x, $p3->y, $p3->z;
```

okay, works nice. But how does our Scheme inspired code will look like
with the same behaviour?

```perl
sub make_point($x,$y)     { return { x => $x, y => $y }          }
sub make_point3($x,$y,$z) { return { x => $x, y => $y, z => $z } }
sub point_x($point)       { $point->{x} }
sub point_y($point)       { $point->{y} }
sub point_z($point)       { $point->{z} }
sub point_str($point)     {
    return defined $point->{z}
         ? sprintf "2D: %f,%f,%f", $point->{x}, $point->{y}, $point->{z}
         : sprintf "3D: %f,%f",    $point->{x}, $point->{y};
}
sub point_add($p1, $p2) {
    my $p = {
        x => $p1->{x} + $p2->{x},
        y => $p1->{y} + $p2->{y},
    };
    if ( defined $p1->{z} && defined $p1->{z} ) {
        $p->{z} = $p1->{z} + $p2->{z};
    }
    return $p;
}
```

Now, let's look at it. Obviously `point_x` and `point_y` don't need to change
anymore. Obviously because all they do is just return an `x` field from a
hash. It works by passing any hash that has `x` field. Very reusable and obviously
not really needed at all. `$hash->{x}` does the same.

We actually could create a `point3d_add` function and differentiate between
`point_add` and `point3d_add` but, why? The whole purpose of dynamic typing
is also to check for structures you pass in and depending on the structure
you can decide different behaviour. Here `point_add` works with both.

You can pass it 2D points and 3D points. You even can mix it and one
can be a 2D point and the other being a 3D point. Not even that, it works
with any hash that just have those fields.

Here are the ways you can use those functions.

```perl
# Point3D
my $p1 = Point3D->new(1,2,3);
my $p2 = Point3D->new(3,2,1);
my $p3 = $p1->add($p2);

# Scheme like versions
my $p4 = point_add(
    Point3D->new(1,2,3),
    { x => 3, y => 2, z => 1 },
);
my $p5 = point_add(
    {x => 1, y => 2},
    {x => 1, y => 2, z => 3},
);
my $p6 = point_add(
    make_point(1,2),
    {x => 1, y => 2, z => 3},
);
my $p7 = point_add(make_point3(1,2,3), make_point3(3,2,1));
my $p8 = point_add(make_point(1,2),    make_point3(3,2,1));

printf "%s\n", point_str($p1); # 3D: 1.000000,2.000000,3.000000
printf "%s\n", point_str($p2); # 3D: 3.000000,2.000000,1.000000
printf "%s\n", point_str($p3); # 3D: 4.000000,4.000000,4.000000
printf "%s\n", point_str($p4); # 3D: 4.000000,4.000000,4.000000
printf "%s\n", point_str($p5); # 2D: 2.000000,4.000000
printf "%s\n", point_str($p6); # 2D: 2.000000,4.000000
printf "%s\n", point_str($p7); # 3D: 4.000000,4.000000,4.000000
printf "%s\n", point_str($p8); # 2D: 4.000000,4.000000
```

Isn't it somehow awesome that we also can pass it `PointP` and `Point3D`
objects? Create the hashes directly instead of calling `make_point` or
`make_point3` and also mix 2D and 3D points? And besides all of that we
even write less code for all of that.

How about performance?

```perl
my $o1 = Point3D->new(1,2,3);
my $o2 = Point3D->new(3,2,1);
my $s1 = make_point3(1,2,3);
my $s2 = make_point3(3,2,1);

cmpthese(-1, {
    point3d => sub { $o1->add($o2)       for 1 .. 1000 },
    scheme  => sub { point_add($s1, $s2) for 1 .. 1000 },
});
```

here are the result.

```
          Rate point3d  scheme
point3d 1778/s      --    -28%
scheme  2461/s     38%      --
```

`40%` is maybe not the greatest jump, but also not the worst. Considering
that writing code this way offers you:

1. Less code
2. More flexible
3. Higher performance

only leaves with one question open.

Why do you ever want to write object-oriented code with classes and a pseudo
type-checking if when you start using a dynamic typed language like a
dynamic typed language has so much more benefits?
