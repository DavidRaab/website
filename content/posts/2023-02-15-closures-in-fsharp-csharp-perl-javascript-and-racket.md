---
title: "Closures in F#, C#, Perl, JavaScript and Racket"
date: 2023-02-15
tags: [fsharp,csharp,perl,js,racket,FPvsOO,closure,iterator]
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
    (+ x y)
)
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

Everytime `add10` is called a new function is created and returned.

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

Everytime `add10` is called a new function is created and returned.

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
    return sub($y) { $x + $y };
}
```

Everytime `add10` is called a new function is created and returned.

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

Everytime `add10` is called a new function is created and returned.

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
    (lambda (y) (+ x y))
)
```

Everytime `add10` is called a new function is created and returned.

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
`x` is now passed as an argument it is now possible to create multiple different
`add` functions where each function has access to its own different `x`.

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

Because `x` is now passed as an argument it is now possible to create multiple
different `add` functions where each function has access to its own different `x`.

```csharp
var add1  = add(1);
var add10 = add(10);

var a = add1(10);  // 11
var b = add10(10); // 20
```

</div><div class="code perl">

```perl
sub add($x) {
    return sub($y) { $x + $y };
}
```

Because `x` is now passed as an argument it is now possible to create multiple
different `add` functions where each function has access to its own different `x`.

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

Because `x` is now passed as an argument it is now possible to create multiple
different `add` functions where each function has access to its own different `x`.

```js
const add1  = add(1);
const add10 = add(10);

const a = add1(10);  // 11
const b = add10(10); // 20
```

</div><div class="code racket">

```racket
(define (add x)
    (lambda (y) (+ x y))
)
```

Because `x` is now passed as an argument it is now possible to create multiple
different `add` functions where each function has access to its own different `x`.

```racket
(define add1  (add 1))
(define add10 (add 10))

(define a (add1  10)) ; 11
(define b (add10 10)) ; 20
```

</div>
</div>

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
as needed to execute a function is called **Partial Application**. In
C#, Perl, JavaScript and Racket you must do this kind of transformation
explicitly.

<div class="code-toggle">
<div class="buttons">
<button data-lang="csharp">C#</button>
<button data-lang="perl">Perl</button>
<button data-lang="js">JavaScript</button>
<button data-lang="racket">Racket</button>
<button data-lang="fsharp">F#</button>
</div>

<div class="code csharp">

```csharp
public static Func<int,Func<int,int>> Add(int x) {
    return (y) => {
        return (z) => {
            return x + y + z;
        };
    };
}
```

used like this:

```csharp
var add10 = Add(6)(4);    // Func<int,int>
var num   = Add(6)(4)(5); // 15
```

</div><div class="code perl">

```perl
sub add($x) {
    return sub($y) {
        return sub($z) {
            return $x + $y + $z;
        }
    }
}
```

used like this:

```perl
my $add10 = add(6)->(4);    # sub
my $num   = add(6)->(4)(5); # 15
```

</div><div class="code js">

```js
function add(x) {
    return function(y) {
        return function(z) {
            return x + y + z;
        }
    }
}
```

used like this:

```js
const add10 = add(6)(4);    // function
const num   = add(6)(4)(5); // 15
```

</div><div class="code racket">

```racket
(define (add x)
    (lambda (y)
        (lambda (z)
            (+ x y z)
        )
    )
)
```

used like this:

```racket
(define add10 ((add 6) 4))    ; lambda
(define num   ((add 6) 4) 5)) ; 15
```

</div><div class="code fsharp">

```fsharp
let add x y z = x + y + z
```

used like this.

```fsharp
let add10 = add 6 4         // int -> int
let numa  = add 6 4 5       // 15
let numb  = (((add 6) 4) 5) // 15
```

This is the reason why F# doesn't use parenthesis for the arguments **and** they
are also not required like in Racket (or another LISP-like language).

</div></div>

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
    // This is the state
    int? current = start;
    // A function is returned and has has access to its
    // own unique mutable current variable.
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
            [else #f]
        )
    )
)
```

</div>
</div>

The `range` function defines `current` and uses `start` as the initial value.
Every function created by `range` has its own copy of `current`. So whenever
we call the function that is returned by `range`, it just iterates from `start`
to `stop`.

First we create a helper function to iterate through an iterator. This way we
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

It is now easy to create multiple iterators and iterate through them.

```fsharp
let a = range  1 10
let b = range 20 80

a |> iter (fun x -> printfn "%d" x) // prints 1 to 10
b |> iter (fun x -> printfn "%d" x) // prints 20 to 80
```

</div><div class="code csharp">

```csharp
public delegate int? RangeDelegate();
public static void Iter(RangeDelegate i, Action<int> f) {
    var x = i();
    while ( x != null ) {
        f(x.Value);
        x = i();
    }
}
```

It is now easy to create multiple iterators and iterate through them.

```csharp
var a = Range( 1,10);
var b = Range(20,80);

Iter(a, x => Console.WriteLine(x)); // prints 1 to 10
Iter(b, x => Console.WriteLine(x)); // prints 20 to 80
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

