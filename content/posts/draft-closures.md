---
title: "Draft Closures"
date: 2023-01-20T19:14:16+01:00
draft: true
tags: [fsharp,csharp,perl,js,racket,fp,oo,closure,iterator]
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

<div class="code fsharp">

```fsharp
let add10 y =
    let x = 10
    x + y
```

</div><div class="code csharp">

```csharp
public static int Add10(int y) {
    var x = 10;
    return x + y;
}
```

</div><div class="code perl">

```perl
sub add10($y) {
    my $x = 10;
    return $x + $y;
}
```

</div><div class="code js">

```js
function add10(y) {
    const x = 10;
    return x + y;
}
```

</div><div class="code racket">

```racket
(define (add10 y)
  (define x 10)
  (+ x y))
```

</div>
</div>

In the example the variable `x` is created only temporary when the function
is being executed. Once the function is finished the variable `x` is freed
from memory. But this can change when we return a function.

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code fsharp">

```fsharp
let add10 () =
    let x = 10
    fun y -> x + y
```

Now we can create multiple functions by calling `add10`.

```fsharp
let f = add10 ()
let g = add10 ()

let a = f 20 // 30
let b = g 20 // 30
```

</div><div class="code csharp">

```csharp
public static Func<int,int> Add10() {
    var x = 10;
    return (int y) => x + y;
}
```

Now we can create multiple functions by calling `add10`.

```csharp
var f = add10();
var g = add10();

var a = f(20); // 30
var b = g(20); // 30
```

</div><div class="code perl">

```perl
sub add10() {
    my $x = 10;
    return sub($y) {
        return $x + $y;
    }
}
```

Now we can create multiple functions by calling `add10`.

```perl
my $f = add10();
my $g = add10();

my $a = $f->(20); # 30
my $b = $g->(20); # 30
```

</div><div class="code js">

```js
function add10(x) {
    const x = 10;
    return y => x + y;
}
```

Now we can create multiple functions by calling `add10`.

```js
const f = add10();
const g = add10();

const a = f(20); // 30
const b = g(20); // 30
```

</div><div class="code racket">

```racket
(define (add10)
  (define x 10)
  (lambda (y) (+ x y)))
```

Now we can create multiple functions by calling `add10`.

```racket
(define f (add10))
(define g (add10))

(define a (f 20)) ; 30
(define b (g 20)) ; 30
```

</div>
</div>

`add10` returns a function instead of doing a calculation. Because the
function that is returned still access the variable `x` it means the
variable is still not freed from memory. In the cases above the new functions
`f` and `g` have access to its own `x` variable. Both functions
have there own copy of `x` that is not shared between them.

This seems not so useful because `x` is always `10`. But we can change the
example to be more useful.

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code fsharp">

```fsharp
let add x y = x + y
let add x   = fun y -> x + y
```

Both definitions of `add` are the same in F# because of currying. Because
`x` is now passed as an argument we can now create multiple different
`add` functions where each function has access to a different `x`.

```fsharp
let add1  = add 1
let add10 = add 10

let a = add1  10 // 11
let b = add10 10 // 20
```

</div><div class="code csharp">

```csharp
public static Func<int,int> Add(int x) {
    return (int y) => x + y;
}
```

Because `x` is now passed as an argument we can now create multiple different
`add` functions where each function has access to a different `x`.

```csharp
var add1  = add(1);
var add10 = add(10);

var a = add1(10);  // 11
var b = add10(10); // 20
```

</div><div class="code perl">

```perl
sub add($x) {
    return sub($y) {
        return $x + $y;
    }
}
```

Because `x` is now passed as an argument we can now create multiple different
`add` functions where each function has access to a different `x`.

```perl
my $add1  = add(1);
my $add10 = add(10);

my $a = $add1->(10);  # 11
my $b = $add10->(10); # 20
```

</div><div class="code js">

```js
function add(x) {
    return y => x + y;
}
```

Because `x` is now passed as an argument we can now create multiple different
`add` functions where each function has access to a different `x`.

```js
const add1  = add(1);
const add10 = add(10);

const a = add1(10);  // 11
const b = add10(10); // 20
```

</div><div class="code racket">

```racket
(define (add x)
  (lambda (y) (+ x y)))
```

Because `x` is now passed as an argument we can now create multiple different
`add` functions where each function has access to a different `x`.

```racket
(define add1  (add 1))
(define add10 (add 10))

(define a (add1  10)) ; 11
(define b (add10 10)) ; 20
```

</div>
</div>

Every time the `add` function is called, it returns a new function expecting
`y` to be passed. But every of those functions have access to its own `x`
that was passed to `add`. This way we can create multiple distinct `add` functions.

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

