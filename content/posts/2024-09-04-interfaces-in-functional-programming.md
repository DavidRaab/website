---
layout:  post
title:   "interfaces in functional programming"
slug:    interfaces-in-functional-programming
date:    2024-09-04T00:00:00
lastmod: 2024-09-04T00:00:00
tags:    [interfaces,oop,fp,fsharp]
description:
---

Recently I worked with [Raylib](https://www.raylib.com/) a graphics library
for doing/learning game development. When I created multiple small demos there
was one thing I wanted todo across multiple projects. I wanted to drag
game objects on the screen. I already solved that problem once so now
I just needed to create a general function that allowed me to drag any kind
of object, instead of writing a specific solution again and again.

As always there are multiple ways on how to solve that. So first i give
you an overview on how draging in a game actually works.

1. Usually we only can drag a single object (or a selection of objects)
at once, because of this the State of what is currently draged is usually hold
in a state (global) variable. We need this State because the Drag State
must be processed from frame to frame.

2. When the Left Mouse Button is Pressed, we actually need to get the
mouse Position and check if the position where we clicked is inside of any
drageable object. If it is, we save it in the drag state.

3. Every dragable object actually needs some kind of collider we need to check
against. Usually simple shapes like Rectangle and Spheres are good enough, but
it also could be any kind of polygon.

4. When the mouse Button is released, we actually need to End the Drag State.
Usually also during draging we actually want to manipulate our drageable objects.
For example update the Position (otherwise we wouldn't drag anything).

So, how do we get a generic solution that works with any kind of object?

# Solution with an interface

The typical approach that is done in object-orientation would probably be to
think of it as a behaviour. So we come up with the idea of an `IDrageable` interface.

Everything that implements this interface can be draged. But what needs to be
implemented? From the description above the only thing we actually need is some
kind of *get a collider* for something that is drageable. Maybe we also need a
*Position* that we are able to change? I let you think about it.

So first I could create an interface. In F# creating such an interface would
look like this.

```fsharp
type IDrageable =
    abstract member GetCollider: unit -> Collider
```

All this interface describes is that every object that inherits from this interface
must implement a `GetCollider()` method that returns a `Collider`.

With such an interface we could now write a function with this signature.

```fsharp
let processDrag currentState (drageables:IDrageable list) mouseInfo =
    ...
```

The idea is simple. We pass it the `currentState` of what is currently draged,
or not draged. Then a bunch of `drageables` objects that can be draged, at last
a `mouseInfo` that contains the current information about the mouse like which
button is pressed/released and the mouse Position.

Now all what we have todo is go through each `IDrageable` object, usually we than
somewhere call `let collider = drageable.GetCollider()` to get the collider of
the drageable object. We check the `currentState` and the `mouseState` and must
determine if something starts draging, if something is already draged and so on.

Code can be a little bit exhaustive, but otherwise the logic behind it is very
easy solveable.

While I could solve it with an interface, this isn't how I solved it. Instead I
solved it with a more *functional* approach. So how does the functional approach
differ?

# The functional approach

So in a functional approach I don't use interfaces, still I want everything
to be able to be draged. I could start with the function signature above, and
without the interface it just looks like this.

```fsharp
let processDrag currentState (drageables:'a list) mouseInfo =
    ...
```

Now it expects a generic list of things. It can be of any type `'a` but all of
them must be the same type. The problem with this definition is that we cannot
solve our problem of getting a collider. We actually need some way to say
that not every kind of object is allowed. Only those, we can have a collider for.

That's why in the object-oriented way we create an interface and than just write.

```fsharp
let collider = obj.GetCollider()
```

but wouldn't it be cool if we just could write

```fsharp
let collider = getCollder obj
```

in the functional approach? Well, we actually can exactly do that! But we
need to change one thing for this to work. The problem is that we don't know
which `getCollider` function should be called, it completely depends on the `obj`.
So what we expect is that whoever calls the function `processDrag` must also pass
the correct `getCollider` function as an argument. Now the signature
looks like this.

```fsharp
let processDrag currentState (drageables:'a list) (getCollider: 'a -> Collider) mouseInfo =
    ...
```

As you can see, working without interfaces isn't so much of a difference, it's
just a little bit different. But maybe you ask why you ever should do the functional
approach? So here are some advantages and thoughts on why you better want
the functional approach instead of the interface solution.

# Advantages and Disadvantages

## The interface approach

Let's start with the interface. One advantage of using the interface approach
over the functional approach is that you need one argument less when calling the
function. We just could provide `IDrageable`. But what would happen if our
`IDrageable` would contain more than just one-method definition?

Then in the functional approach we have to pass one extra argument for every
method definition in the interface.

The next thing is an advantage and disadvnatage at the same time. We have to
*tag* all our objects with *IDrageable*. Here is one thing I dislike at many
*static-typed* object-oriented language. An interface is only implemented
if that silly *IDrageable* is added, but every object must explicitly add
that interface. So even if you have an object that implements an `GetCollider`
method with the correct interface, you cannot use it until you add `IDrageable`
to it. In that sense I think the language *Go* made it right with their interfaces
as otherwise being a horrible language. But actually this is one of the reasons
why in my opinion *object-orientation* should be *dynamic-typed*.

So adding the interface can be an advantage, now you can see which game objects
can be draged, and which not. And somehow it makes sense if every object knows
best how the `Collider` for draging must be created? Or maybe not?

It also can be an disadvantage. Let's assume you get a `Car` object and you
want to be able to drag it. But whoever implemented `Car` doesn't want to add
your `IDrageable` interface. Then you just have a problem.

Okay, there are workarounds like creating an Adapter and so on, but that doesn't
turn it into an advantage.

So the general problem of interfaces (at least in most languages) are that
the definition of a data-type, and its implementation of an interface are not
separated, but they should be. As far as I know that is what
[Rust Traits](https://doc.rust-lang.org/book/ch10-02-traits.html) does.

## The functional approach

### Too many arguments

So let's start with the disadvantage. For every method in an interface we
need to add one extra argument. When there is only one method in an interface
we only need to add one argument. In the case for 1-5 it is maybe acceptable.
I guess it depends on who you ask. I think already up to three becomes a little bit
clumsy, but would be *okay*.

But there is a functional solution to that problem very similar to an interface.
We create records containing functions.

```fsharp
type Drageable = {
    GetCollider: unit -> Collider
    Method1: X -> Y
    ...
}
```

Now we can create a data-structure containing multiple functions. In some sense
this approach is similar to object-orientation you see in `C` or `JavaScript`.
They create data-structures and the data-structure itself contains a pointer
to a function.

You can easily add as many functions to it as you wish, at some point there
is no big difference to an interface definition. In F# land people usually
discourage this kind of usage and they tell you to use interfaces instead.
I absolutely disagree to that. But more to this later.

With this kind of approach you always can keep the extra argument to exactly
one if needed.

### Types don't know about Drageable

Let's get back to the `Car` example. You get a `Car` object and want it to
be dragged. With the functional approach you can make any kind of object
compatible with your function, you just need to provide a `getCollider`
function.

You don't need to add an interface to `Car`, write another new wrapper class
around `Car`, you maybe can just provide an inline implementation of getting
a Collider just with a lambda function.

As an example, testing my Spatial Tree implementation I wanted to be able to
Drag every point on the Screen. A Point is defined like this.

```fsharp
type Point = {
    mutable Pos: Vector2
    Radius: float32
}
```

the code to Drag every point looked like this.

```fsharp
drag <- processDrag drag points (fun p -> Circle (p.Pos,p.Radius)) mouse
```

Here you can see that i just pass the Lambda `(fun p -> Circle (p.Pos,p.Radius))`
to create a Circle Collider for every point.

### Behaviour not Names

You know what sometimes is the worst in interfaces? Its requirements on establishing
*Names* as a convention. You know, I am not interested in if a method is
named `GetCollider()`, `Collider()`, `RectCollider()` or whatever name someone
comes up with. Maybe a data-type even has multiple Collider to choose from?

The only thing I am interested, from the perspective of implementing `processDrag`,
is that I somehow get a `Collider`, how that collider is exactly computed or which
method I need to call is not my business.

That's why I am expecting it to be passed as an argument. The advantage is also
seen in the `Point` example above. I can make `Point` Drageable without that
`Point` knows anything about draging. In fact, it doesn't even know anything about
a Collider, but I can create one if needed. In object-orientation land this is
also called *loose-coupling*. Something OOP people always try to achieve but
are unable todo.

It also doesn't need to apply to some kind of naming-convention.

### Naming Collisions

In the interface example above I said that somehow it makes sense that each
object knows best how to create a collider. Now I say the opposite, it doesn't.

You know dragging itself is not something that each object itself is aware of,
it is a feature of whatever you (the game) needs todo with it. In some games
it maybe makes sense to be able to drag cars, because you are building
a strategy top-down game that is about managing stolen cars, or something like
that. In a racing game it probably doesn't make sense at all.

That's also why something like `GetCollider` doesn't make sense. Or maybe a
`Car` already has a `GetCollider()`? I mean, how would a car otherwise have
collision detection in a racing game without having some idea of collider?

The problem here is that the same name can refer to different kind of *behavior*
or *ideas*. The `GetCollider()` that will be returned will probably be a very
accurate one. Some kind of complex concave mesh so collision detection is very
accurate. The problem is that this kind of collider can be bad
for mouse draging from a top-down perspective.

Or maybe should we have named our method in the interface `GetColliderSuitableFor2DScreenCollision()`?

For Draging you usually want just some kind of box around an object, not a pixel
perfect collision system. With the functional approach you don't run
into this problem, as you are the one when calling `processDrag` that tells
which Collider should be choosen. Or maybe you even create a box Collider
out of a Car because a Car doesn't provide a useful one by default? It's up to
you and your needs in whatever you are creating.

It doesn't make any sense that a Car itself should know what the best collider
should be to be drageable.

### Switchable behavior

That also leads to the next thing. For example I could add the interface
implementation to the Point like this.

```fsharp
type Point = {
    mutable Pos: Vector2
    Radius: float32
}
    interface IDrageble with
        member self.GetCollider() =
            Circle (self.Pos,self.Radius)
```

it has some quirks adding it to a Record, maybe a class would be better in F#.
Because F# has no implicit casting we actually need to explicitly cast it to
`IDrageable` when a function needs `IDrageable`, but this shouldn't be the
point of this article. Let's assume we could pass this record to an `IDrageable`
(or use a class instead), then this solutions is still less flexible.

Now again, the type itself is under control of creating the collider, but
why should the type know what the best collider actually is?

Let's assume I have very little points. Maybe most points just have a radius of
just very small pixels. Wouldn't it be annoying to try to drag 3 pixels points
on the screen?

Again, and I guess I am saying it for the third time. A type shouldn't know
how to be draged. Maybe I want to ensure when I create the collider that the
collider have at least a minimum size?

Maybe that minimum-size depends on the Camera-Zoom level of the game?
Could be, as this seems absolutely reasonable. But I certainly don't want `Point`
now about the Camera-Zoom Level. You remember the idea of loose coupling?
With the functional approach this all can be solved.

One big advantage is that during runtime I can easily switch the behaviour of different
collision detection systems. For example I could have a `colliderType` variable
that picks which collider generating version i should pick.

```fsharp
let getCollider =
    match colliderType with
    | Small  -> (fun p -> Circle (p.Pos, p.Radius)
    | Medium -> (fun p -> Circle (p.Pos, p.Radius + 10f) // increase radius by 10px
    | Large  -> (fun p -> Circle (p.Pos, p.Radius + 50f) // increase radius by 50px

drag <- processDrag drag points getCollider mouse
```

How would you do that with interfaces? Because then you actually need to implement
the same interface three times with three different implementations. Usually most
languages just allow you to provide one implementation for one interface. (You
could achieve that with three Adapter classes implementing it)

### Multiple implementations

The idee of multiple implementations is also the reason why I prefer records
with functions instead of interfaces. You absolutely can achieve the same
with an interface. You first write your interface definition.

For example you create `IDrageable` with three methods on it. Let's say
you want to provide three different `IDrageable` implementations. Then
you create three classes all implementing `IDrageable` let's say `SmallDrageable`,
`MediumDrageable` and `LargeDrageable`.

Now, those classes itself are pretty useless and you need to create an object of
them. So maybe in your code you do.

```fsharp
let small  = new SmallDrageable()
let medium = new MediumDrageable()
let large  = new LargeDrageable()
```

those classes are a little bit different. Instead of containing other state variables
you use them like this.

```fsharp
let collider = small.getCollider(obj)
```

and if that now looks familiar to you. It's basically the same as the first
functional approach by just accepting a `getCollider` function as an argument.
But instead of passing a single function, you pass an object containing multiple
functions at once. But you also could use this approach with a single function,
[because functions are interfaces]({{< ref 2024-05-30-functions-are-interfaces >}}).

But the problem of this approach is that it basically never makes sense to ever
have more than one object of `SmallDrageable` and so on.

On the other hand with Records this feels a lot more natural. You can define
a single record that holds your definition (like an interface) and you would
create three different records of the same type, instead of three different
classes again.

```fsharp
let small  = { GetCollider = ... }
let medium = { GetCollider = ... }
let large  = { GetCollider = ... }
```

because they are basically three objects on its own I also could create a new
record out of an existing one. Let's say I want to re-use `large` but only change
one method of it. Easy.

```fsharp
let newWhatever = {
    large with
       Method1 = ... // New implementation just for Method1
}
```

Those objects can be created once, and that's it. How would you create a new version
out of Large with just one method changed?

I guess maybe a new class and copy & paste all functions that don't change?

And no, don't tell me with inheritance. So `Large` inherits from `Medium`, `Medium` inherits
from `Small` and `Small` inherits from a Base class. The `Method1` implementations
of `Small` changes, now quickly tell me, does it affect `Large`?

### Only what you need

One of the biggest problem of an interface approach is to what it leads after some
time. It can go into two directions. Either you have hundreds of small interfaces
that all implement one method or maybe two.

Well if that is the case you directly can jump to the functional approach, because
then you have achieved a functional approach with lots of interfaces given every
signature just its own name.

Or maybe you are annoyed with hundreds of small interfaces and implementing
them and tagging all your objects with all kinds of interfaces. Then you go
to the *typical* OO approach that emerges after some time. With big interfaces
that just do too much.

I mean you have one interface that defines a *Position* and a *Collider* and
another interface defining a *Position* and a *Rotation*, in some other cases
you need a *Rotation* and a *Position* and so on. The chance that you create
hundreds of classes with all kinds permutation are in my opinion nearly zero.

What you likely do is maybe you have a *ITransform* (usually you also only
have a single implementation) and maybe you add a `Collider` to that definition,
because you are annoyed to create another new interface and it is so pleasent and easy
to just add it to `ITransform`.

So in the end you maybe end up with just very few interfaces in your code, but
all interfaces contain like 20 methods. And because they contain so much methods
you very likely only have one implementation of each of them, what makes interfaces
in itself more or less useless.

But you know, when you pick up the functional approach you always start with
only what you need. You have a collection of `'a`. And if you just need the
ability to get a collider from `'a` then this is also the only function you
expect to be passed as an argument. You don't expect to pass some kind of
interface that maybe has ten different methods on it you maybe never need.

And if it gets annoying that you add more and more function to be passed as arguments
then I think it is even something good.

It maybe let's you think about picking another solution. Creating some kind
of intermediate data-structure and so on. It actually needs some brain power
to solve a problem and let's you think deeply about how to solve it, instead
of just being able to add just one extra method to any crappy interface because
it is just easy and convenient.

As an example, let's say you have something of type `'a` and now in some general
function you need information about three properties of any object.

1. Position
2. Color
3. Collider

then you could expect three functions like `getPosition`, `getColor` and `getCollider`
to be passed as arguments to a function. But you also could create an
intermediate data-structure just for this.

```fsharp
type InfoNeeded = {
    Position: Vector2
    Color:    Color
    Collider: Collider
}
```

and then expect a function of type `'a -> InfoNeeded` that just returns all
information you need at once. Then you use it like this.

```fsharp
let whatever (xs:'a list) (getInfo:'a -> InfoNeeded) =
    for x in xs do
        let info = getInfo x
        printfn "Position is %A" info.Position
        ...
```

# Further Reading

* [OOP is Partial Application]({{< ref 2024-06-10-oop-is-partial-application.md >}})
* [Functions are interfaces]({{< ref 2024-05-30-functions-are-interfaces.md >}})