It is now easy to create multiple iterators and iterate through them.

```perl
my $a = range( 1, 10);
my $b = range(20, 80);

iter($a, sub($x) { say $x }); # prints 1 to 10
iter($b, sub($x) { say $x }); # prints 20 to 80
```

</div><div class="code js">

```js
function iter(i, f) {
    let x = i();
    while ( x !== undefined ) {
        f(x);
        x = i();
    }
}
```

It is now easy to create multiple iterators and iterate through them.

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
        [x (f x) (iter i f)]
    )
)
```

It is now easy to create multiple iterators and iterate through them.

```racket
(define a (range  1 10))
(define b (range 20 80))

(iter a (lambda (x) (displayln x))) ; prints 1 to 10
(iter b (lambda (x) (displayln x))) ; prints 20 to 80
```

</div>
</div>

*Closures* are tied to *functional programming*. But let's assume you have an
object-oriented language and want to achieve the same (without using its
functional features). How do *closures* translate to object-oriented programming?

**Answer:** They are just classes with fields. A closure is the same as an object
that contains data with a single method you can call.

This is how you implement `add10`.

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
type Add10() =
    let mutable x = 10

    member this.Call y =
        x + y
```

Like in the closure example it is now possible to create `f` and `g` and both
objects will have its own `x` field.

```fsharp
let f = new Add10()
let g = Add10()    // new is optional

let a = f.Call(20) // 30
let b = g.Call(20) // 30
```

</div><div class="code csharp">

```csharp
public class Add10 {
    private int x;
    public Add10() {
        this.x = 10;
    }
    public int Call(int y) {
        return this.x + y;
    }
}
```

Like in the closure example it is now possible to create `f` and `g` and both
objects will have its own `x` field.

```csharp
var f = new Add10();
var g = new Add10();

var a = f.Call(20); // 30
var b = f.Call(20); // 30
```

</div><div class="code perl">

```perl
package Add10;
use v5.36;

sub new($class) {
    bless {
        x => 10,
    }, $class;
}

sub call($self, $y) {
    return $self->{x} + $y;
}
```

Like in the closure example it is now possible to create `f` and `g` and both
objects will have its own `x` field.

```perl
my $f = Add10->new;
my $g = Add10->new;

my $a = $f->call(20); # 30
my $b = $g->call(20); # 30
```

</div><div class="code js">

```js
function Add10() {
    this.x = 10;
}

// I am using invoke because JavaScript already has a global call method
Add10.prototype.invoke = function(y) {
    return this.x + y;
}
```

Like in the closure example it is now possible to create `f` and `g` and both
objects will have its own `x` field.

```js
const f = new Add10();
const g = new Add10();

const a = f.invoke();
const b = f.invoke();
```

</div><div class="code racket">

```racket
(define Add10%
  (class object%
    (super-new)
    (field [x 10])

    (define/public (call y)
      (+ x y)
    )
  )
)
```

Like in the closure example it is now possible to create `f` and `g` and both
objects will have its own `x` field.

```racket
(define f (new Add10%))
(define g (new Add10%))

(define a (send f call 20))
(define b (send f call 20))
```

</div></div>

Like in the example with the closure it seems useless to have `x` as a field,
but we can make it configurable by passing it to the constructor. This would
be `add`.


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
type Add(x) =
    let mutable x = x

    member this.Call y =
        x + y
```

It doesn't really matter how that one-method is named. Use `Run`, `Execute`,
`Invoke`, `Call` or any other synonyms for saying *function*.

```fsharp
let add1  = Add(1);
let add10 = Add(10);

let a = add1.Call(10);  // 11
let b = add10.Call(10); // 20
```

</div><div class="code csharp">

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

It doesn't really matter how that one-method is named. Use `Run`, `Execute`,
`Invoke`, `Call` or any other synonyms for saying *function*.

```csharp
var add1  = new Add(1);
var add10 = new Add(10);

var a = add1.Call(10);  // 11
var b = add10.Call(10); // 20
```

</div><div class="code perl">

```perl
package Add;
use v5.36;

sub new($class, $x) {
    return bless { x => $x }, $class;
}

sub call($self, $y) {
    return $self->{x} + $y;
}
```

It doesn't really matter how that one-method is named. Use `Run`, `Execute`,
`Invoke`, `Call` or any other synonyms for saying *function*.

```perl
my $add1  = Add->new(1);
my $add10 = Add->new(10);

my $a = $add1->call(10);  # 11
my $b = $add10->call(10); # 20
```

</div><div class="code js">

```js
function Add(x) {
    this.x = x;
}

// I am using invoke because JavaScript already has a global call method
Add.prototype.invoke = function(y) {
    return this.x + y;
}
```

It doesn't really matter how that one-method is named. Use `Run`, `Execute`,
`Invoke`, `Call` or any other synonyms for saying *function*.

```js
var add1  = new Add(1);
var add10 = new Add(10);

