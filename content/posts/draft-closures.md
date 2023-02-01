---
title: "Draft Closures"
date: 2023-01-20T19:14:16+01:00
draft: true
---

Today, more and more languages supports functions as first-class values. This
means a function is just a value like any other. You can pass functions as
arguments to functions, but you are also able to create functions and return
them from functions.

Whenever this is done we have to think about the life-time of variables. Usually
all variables are lexical scoped. Consider the following example.

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code-fsharp">

```fsharp
let add10 x =
    let y = 10
    x + y
```

</div><div class="code-csharp">

```csharp
public static int Add10(int x) {
    var y = 10;
    return x + y;
}
```

</div><div class="code-perl">

```perl
sub add10($x) {
    my $y = 10;
    return $x + $y;
}
```

</div><div class="code-js">

```js
function add10(x) {
    const y = 10;
    return x + y;
}
```

</div><div class="code-racket">

```racket
(define (add10 x)
  (define y 10)
  (+ x y))
```

</div>
</div>

In the above example the variable `y` is created only temporary when the function
is being executed. Once the function is finished, the memory holding the value
`10` is freed. This can change when we return a function.

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code-fsharp">

```fsharp
let add10 () =
    let y = 10
    fun x -> x + y
```

You use it this way:

```fsharp
let f = add10 ()
let g = add10 ()

let a = f 20 // 30
let b = g 20 // 30
```

</div><div class="code-csharp">

```csharp
public static Func<int,int> Add10() {
    var y = 10;
    return (int x) => x + y;
}
```

You use it this way:

```csharp
var f = add10();
var g = add10();

var a = f(20); // 30
var b = g(20); // 30
```

</div><div class="code-perl">

```perl
sub add10() {
    my $y = 10;
    return sub($x) {
        return $x + $y;
    }
}
```

You use it this way:

```perl
my $f = add10();
my $g = add10();

my $a = $f->(20); # 30
my $b = $g->(20); # 30
```

</div><div class="code-js">

```js
function add10(x) {
    const y = 10
    return x => x + y;
}
```

You use it this way:

```js
const f = add10();
const g = add10();

const a = f(20); // 30
const b = g(20); // 30
```

</div><div class="code-racket">

```racket
(define (add10)
  (define y 10)
  (lambda (x) (+ x y)))
```

You use it this way:

```racket
(define f (add10))
(define g (add10))

(define a (f 20)) ; 30
(define b (g 20)) ; 30
```

</div>
</div>

In the above example `add10` returns a function instead of doing a calculation.
But the function that is being returned still access the variable `y`. Because
of this when we create the two functions `f` and `g` both of them will have
its own copy of `y`. It seems useless to create `y` in the
function, and this is right, but we can change the example being
more useful.

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code-fsharp">

```fsharp
let add x y = x + y
let add x   = fun y -> x + y
```

Both definitions of `add` are the same in F# because of currying. You use
it this way:

```fsharp
let add1  = add 1
let add10 = add 10

let a = add1  10 // 11
let b = add10 10 // 20
```

</div><div class="code-csharp">

```csharp
public static Func<int,int> Add(int x) {
    return (int y) => x + y;
}
```

You use it this way:

```csharp
var add1  = add(1);
var add10 = add(10);

var a = add1(10);  // 11
var b = add10(10); // 20
```

</div><div class="code-perl">

```perl
sub add($x) {
    return sub($y) {
        return $x + $y;
    }
}
```

You use it this way:

```perl
my $add1  = add(1);
my $add10 = add(10);

my $a = $add1->(10);  # 11
my $b = $add10->(10); # 20
```

</div><div class="code-js">

```js
function add(x) {
    return y => x + y;
}
```

You use it this way:

```js
const add1  = add(1);
const add10 = add(10);

const a = add1(10);  // 11
const b = add10(10); // 20
```

</div><div class="code-racket">

```racket
(define (add x)
  (lambda (y) (+ x y)))
```

You use it this way:

```racket
(define add1  (add 1))
(define add10 (add 10))

(define a (add1  10)) ; 11
(define b (add10 10)) ; 20
```

</div>
</div>

Now the `add` function returns a function, but each time we call `add` it keeps
holding a copy of the argument `x`. So we can create two distinct `add` functions
that either add `1` or `10` to its argument `y`.

<div class="info">
This style is related to <strong>currying</strong>. We do <strong>currying</strong>
whenever we turn a
function with multiple arguments into a series of one-argument functions. In
F# we could turn the following function.

