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
its own copy of `y`. In seems useless to create `y` in the
function, and this is right, but we can change the example being
more useful.

```perl
sub add($x) {
    return sub($y) {
        return $x + $y;
    }
}

my $add1  = add(1);
my $add10 = add(10);

my $a = $add1->(10);  # 11
my $b = $add10->(10); # 20
```

Now the `add` function returns a function, but each time we call `add` it keeps
holding a copy of the argument `x`. So we can create two distinct functions
that add `1` or `10` to its argument `y`.

While being more usefull i still guess you probably wonder, if you never
encounter this before, why you should do this. Why not create a function that adds
two numbers directly. So here comes an even more useful example.

Consider we want to create a `range` function that iterates from a start to
a given finish. We let those function return another function, and every time
we call this function we get the next value. This is also called a **iterator**.


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
    // a function is returned, but every function has access it its own
    // unique mutable current. This is called a closure.
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
    # a function is returned, but every function has access to its own unique
    # $current. This is called a closure.
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
    // a function is returned, but every function has access to its own
    // unique current. This is called a closure.
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
  ; a function is returned, but every function has access to its own
  ; unique current. This is called a closure.
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

The `range` function defines `current` and uses `start` is initial value. Every
function created by `range` has its own copy of `current`. So whenever
we call the function that is created by `range` it just iteratres from start
to stop.

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