`add10` now *accepts* the missing argument `z`. Passing fewer arguments
as needed to execute a function is called **Partial Application**.
</div>

While being more usefull I guess you will still wonder why
you ever want to do this kind of transformation. So here comes a more
advanced example.

Consider we want to create a `range` function accepting a `start` and
`stop` value. But instead of returning a List/Array or other kind
of data, we want to return a function instead. Whenever this function
is called it returns the next value starting with `start` upto `stop`.

This concept is also called an **iterator** and we can implement it easily
with just a **closure**.

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code fsharp">

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

</div><div class="code csharp">

```csharp
public delegate int? RangeDelegate();
public static RangeDelegate Range(int start, int stop) {
    int? current = start;
    return () => {
        if ( current <= stop ) {
            return current++;
        }
        return null;
    };
}
```

</div><div class="code perl">

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

</div><div class="code js">

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

</div><div class="code racket">

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
      [else #f])))
```

</div>
</div>

The `range` function defines `current` and uses `start` as the initial value.
Every function created by `range` has its own copy of `current`. So whenever
we call the function that is returned by `range`, it just iterates from `start`
to `stop`.

First we will create a higher-order function that iterates through an iterator
that just passes every value to the user-passed function. This way we
don't need to write the logic for iteration every single time. Even if the
code is short for iteration we can easily make mistakes.


<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
</div>

<div class="code fsharp">

```fsharp
let rec iter f i =
    match i() with
    | Some value ->
        f value
        iter f i
    | None ->
        ()
```

With this function we can easily create multiple iterators and iterate
through them.

```fsharp
let a = range  1 10
let b = range 20 80

a |> iter (fun x -> printfn "%d" x) // prints 1 to 10
b |> iter (fun x -> printfn "%d" x) // prints 20 to 80
```

</div><div class="code csharp">

```csharp
public static void Iter(RangeDelegate i, Action<int> f) {
    var x = i();
    while ( x != null ) {
        Console.WriteLine(x.Value);
        x = i();
    }
}
```

With this function we can easily create multiple iterators and iterate
through them.

```csharp
var a = Range( 1,10);
var b = Range(20,80);

Iter(a, x => Console.WriteLine(x));
Iter(b, x => Console.WriteLine(x));
```

</div><div class="code perl">

```perl
sub iter($iter, $f) {
    while ( defined(my $x = $iter->()) ) {
        $f->($x);
    }
    return;
}
```

With this function we can easily create multiple iterators and iterate
through them.

```perl
my $a = range( 1, 10);
my $b = range(20, 80);

# prints 1 to 10
iter($a, sub($x) {
    printf "%d\n", $x;
});

# prints 20 to 80
iter($b, sub($x) {
    printf "%d\n", $x;
});
```

</div><div class="code js">

```js
function iter(i, f) {
    let x = i();
    while ( x !== undefined ) {
        f(x);
        x = r();
    }
}
```

With this function we can easily create multiple iterators and iterate
through them.

```js
const a = range( 1,10);
const b = range(20,80);

iter(a, x => console.log(x)); // prints 1 to 10
iter(b, x => console.log(x)); // prints 20 to 80
```

</div><div class="code racket">

```racket
(define (iter i f)
  (define x (i))
  (cond
    [x (f x) (iter i f)]))
```

With this function we can easily create multiple iterators and iterate
through them.

```racket
(define a (range  1 10))
(define b (range 20 80))

(iter a (lambda (x) (displayln x))) ; prints 1 to 10
(iter b (lambda (x) (displayln x))) ; prints 20 to 80
```

</div>
</div>

*Closures* are tied to *functional programming*. But let's assume you have an
object-oriented language and want to achieve the same. How do *closures* translate
to object-oriented programming?

Answer: They are just classes with fields. A closure is the same as an object
that contains data with a single method you can call.

This is how you implement `add10`.

```csharp
public class Add10 {
    private int x;
    public Add10() {
        this.x = 10;
    }
    public int Call(int y) {
        return x + this.x;
    }
}
```

In the class example we now have `f` and `g` and both objects will have its
own `x` field with the value of `10` like the closure had.

```csharp
var f = new Add10();
var g = new Add10();

var a = f.Call(20); // 30
var b = f.Call(20); // 30
```

We also can make `x` configurable by the user. This is `add`.

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
```

It doesn't really matter how that one-method is named. Usually `Run`, `Execute`, `Invoke`, `Call` or other synonyms for saying *function* is used.

```
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

<div class="code perl">

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

</div><div class="code csharp">

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