```fsharp
let add x y z =
    x + y + z
```

into a curried form by writing.

```fsharp
let add x =
    fun y ->
    fun z ->
        x + y + z
```

but in F# it's not needed because all functions are curried by default.
With both definitions you can write:

```fsharp
let add10 = add 4 6
```

The above call will return a function *accepting*
argument `z`. Passing fewer arguments as needed to execute
a function is called **Partial Application**.
</div>

While being more usefull I guess you will still wonder why
you ever want to do this kind of trasformation. So here comes a more
advanced example.

Consider we want to create a `range` function accepting a `start` and
`stop` value. But instead of returning a List/Array or other kind
of data return a function. Whenever we call the function it returns
the next value starting with `start` upto `stop`.

This concept is also called an **iterator** and we can implement it easily
with just a **closure**.

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code-fsharp">

```fsharp
// range : int -> int -> (unit -> option<int>)
let range start stop =
    // this is the state
    let mutable current = start
    // A function is returned and has has access to its
    // own unique mutable current variable.
    fun () ->
        if current <= stop then
            let tmp = current
            current <- current + 1
            Some tmp
        else
            None
```

</div><div class="code-perl">

```perl
sub range($start, $stop) {
    # This is the state
    my $current = $start;
    # A function is returned and has has access to its
    # own unique mutable current variable.
    return sub {
        if ( $current <= $stop ) {
            return $current++;
        }
        else {
            return;
        }
    }
}
```

</div><div class="code-js">

```js
function range(start, stop) {
    // This is the state
    let current = start;
    // A function is returned and has has access to its
    // own unique mutable current variable.
    return function() {
        if ( current <= stop ) {
            return current++;
        }
        else {
            return;
        }
    };
}
```

</div><div class="code-racket">

```racket
(define (range start stop)
  ; This is the state
  (define current start)
  ; A function is returned and has has access to its
  ; own unique mutable current variable.
  (lambda ()
    (cond
      [(<= current stop)
       (define tmp current)
       (set! current (add1 current))
       tmp
       ]
      [else null])))
```

</div>
</div>

The `range` function defines `current` and uses `start` as the initial value.
Every function created by `range` has its own copy of `current`. So whenever
we call the function that is returned by `range`, it just iterates from `start`
to `stop`.

```perl
my $r = range(1,10);
my $s = range(20,100);

# prints 1 to 10
while( defined($x = $r->()) ) {
    say $x;
}

# prints 20 to 100
while( defined($x = $s->()) ) {
    say $x;
}
```

*Closures* are tied to *functional programming*. But let's assume you have an
object-oriented language and want to achieve the same. How do *closures* translate
to object-oriented programming?

Answer: They are just classes with fields. A closure is the same as an object that contains data with a single method you can call.

This is how you implement `add10`.

```csharp
public class Add10 {
    private int c;
    public Add10() {
        this.c = 10;
    }
    public int Call(int x) {
        return x + this.c;
    }
}

var f = new Add10();
var g = new Add10();

var a = f.Call(20); // 30
var b = f.Call(20); // 30
```

This is `add`.

```csharp
public class Add {
    private int x;
    public Add(int x) {
        this.x = x;
    }
    public Call(int y) {
        return this.x + y;
    }
}

var add1  = new Add(1);
var add10 = new Add(10);

var a = add1.Call(10);  // 11
var b = add10.Call(10); // 20
```

And finally `range`.

<div class="code-toggle">
<div class="buttons">
<button data-lang="perl">Perl</button>
<button data-lang="csharp">C#</button>
</div>

<div class="code-perl">

```perl
package Range;
use v5.36;

sub new($class, $start, $stop) {
    return bless {
        current => $start,
        stop    => $stop,
    }, $class;
}

sub next($self) {
    if ( $self->{current} <= $self->{stop} ) {
        return $self->{current}++;
    }
    return;
}

# how you use the class
package main;
my $r = Range->new(1,10);
while ( defined(my $x = $r->next) ) {
    say $x;
}
```

</div><div class="code-csharp">

```csharp
public class Range {
    private int? current;
    private int stop

    public Range(int start, int stop) {
        this.current = start;
        this.stop    = stop;
    }

    public Next() {
        if ( this.current <= this.stop ) {
            return this.current++;
        }
        return null;
    }
}

// Somewhere in your code
var r = new Range(1,10);
var n = r.Next();
while ( n != null ) {
    Console.WriteLine(n);
    n = r.Next();
}
```

</div>
</div>