---
layout:  post
title:   "GameDev: Implementing a Hash like structure"
slug:    gamedev-implementing-a-hash-like-structure
date:    2024-07-24T00:00:00
lastmod: 2024-07-24T00:00:00
tags:    [gamedev,fsharp,dictionary,optimization,ecs]
description:
---

# What Is

Recently I implemented my own data-structure to *optimize* the storage of my
components in a game. Okay its more an architecture i am exploring not a real
game at the moment. I just try to render and move as many things as possible
while i try to keep frames per second as high as possible.

For example rendering 10,000 boxes let them move in random directions, and
change direction every second to a new random direction runs at 600 fps. When
those boxes have a parent and for example the parent rotates (and thus all
10,000 boxes rotate around the parent) then fps drops down to 350 fps.

This is excaptable as transform, scale and rotation to a parent aren't that
cheap. But anyway, up to that point i just used `Dictionary<'a,'b>` to
implement the storage of my *Components*.

I am following the idea of an [ECS][ECS] so my main game component
is just an integer. Defined as

```fsharp
[<Struct>]
type Entity =
    Entity of int
```

that's it. In an ECS I need to be able to randomly add or remove Components
for every Entity so i just picked the best data-structure when you wanna be
able to randomly add/remove stuff by some key, a `Dictionary<'a,'b>`.

# The Problem

So for every Component I just have a global dictionary as a State that I
can add or remove components from.

Looks similar to this

```fsharp
module State =
    let Transform = Dictionary<Entity,Transform>()
    let Movement  = Dictionary<Entity,Movement> ()
    let Animation = Dictionary<Entity,Animation>()
    let View      = Dictionary<Entity,View>     ()
```

Only similar because I added some stuff to it, but that's still what you can
imagine how everything is stored.

While trying to optimize frames and getting the architecture right I thought
of improving that. A *Dictionary* has very good performance when it comes
to inserting and deleting by a `'Key` or `'Value`, and that's surely what i need.

But I also need good iteration performance. For example when I start rendering all
my game objects my rendering system basically iterates through all entities
that have a `View` defined, then i fetch the `Transform` and when both are available
I issue a Draw Command.

It looked like this.

```fsharp
let draw (sb:SpriteBatch) =
    let transformAndView = [|
        for view in State.View.visible.Data do
            match Storage.get view.Entity State.Transform with
            | ValueSome t -> (t,view)
            | ValueNone   -> ()
    |]

    transformAndView |> Array.sortInPlaceBy (fun (_,v) -> v.Layer)
    for transform,view in transformAndView do
        match calculateTransform transform with
        | ValueNone                           -> ()
        | ValueSome (position,rotation,scale) ->
            sb.Draw(
                texture         = view.Sprite.Texture,
                position        = position,
                sourceRectangle = view.Sprite.SrcRect,
                color           = view.Tint,
                rotation        = float32 (rotation + view.Rotation),
                origin          = view.Origin,
                scale           = view.Scale * scale,
                effects         = view.Effects,
                layerDepth      = 0f
            )
```

There is a loot of stuff i still can optimize in it, but at the moment
I start with iterating through all *Visible* components.

You see that in the for loop `State.View.visible.Data`. This is basically
a `Dictionary` containing all the game objects that have a `View` Component
attached to it, and should be rendered. It contains information about which
`Sprite` should be shown, at which rotation, scale or at which layer it should
be drawn. When I have 10,000 visible component then I also have a `Dictionary`
containing 10,000 elements.

Usually i just start by iterating through one Dictionary. Then I fetch other
components from other dictionary that are needed. In this case it is the
Transform that contains information of the World Position, Rotation and
Scale an `Entity` has.

But this `draw` function has to run every frame, including iterating through
a dictionary every frame. So I thought about optimizing it. Is a `Dictionary`
really fast enough for iterating it? Wouldn't it be better to have an
Array instead?

But the problem of an `Array` is that it has worse insertion and deletion time
of `Components`. It's not that Components gets added or removed a thousand
times per frames, probably not even per second. But in a later game
it should be high enough to worry about it, because that's the whole point of
an ECS

Usually at runtime you either add or remove components to change the behaviour
of an `Entity`. It's basically a feature to be able to constantly add and
remove Components for Entity. So those operation should be fast,
but still, iterating over a dictionary still happens more often (every frame)
and has to go through the whole dictionary.

# The Storage

So having a data-structure that has a lot more performance for iteration, but
is a little bit worse for inserting/deleting would be absolutely fine. It could
improve performance a lot. So I thought about an Idea that I somewhere
read/heard about.

Instead of using a `Dictionary` I just use an `Array`. Basically all new Components
are just getting added to the end. To get good lookup performance I just use
a `Dictionary` that stores the `index` position of the element.

So whenever I add a *Component* for an *Entity* I just add it to an array,
get the index of it and additionaly store it in a Dictionary like
`keyToIndex.Add(key, arrayIndex)`.

Whenever I want to read a Dictionary I first need to get the `index` from the
`keyToIndex` dictionary and then i can get the Component from the array by an
array index lookup.

Deletion is a little bit more tricky. Whenever I delete a Component I basically
swap the *to be deleted* Component element with the last element of the Array.
Then I remove the last element. Then I also need to update the `keyToIndex`
for the last element that I moved.

