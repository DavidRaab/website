---
title: "Draft Closures"
date: 2023-01-20T19:14:16+01:00
draft: true
---

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