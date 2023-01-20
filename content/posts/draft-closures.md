---
title: "Draft Closures"
date: 2023-01-20T19:14:16+01:00
draft: true
---

<div class="code-toggle">
<div class="buttons">
<button data-lang="fsharp">F#</button>
<button data-lang="perl">Perl</button>
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
            current <- current + 1
            Some current
        else
            None
```

</div>
<div class="code-perl">

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

</div>
</div>