This implementation gives me still good add/deletion time. They are still O(1)
timing for add and deletion, but still have some overhead of using a normal
dictionary. But when I iterate through an array, now I get the faster/better
iteration time of an array.

But is this really faster?

# Benchmarking

So before I implemented this kind of data-structure i created some
[micro-benchmarks][micro-benchmarks] to test the iterative performance
of a *Dictionary* compared to an *Array*.

The results seemed very promising.

I created a Dictionary and an Array both containing 50,000 elements. I just
iterated through them and measured time.

* Dictionary tooked 1ms
* Array tooked 0.1ms

The result was that an Array was around 10 times faster. Actually not what
I expected. I guessed that a Dictionary would be slower, maybe at the
scale of 2x - 5x, but not 10x.

After this little benchmark I started to implement my *Storage* data-structure.

I mean, 10x better performance for iterating, must have a good performance jump,
right?

# The result

So what I did then was implemented my own data-structure that I named
`Storage<'a,'b>`. Then I replaced the `State.Transform` and `State.View` with
my data-structure. I started my game to see how much better my frames would
hopefully be and I was surprised.

In the end the frame-rates was exactly the same. Yes exactly, the frame rates
neither improved nor did they got worse (okay, not quite right).

But why? Should it not improve anything? Sure one thing is that now i iterate
the `View` fast and I still need to fetch the `Transform` and lookup is
a little bit slower compared to *Dictionary*, but before replacing `Transform`
with my `Storage` i keeped it a `Dictionary` and it also had no speed
improvements.

Then I got back to my Benchmark. One problem of Benchmarks usually is that
*we* as developers Benchmark stuff that in that case never happens in a real
benchmark, or we do Benchmarking wrong. Here it was was me.

First, i just measured iterating through the *Dictionary* only once. But
this doesn't happen in a game. In a game the same *Dictionary* is iterated
over and over again at every frame. Usually a runtime like .Net will probably
see that, issues some JIT compilation or has better caching when iterating
the same Dictionary over and over again.

So instead of iterating a Dictionary just once and get the timing i measured
the average of 1,000 iteration (for both, array and dictionary). This also
showed other results.

While the iteration for an *Array* pretty much stayed the same around *0.15ms*
the iteration speed of a Dictionary greatly improved. Now it was only *0.4ms*.

Sure, its still slower but now it's only in the range of 2x - 3x, not 10x.

But still, should it not still make any difference to the fps?

Suddenly it struck me of why there was no difference.

# Premature Optimization

There is a saying that *Premature Optimization* is the root of all evil.
Maybe it is true as it is easy to get lost into details that aren't that
important. Spending time for something that doesn't matter.

Here it was the case. Yes in those [micro-benchmarks][micro-benchmarks]
that are completely isolated from everything else I can show that an *Array*
has a lot better iterating performance compared to a *Dictionary*.

In those benchmarks I also could show that my `Storage` had the same performance
as an array, what isn't suprising as I make the internal Array just public
available and directly iterate over the Array as I need it.

But what did I tested anyway? Yes, 50,000 elements, but I usually don't
have that many shown on the screen. Even if iterating of 50,000 elements
takes *1ms* that i benchmarked before. A Dictionary would technically be
capable of rendering 50,000 Sprites at a 1,000 fps.

But in my game-demo i am already at the 300fps with just 10,000. And iterating
just 10,000 elements takes even less time as iterating 50,000.

And sure, the main performance does not come from iterating a data-structure.
The main thing that eats up performance are usually things that issue DrawCalls
to the GPU. Calculating Positions, and creating the Matrix for the Camera and
applying all those Transform Movements. This is where probably 90%+ of the
performance is lost.

Just iterating through the data-structure may it be a *Dictionary* or an *Array*
probably just takes around 1% of the time (maybe even less) to generate
a single frame to the screen.

Even if would make the iteration 10x faster. Then it doesn't mean I improved
everything by 10x. It just means that i just improved a single component
that took 1% of the total time, and improved that 1% by 90%. So when an application
takes 100 seconds to run, and I optimize 1 second of this to only run in 0.1 seconds.
Then my application still takes 99,1 seconds to run. A speed improvement that
is more over pointless.

And this is why *Premature Optimiziation is the root of all evil*.

Instead of guessing and randomly improving stuff, even when we have written
some *micro-benchmarks* doesn't mean much at all. Before we apply performance
improvement we should always first measure where optimization is needed.

We measure this by [profiling][profiling] an application. Something I didn't do.

But anyway for me doing all of this was still an elightening experience I got
a little bit deeper into *data-oriented* programming and learned a lot of stuff
while doing all of it.


# Interesting Links

* [ECS-FAQ][ECS]
* [Micro Benchmarks][micro-benchmarks]
* [Profiling][profiling]
* [YouTube - Andrew Kelley Practical Data Oriented Design][DoD]

[ECS]: https://github.com/SanderMertens/ecs-faq
[micro-benchmarks]: https://stackoverflow.com/a/2842707/338059
[profiling]: https://en.wikipedia.org/wiki/Profiling_(computer_programming)
[DoD]: https://www.youtube.com/watch?v=IroPQ150F6c&t=993s