var a = add1.invoke(10);  // 11
var b = add10.invoke(10); // 20
```

</div><div class="code racket">

```racket
(define Add%
  (class object%
    (super-new)
    (init-field x)

    (define/public (call y)
      (+ x y)
    )
  )
)
```

It doesn't really matter how that one-method is named. Use `Run`, `Execute`,
`Invoke`, `Call` or any other synonyms for saying *function*.

```racket
(define add1  (new Add% [x 1]))
(define add10 (new Add% [x 10]))

(define a (send add1  call 10)) ; 11
(define b (send add10 call 10)) ; 20
```

</div></div>

This would be the iterator `range`.

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
type Range(start, stop, ?current) =
    let mutable current = defaultArg current start

    // only get Properties
    member val Start   = start
    member val Stop    = stop
    member val Current = current

    member this.Next () =
        if current <= stop then
            let tmp = current
            current <- current + 1
            Some tmp
        else
            None

    member this.Iter f =
        let mutable x = this.Next ()
        while Option.isSome x do
            f (x.Value)
            x <- this.Next ()
```

In the object-oriented code I make the `iter` function a method. The properties
are not really needed but still shown for comparison. Like mentioned in the
C# version it is possible to use an interface and extension methods. I also
could have used a `Range` module and use functions again, but then the `current`
field had to be `public`. I also could have created a record with a module instead
but here I wanted to show a class.

Now it becomes easy to define multiple ranges and iterate through them.

```fsharp
let a = Range(1, 5)
let b = Range(6,10)

a.Iter(fun x -> printfn "%d" x)
b.Iter(fun x -> printfn "%d" x)
```

</div><div class="code csharp">

```csharp
public class Range {
    public int  Start   { get; }
    public int  Stop    { get; }
    public int? Current { get; set; }

    public Range(int start, int stop, int? current = null) {
        this.Start   = start;
        this.Stop    = stop;
        this.Current = current.HasValue ? current : start;
    }

    public int? Next() {
        if ( this.Current <= this.Stop ) {
            return this.Current++;
        }
        return null;
    }

    public void Iter(Action<int> f) {
        var x = this.Next();
        while ( x.HasValue ) {
            f(x.Value);
            x = this.Next();
        }
    }
}
```

In the object-oriented code i make the `iter` function a method. But I also
cold have used an interface for the `Range` class and create Extensions Methods
for additional functions. This is how LINQ is implemented.

Now it becomes easy to define multiple ranges and iterate through them.

```csharp
var a = new Range(1, 5);
var b = new Range(6,10);

a.Iter(x => Console.WriteLine(x));
b.Iter(x => Console.WriteLine(x));
```

</div><div class="code perl">

```perl
package Range;
use v5.36;
use Moo;
use Types::Standard qw(Int);
use namespace::clean;

has 'start'   => ( is => 'ro', isa => Int, required => 1 );
has 'stop'    => ( is => 'ro', isa => Int, required => 1 );
has 'current' => ( is => 'rw', isa => Int, required => 0 );

sub BUILD($self, $args) {
    if ( not exists $args->{current} ) {
        $self->current($self->start);
    }
}

sub next($self) {
    if ( $self->current <= $self->stop ) {
        my $tmp = $self->current;
        $self->current($tmp+1);
        return $tmp;
    }
    return;
}

sub iter($self, $f) {
    while ( defined(my $x = $self->next) ) {
        $f->($x);
    }
    return;
}
```

In the object-oriented code i make the `iter` function a method. Now it becomes
easy to define multiple ranges and iterate through them.

```perl
my $r1 = Range->new(start => 1, stop =>  5);
my $r2 = Range->new(start => 6, stop => 10);

$r1->iter(sub ($x) { say $x });
$r2->iter(sub ($x) { say $x });
```

</div><div class="code js">

```js
function Range(start, stop, current) {
    this.start   = start;
    this.stop    = stop;
    this.current = current === undefined ? start : current;
}

Range.prototype.next = function() {
    if ( this.current <= this.stop ) {
        return this.current++;
    }
    return;
}

Range.prototype.iter = function(f) {
    let x = this.next();
    while ( x !== undefined ) {
        f(x);
        x = this.next();
    }
}
```

In the object-oriented code i make the `iter` function a method. Now it becomes
easy to define multiple ranges and iterate through them.

```js
let a = new Range(1, 5);
let b = new Range(6,10);

a.iter(x => console.log(x));
b.iter(x => console.log(x));
```

</div><div class="code racket">

```racket
(define Range%
  (class object%
    (super-new)
    (init-field start stop)
    (field [current start])

    (define/public (next)
      (cond
        [(<= current stop)
         (define tmp current)
         (set! current (add1 current))
         tmp]
        [else #f]
      )
    )

    (define/public (iter f)
      (define x (send this next))
      (cond
        [x (f x) (send this iter f)]
      )
    )
  )
)
```

In the object-oriented code i make the `iter` function a method. Now it becomes
easy to define multiple ranges and iterate through them.

```racket
(define a (new Range% [start 1] [stop  5]))
(define b (new Range% [start 6] [stop 10]))

(send a iter (lambda (x) (displayln x)))
(send b iter (lambda (x) (displayln x)))
```

</div>