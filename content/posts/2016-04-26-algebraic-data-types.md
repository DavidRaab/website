---
layout: post
title: "Algebraic Data-Types"
slug: algebraic-data-types
date: 2016-04-26
tags: [FSharp,types,data,units-of-measure]
description: An Introduction to algebraic data-types in F#
keywords: f#, fsharp, algebraic, data, types, functional, programming
---

When we work in a programming language we usually have some primitive data-types likes
`int`, `float`, `string`, `bool` and so on. All of those are important, but when we
need to create more advanced logic we usually want to create our own data-types
and group/compose different types together to create new data-types.

In an *algebraic type-system* there exists two different ways in how we can compose
types. They are named *Product-types* and *Sum-type*. I often name them *AND-composition*
and *OR-composition*. F# provides two *Product-types* and one *Sum-type*.

Those compositions are immutable by default. Because of this, we already have default
implementation for equality and comparison.

# Table of Content

<ul class="toc">
  <li><a href="#tuples">Tuples</a></li>
  <li><a href="#records">Records</a></li>
  <li><a href="#discriminated-unions">Discriminated Unions</a></li>
  <li><a href="#du-as-sum">Discriminated Unions as Sum-Types</a></li>
  <li><a href="#single-du">Single-case Discriminated Unions</a></li>
  <li><a href="#units-of-measure">Units of Measure</a></li>
  <li><a href="#recursive-du">Recursive Discriminated Unions</a></li>
    <ul>
      <li><a href="#rdu-lists">Lists</a></li>
      <li><a href="#rdu-btree">Binary Trees</a></li>
      <li><a href="#rdu-ds">Hierarchical Data-Structures</a></li>
    </ul>
  <li><a href="#invalid-state">Make invalid states un-representable</a></li>
  <li><a href="#summary">Summary</a></li>
  <li><a href="#further">Further Reading</a></li>
  <li><a href="#comments">Comments</a></li>
</ul>

<a name="tuples"></a>
# Tuples

A Tuple is a *Product-type*. The nice thing about Tuples is that we don't have to define
a *type* before-hand. We can easily create any kind of tuple by just separating
variables with a comma.

```fsharp
let foo = "David", 123
let bar = "Foo", "Bar", "Baz", 55
```

We can extract values from a tuple with *Pattern Matching*.

```fsharp
let name, nr = foo
let str1, str2, str3, nr = bar
```

When we look at the type-signature we see that `foo` has the type `string * int` and
`bar` has the type `string * string * string * int`. Tuples only can be compared
if they have the same amount of elements and the types are the same in the exact order.

A Tuple `string * int` is something different as `int * string`. But why are they anyway
named *Product type* and why do we *multiply* types? To understand this, let's look at
a simple tuple consisting of two boolean values. It's type would be `bool * bool`.
But how many possible values can we create with such a type? As `bool` only has two
states, we only can create four different values.

```fsharp
let b1 = true, true
let b2 = true, false
let b3 = false, true
let b4 = false, false
```

Every `bool` has two different values. A tuple like `bool * bool` contains two `bool`
at the same time. We can calculate the maximum amount of possible values by multiplying
the amount of every single type. In this case `2 * 2`. That is why we name it a
*Product-type*. We also could say a `bool * bool` is a `bool` **AND** another `bool`.

We also can create a type-definition first. But it is important to note that those are
not types on it's own, only *type-aliases*. For example we could define a `Point`
and then define a `Line` and a `Rect` like this:

```fsharp
type Point = float * float
type Line  = Point * Point
type Rect  = Point * Point
```

`Point`, `Line` and `Rect` in this example are not types on there own. We can use the names
as they are more declarative, but because their are just *aliases* we also can compare a
`Line` with a `Rect`. This is usually not what we want, and later we see how we can fix this.

But overall it is an advantage that we don't have to provide a type first. Tuples
are used in a lot of places for some intermediate types where we don't care for a named type.
For example if we have two lists and we want to zip them (1 element of 1.list, 1.element
of second list, 2 element of 1 list, 2 element of 2 list, ...) then we could just use
`List.zip` and it just returns us a tuple with values from both lists.

We also can use it to easily return multiple arguments from a function.

```fsharp
let doSomething () =
    5, 10

let x,y = doSomething ()
```

In those examples we don't want to define types all the time before-hand.

<a name="records"></a>
# Records

A *records* is also a *Product-type*. But we must define a type before-hand.
*Records* are also often named *Named Tuples*.

One advantage of a *Record-type* over a tuple is that we can use names to describe the
different fields. We could for example create a Person with fields like *First-name*,
*Last-Name*, *Hair-Color*, *Birthday* and *Size* with a tuple.

```fsharp
type Person = string * string * string * DateTime * float
```

But working with that is not really a pleasure, when we want to extract the birthday we
could write a function like:

```fsharp
let birthday (_,_,_,birthday,_) = birthday
```

But that becomes really messy fast. On top, it is also is not clear what the first three
`string` are. Actually i would say that using primitive types like `string`, `int`,
`float` is in general a bad practice. I will later talk more about this topic, but for now,
let's improve that example by just using a Record.

```fsharp
type Person = {
    FirstName: string
    LastName:  string
    HairColor: string
    Birthday:  DateTime
    Size:      float
}
```

A Record type gives us a default constructor: `{ ... }`

```fsharp
let me = {
    FirstName = "David"
    LastName  = "Raab"
    HairColor = "Blond"
    Birthday  = DateTime(1983, 2, 19)
    Size      = 100.0
}
```

We always must provide all fields, and we can access a field by its name:

```fsharp
printfn "%s %s has %s hair" me.FirstName me.LastName me.HairColor
// David Raab has Blond hair
```

A Record is immutable by default and has value semantic by default. That means we can compare those.

```fsharp
let otherMe = {
    FirstName = "David"
    LastName  = "Raab"
    HairColor = "Blond"
    Birthday  = DateTime(1983, 2, 19)
    Size      = 100.0
}

let markus = {
    FirstName = "Markus"
    LastName  = "Schwarz"
    HairColor = "Blond"
    Birthday  = DateTime(1983, 6, 3)
    Size      = 160.0
}

me = otherMe // true
me = markus  // false

me < markus // true
```

As we can see, as long all fields are the same we get a `true`. But how does `me < markus` work?
By default it uses the order of the fields. It starts comparing `FirstName`. Because "D" from
"David" is smaller than "M" from "Markus" we get `true`. If `FirstName` would be the same,
it would compare `LastName` and so on.

Often we want to change a field, but because we have an immutable data-type the only thing
we can do is to create a new record. F# provides a *Copy-and-update* syntax `{ source with ... }`

```fsharp
let me2     = { me     with Size = 173.0 }
let markus2 = { markus with Birthday = DateTime(1983,6,4); Size = 173.0 }

me < me2 // true -- because me2 has a bigger size
```

We also can use tuples inside of records. For example in a game we could use a money
field that uses three `int` to represent gold, silver and bronze.

```fsharp
type Player = {
    Name:  string
    Money: int * int * int
}

let richPlayer = {
    Name  = "Foo"
    Money = (3000, 200, 100) // 3000 gold, 200 silver, 100 bronze
}
```

Records also can contain other records.

```fsharp
type Point = { X:int; Y:int }
type Line  = { Start:Point; Stop:Point }
type Rect  = { TopLeft:Point; BottomRight:Point }

let line = {
   Start = { X=0; Y=0  }
   Stop  = { X=0; Y=10 }
}

let rect = {
   TopLeft     = { X=0;  Y=0  }
   BottomRight = { X=10; Y=10 }
}
```

Because records are comparable it also means a record that uses other records is also
automatically comparable. But as records are distinct types we cannot compare records of
different types. For example trying to compare `line` with `rect` will result
in a compile-time error.

If you know serialization formats like JSON you already can imagine that we can easily
represent JSON Structures this way. But we get the benefit that we have a statically
type-safe version of it, and those data-structures are immutable, and comparable!

For example we could generate a type-definition from
the [JSON example on Wikipedia](https://en.wikipedia.org/wiki/JSON)

```fsharp
module Contact =
    type Address = {
        StreetAddress: string
        City:          string
        State:         string
        PostalCode:    string
    }

    type PhoneNumber = {
        Type:   string
        Number: string
    }

    type Contact = {
        FirstName:    string
        LastName:     string
        IsAlive:      bool
        Age:          int
        Address:      Address
        PhoneNumbers: PhoneNumber list
        Children:     Contact list
        Spouse:       Contact option
    }
```

And create a value like this:

```fsharp
let contact = {
    FirstName = "John"
    LastName  = "Smith"
    IsAlive   = true
    Age       = 25
    Address   = {
        StreetAddress = "21 2nd Street"
        City          = "New York"
        State         = "NY"
        PostalCode    = "10021-3100"
    }
    PhoneNumbers = [
        {
            Type   = "Home"
            Number = "212 555-1234"
        }
        {
            Type   = "Office"
            Number = "646 555-4567"
        }
        {
            Type   = "Mobile"
            Number = "123 456-7890"
        }
    ]
    Children = []
    Spouse   = None
}
```

We can work with it like you would expect:

```fsharp
printfn "%s %s lives in %s" contact.FirstName contact.LastName contact.Address.City
// John Smith lives in New York

let numbers = contact.PhoneNumbers |> List.map (fun phone -> phone.Number)
printfn "He has the following numbers: %A" numbers
// He has the following numbers: ["212 555-1234"; "646 555-4567"; "123 456-7890"]
```

<a name="discriminated-unions"></a>
# Discriminated Unions (DU)

Up to that point, we only discussed two types: *Tuples* and *Records*. Both are *Product-types*
or how I would name them *AND Composition*. Both data-types always contains all the specified
fields. But a Discriminated Union is different. A Discriminated Union is a *Sum-Type* and
gives us the Possibility of a choice, or how I would name it, an *OR Composition*.

Let's review the JSON example above. What I dislike about it in general is that it just contains
too much `string` types. For example the `PhoneNumbers` part has a `Type` field, in an application
you usually expect that `Type` only can be specific `strings` but not all possible `string`
values. A better approach would be if we can represent the different available choices in
it's own type. With a DU we just can do that:

```fsharp
type PhoneNumberType =
    | Home
    | Office
    | Mobile
```

At this point you might probably compare it to an *enum* type as you know it from C# or Java.
But as we see soon there are not really comparable at all. The first difference is. An *enum*
usually is just a wrapper around an `int`, `byte` and so on. Usually in C# you could
for example still save the number `10` in a `PhoneNumberType` even if you just defined
three choices. But with a Discriminated Union that is not possible. There are concrete types
not just a wrapper for other values. The names `Home`, `Office` and `Mobile` are
basically constructors to create a `PhoneNumberType` and they are not just integers.

The second important difference. Every Constructor can carry an additional value. This value
can once again be a record, a tuple or another Discriminated Union. At this point we could
change the representation of `PhoneNumber` completely to a Discriminated Union instead
of a Record.

```fsharp
module ContactWithDU =
    type Address = {
        StreetAddress: string
        City:          string
        State:         string
        PostalCode:    string
    }

    type PhoneNumber =
        | Home   of string
        | Office of string
        | Mobile of string

    type Contact = {
        FirstName:    string
        LastName:     string
        IsAlive:      bool
        Age:          int
        Address:      Address
        PhoneNumbers: PhoneNumber list
        Children:     Contact list
        Spouse:       Contact option
    }
```

We now have a type that is either a `Home`, `Office` or `Mobile`. Every case
contains an additional `string`. With that in place, we could change the above JSON
construct to.

```fsharp
let contactWithDU = {
    FirstName = "John"
    LastName  = "Smith"
    IsAlive   = true
    Age       = 25
    Address   = {
        StreetAddress = "21 2nd Street"
        City          = "New York"
        State         = "NY"
        PostalCode    = "10021-3100"
    }
    PhoneNumbers = [
        Home   "212 555-1234"
        Office "646 555-4567"
        Mobile "123 456-7890"
    ]
    Children = []
    Spouse   = None
}
```

Not only is that shorter and more readable, it is also less error-prone because we cannot
provide an arbitrary string for the phone type. On top we now must handle all different cases
when we write code.

```fsharp
let printPhoneNumber number =
    match number with
    | Home nr   -> printfn "Home Nr. %s" nr
    | Office nr -> printfn "Office Nr. %s" nr
    | Mobile nr -> printfn "Mobile Nr. %s" nr

printPhoneNumber (Home   "123") // Home Nr. 123
printPhoneNumber (Office "456") // Office Nr. 456
printPhoneNumber (Mobile "789") // Mobile Nr. 789
```

Not *Pattern Matching* all cases leads to an error. If we forgot to write Code for the
`Mobile` case we get a compile-time *warning* that tells us that we forgot to
handle this case.

Imagine in the future you want to extend `PhoneNumber` and add additional types like
`Private`, `Fax`, `Pager` and `Other`. With a `string` you will just end up with buggy
code, as now you have to find all the places in your code where yo do decisions based on
the phone number types. The language also don't help you to find all those spots.

But with a DU instead the compiler will give you all places where you worked with
the `PhoneNumber` type and you didn't handle the new cases.

<a name="du-as-sum"></a>
# Discriminated Unions as Sum-Types

So far we talked about *Tuple* and *Records* as a *Product-type*. We name it *Product-type*
because we get the amount of possible combinations by multiplying the types. A tuple with
2, 3 or 4 `bool` can handle `2 * 2` (4), `2 * 2 * 2` (8) or `2 * 2 * 2 * 2` (16) different
values. The same is true for a *Record type* as a record type only provides a name for the
fields, but doesn't change the structure itself.

But with a Discriminated Union the possible amount of values only adds up. For example
a choice with four `bool` looks like this.

```fsharp
type Choice =
    | Choice1 of bool
    | Choice2 of bool
    | Choice3 of bool
    | Choice4 of bool
```

But we only have `8` possible values. We have four possible choices, and each choice can contain
two different states `true` or `false`. So we only have `8` possible values.

```fsharp
Choice1 true
Choice1 false
Choice2 true
Choice2 false
Choice3 true
Choice3 false
Choice4 true
Choice4 false
```

We can calculate the amount of possible values by adding the types. `2 + 2 + 2 + 2`. That's why
we call it a *Sum-Type*. But what is the point of calculating the amount of possible values?

First, we usually don't care for the exact amount of values a type can represent. When we use
four `int` we don't care how many billions of different values it can save. But we can use it in a
mathematical way. For example we could use `i` that represents `int`. A Product type with
four `int` can thus save `i*i*i*i` or `i ^ 4` possible values.

A Discriminated Union like `Choice` with four cases and every case contains a single `int` can
save `i + i + i + i` or in other words `4i` cases. Doing that kind of calculations is what
we call the [Cardinality](https://en.wikipedia.org/wiki/Cardinality). The point of calculating
the [Cardinality](https://en.wikipedia.org/wiki/Cardinality) is that we can calculate if two
different types can represent the same data.

For example let's look at the following type definitions:

```fsharp
module Cardinality =
    type Line  = (int * int) * (int * int)
    type Line2 = int * int * int * int

    let line  = (0,0), (10,10)
    let line2 = 0,0,10,10
```

`Line` and `Line2` are two different types.

1. `Line` contains **two Tuples** and each of those Tuple contain two `int`.
1. `Line2` on the other hand is a single tuple with four `int`.

Even if their are of different shapes, we can easily see that both types can save
the same amount of data. We can easily proof that by calculating the Cardinality.

1. For `Line` it looks like `(i * i) * (i * i)` that turns into `(i ^ 2) * (i ^ 2)` and this
   again turns into `i ^ 4`.
1. `Line2` is `i * i * i * i` and directly turns into `i ^ 4`.

With this we now know that we can transform any `Line` into a `Line2` or vice-versa.
This is important, because in programming it is important to choose the right data-format.
Choosing the right format often means some task can either becomes more easier to solve,
or we could come up with more efficient algorithms. By calculating the *Cardinality*
we can be sure that we have different representation of the same data.

Another way to use it is when we explicitly want to reduce the amount of possible values.
We did that above with the `PhoneNumber` type. Having less possible values also means we
need less code to check the edge-cases. Changing a `string` into a DU with just four cases
makes the code simpler and less error-prone. Especially if the language forces you
to check for all cases you could have, what F# does.

<a name="single-du"></a>
# Single-case Discriminated Unions

So far we only have seen Discriminated Unions that contain multiple choices, but we also
can create a DU with just a single choice. You will probably wonder why that is useful in
the first place.

Do you remember the Problem with `Line` and `Rect` at the beginning and that we could
compare `Line` and `Rect` because they were just type-aliases? By wrapping the
tuples in a *Single-case Discriminated Union* we get distinct types.

```fsharp
module TupleWithDU =
    type Point = Point of float * float
    type Line  = Line  of Point * Point
    type Rect  = Rect  of Point * Point

let p1   = Point (0.0, 5.0)
let p2   = Point (5.5, 8.8)
let line = Line (p1,p2)
let rect = Rect (p1,p2)
```

Now we cannot do `line = rect` anymore. We also cannot pass a `Rect` to a function
that expects a `Line`.

But how do we work with those values? Do we always have to *Pattern Match* even that we know
there only exists a single case? Yes we do, but every `let` expression is already *Pattern
Matching*. So we also can write:

```fsharp
let (Rect (tl,br)) = rect
```

This way we can extract a DU the same way we did with a Tuple. Not only that, we also can
use *Pattern Matching* in a function definition, because a function definition is also just
defined by `let`.

```fsharp
let printRect (Rect (tl,br)) =
    let (Point (x1,y1)) = tl
    let (Point (x2,y2)) = br
    printfn "Top Left: %f,%f" x1 y1
    printfn "Bottom Right: %f,%f" x2 y2

printRect rect
// Top Left: 0.000000,5.000000
// Bottom Right: 5.500000,8.800000
```

`printRect` is now a function expecting `Rect` as it's input. We cannot pass `line` anymore
to that function. Another important thing is that we also can nest *Pattern Matches* inside another.
This way we also can extract `tl` and `br` inside the function definition instead of doing
it in its own `let` bindings.

```fsharp
let printRect (Rect (Point (x1,y1), Point (x2,y2))) =
    printfn "Top Left: %f,%f" x1 y1
    printfn "Bottom Right: %f,%f" x2 y2
```

A Single-case DU is not only useful for giving tuples a distinct type. We can generally wrap
all types into a Single-Case DU and separate it in this way. For example at the beginning i
created a `Person` tuple like that.

```fsharp
type Person = string * string * string * DateTime * float
```

Do you still remember what the different `string`, `DateTime` or `float` types represented?
Me neither. So, why not wrap those Types in a Single-Case DU and give it more meaning?

```fsharp
module PersonDU =
    type FirstName = FirstName of string
    type LastName  = LastName  of string
    type HairColor = HairColor of string
    type Birthday  = Birthday  of DateTime
    type Size      = Size      of float
    type Person    = Person    of FirstName * LastName * HairColor * Birthday * Size
```

I think a type like

```fsharp
type Person = Person of FirstName * LastName * HairColor * Birthday * Size
```

is by far more readable and understandable. A function like:

```fsharp
let getBirthday (Person (_,_,_,bd,_)) = bd
```

Now also has the Type Signature `Person -> Birthday`. We would easily spot errors like this
one.

```fsharp
let getBirthday (Person (_,_,bd,_,_)) = bd
```

A function named `getBirthday` with the signature `Person -> HairColor` doesn't seems right!
But it overall helps by eliminating a lot of mistakes. By wrapping types into a DU, we can
enhance a type and make it more distinct to other type.

For example instead of using `string` for an `Email`. We could generate a `Email` type on it's own.
On top of it, we can create functions that take a `string` and return an `Email`, and we can add
validation to it, so when we want an `Email` we always ensure we have a valid `Email`, not
just some random `string`.

That's why I mentioned at the beginning that using `string`, `float`, `int` and so on
is bad practice. Because we hardly never expects that something can be any `string`, we
always have some validation. A `Uri`, `Email`, `Title`, `Content`, `HairColor`, `Name` and
so on might all be represented by a `string`. But that doesn't mean they are interchangeable,
or that every `string` is automatically a valid `Email`, and it also doesn't makes sense
to compare those different types at all.

Using primitive types like `string`, `float` throughout your code is also what we
name [Primitive Obsession](http://enterprisecraftsmanship.com/2015/03/07/functional-c-primitive-obsession/).

With DUs we can easily get rid of *Primitive Obsession* and eliminate a lot of bugs.

<a name="units-of-measure"></a>
# Units of Measure

One feature that F# offers that is not directly related to *Algebraic-data types* is
*Unit of Measures*. Before we go deeper into this topic, let's re-look at our latest
`Person` definition. We created a *wrapper* called `Size`, a `Size` contains a `float`,
but is that really a good definition? What means `Size 172.0` anyway? `172` of what?

*Meters*, *feets*, *miles* or *egg-sizes*? Working just with `int`, `float` and so
on is probably one common source of errors. The most famous one is probably
the [Mars Climate Orbiter](https://en.wikipedia.org/wiki/Mars_Climate_Orbiter). Because of
two software-systems, one produced *newton-seconds*, and another one produced *pound-seconds*
the calculation to enter the Mars atmosphere was wrong and it resulted in the destruction
of the *Mars Orbiter*.

A simple definition like:

```fsharp
type Lbfs = Lbfs of float
type Ns   = Ns   of float
```

Could have eliminated that bug, because we were not able to use *Lbfs* and *Ns* interchangeable.

In the same sense it usually makes sense to add unit of measures to types like `float`, to make
it clear which unit we expect. We not only eliminate dozens of possible bugs, we also make
the code more readable by having types like `meter`, `km`, `feet` and so on, instead of just
`float` everywhere. We could for example upgrade `Size` like that.

```fsharp
type Size = Cm of float
```

Now a definition like `Cm 172.0` makes it clear that we have `172` *centimeter*. We also
cannot easily add a float to a `Size` anymore. Because they are of different types.

Specifying units for numeric values is quite common and can help a lot to make code less
error-prone. But when we work with `Cm` we now always have to *unwrap* a `Cm` type
to work with it. If we just want to add two `Cm` together we have to write a function
that unwraps both add the `float` together and re-wrap it again in a `Cm` type.

It has advantages, but it is annoying to do that all over again for every numeric type,
because especially for numbers we always expect that we can add, subtract, multiply or divide
numbers. Because wrapping of numbers is such a common task, F# provides a feature named
*Units of Measure* that we can use instead.

```fsharp
[<Measure>] type meter
[<Measure>] type miles
[<Measure>] type hour

let sizeA = 10.0<meter>
let sizeB = 10.0<miles>
```

When we now do something like

```fsharp
let result = sizeA + sizeB
```

We get a compile-time error telling us that *meter* and *miles* doesn't match up. The benefit is
that we can work with those numbers without unwrapping them.

```fsharp
let doubleSize   = sizeA * 2.0
let area         = sizeA * sizeA
let meterPerHour = sizeA / 3.0<hour>
```

It is quite interesting to look at the types we now get. `doubleSize` is `20.0<meter>`. But
`area` is `100.0<meter ^ 2>`. So we get correctly `m²` not just `meter`. Trying to do
`doubleSize + area` would lead to an error again, because the types don't match up.

`meterPerHour` is `3.333333333<meter/hour>`, once again a new unit. It is more common
to use `kilometer` and `kmh` (kilometer per hour) and `mph` (miles per hour).

```fsharp
[<Measure>] type km
[<Measure>] type kmh = km / hour
[<Measure>] type mph = miles / hour

let distance  = 320.0<km>
let timeTaken = 4.5<hour>
let speed     = distance / timeTaken  // 71.1111111<kmh>

let newSpeed     = speed + 30.0<kmh>   // 101.1111111<kmh>
let newTimeTaken = distance / newSpeed // 3.164835165<hour>
```

With this example we not only can tag the different numbers with `km`, `hour` and so on.
The types also correctly convert to other type. Dividing `distance` by `timeTaken` produces `kmh`.
So we get the result that we travelled with an average speed of `71.11<kmh>`.

When we drive `30<kmh>` faster, we will only need `3.16<hour>` instead of `4.5<hour>`. We can
look at the types and see if we get our wanted result. For example when we write

```fsharp
let newTimeTaken = distance / newSpeed
```

we divide a `<km>` by `<km/hour>` this result is just a `<hour>`. But when we do

```fsharp
let newTimeTaken = distance * newSpeed
```

then we still get a result `32355.55556<km ^ 2/hour>`. But we probably didn't wanted
to calculate the square kilometer per hour. Whatever that means. Trying to pass that value to
a function that expects `<hour>` would fail at compile-time, because both are different types.

From a theoretical standpoint *Units of Measure* are not directly related to *Algebraic Data-types*,
but whenever you want to wrap a number in a Single-case DU, you should consider using
*Units of Measure* instead.

As a self-speaking example, let's reconsider the player example with money in a game.

```fsharp
module PlayerGame =
    [<Measure>] type Gold
    [<Measure>] type Silver
    [<Measure>] type Bronze

    type Player = {
        Name:  string
        Money: int<Gold> * int<Silver> * int<Bronze>
    }

    let richPlayer = {
        Name  = "Me"
        Money = 3000<Gold>, 200<Silver>, 100<Bronze>
    }
```

<a name="recursive-du"></a>
# Recursive Discriminated Unions

The biggest advantage of Discriminated Unions is that the definition can be recursive. *Tuples*
or *Records* cannot be recursive because F# is not lazy by default and we couldn't create
an immutable recursive record. It would be infinite. But a Discriminated Union contains multiple
different cases. So a DU can refer to itself, as long we have at least one non-recursive case.

<a name="rdu-lists"></a>
## Lists

For example, we could create our own `MyList` type that way. An Immutable linked-list is just
a DU with two cases. We either reached the end of a list or we have a single-element and another
`MyList`.

```fsharp
type MyList<'a> =
    | Empty
    | Cons of 'a * MyList<'a>
```

We now can create a `MyList` like this:

```fsharp
let nums = Cons(1, Cons(2, Cons(3, Cons(4, Cons(5, Empty)))))
```

When that looks impractical, actually the built-in F# list is exactly defined like this. The only
difference is that instead of `Empty` we use `[]`, and `Cons` is `::`. Because it is written
infix it looks a little bit nicer, but overall it is the same.

```fsharp
let numsl = 1::2::3::4::5::[]
```

We also can easily create a `fold` function, for further information on how fold works
or how you can create your own list functions you can continue with:
[From mutable loops to immutable folds]({{< ref 2016-04-05-mutable-loops-to-immutability >}}).

```fsharp
let rec fold folder (acc:'State) list =
    match list with
    | Empty        -> acc
    | Cons(x,list) -> fold folder (folder acc x) list

let rev list   = fold (fun acc x -> Cons(x,acc))   Empty list
let map f list = fold (fun acc x -> Cons(f x,acc)) Empty (rev list)

map (fun x -> x * x) nums
// Cons (1, Cons (4, Cons (9, Cons (16, Cons (25, Empty)))))
```

<a name="rdu-btree"></a>
## Binary Trees

We also can generate any kind of Binary Trees easily. Or in general any kind of Tree type. A
Binary Tree is either a `Leaf` an End-node without data. Or we have a `Node` that contains
a single element and a Left and a Right Node.

```fsharp
type Tree<'a> =
    | Leaf
    | Node of 'a * Tree<'a> * Tree<'a>

//    4
//  2   6
// 1 3 5 7

let tree =
    Node(4,
        Node(2,
            Node(1, Leaf, Leaf),
            Node(3, Leaf, Leaf)),
        Node(6,
            Node(5, Leaf, Leaf),
            Node(7, Leaf, Leaf)))

let fold f acc tree =
    let rec loop acc tree =
        match tree with
        | Leaf        -> acc
        | Node(x,l,r) ->
            let acc = loop acc l
            let acc = f acc x
            loop acc r
    loop acc tree

fold (fun acc x -> acc + (string x)) "" tree
// "1234567"
```

<a name="rdu-ds"></a>
## Hierarchical Data-Structures

We not only can generate general Data-Structure like Lists or Trees, but in general any kind
of hierarchical data-structures, for example we can use it in general to represent XML,
HTML and so on. We could use it to represent a document in the Markdown Structure.
(It doesn't contain all elements of Markdown)

```fsharp
type Markdown =
    | NewLine
    | Literal    of string
    | Bold       of string
    | InlineCode of string
    | Block      of Markdown list
```

We then can generate a structure like this:

```fsharp
let document =
    Block [
        Literal "Hello"; Bold "World!"; NewLine
        Literal "InlineCode of"; InlineCode "let sum x y = x + y"; NewLine
        Block [
            Literal "This is the end"
        ]
    ]
```

Usually we would write a parser that turns a string into such kind of structure.
Once you have such a structure working with it becomes pretty easy. To turn
any markdown document into HTML we just write.

```fsharp
let produceHtml markdown =
    let sb  = StringBuilder()
    let rec recurs = function
        | NewLine         -> sb.Append("<br/>")
        | Literal    str  -> sb.Append(str)
        | Bold       str  -> sb.AppendFormat("<strong>{0}</strong>", str)
        | InlineCode code -> sb.AppendFormat("<code>{0}</code>", code)
        | Block  markdown ->
            sb.Append("<p>") |> ignore
            for x in markdown do
                recurs x |> ignore
            sb.Append("</p>")
    recurs markdown |> ignore
    sb.ToString()
```

We just handle every DU case. If we encounter a recursive case like `Block` we just recurs.

```fsharp
produceHtml document
// <p>Hello<strong>World!</strong><br/>InlineCode of<code>let sum x y = x + y</code><br/><p>This is the end</p></p>
```

<a name="invalid-state"></a>
# Make invalid states un-representable

The general idea by designing data-types is to make illegal state un-representable. If invalid
states are un-representable, bugs cannot happen in the first place. This topic alone is big enough
of it's own and I will suggest to watch the videos in the *Further Reading* sections instead.

But to give a quick overview of the idea. Let's imagine we design a system and a user
could provide multiple e-mail addresses for his account. First, we want to make sure that
all e-mails are valid, so instead of using a `string` we create an `Email` type and
we ensure that we must go through validation. So instead of a list of strings we have
a list of `Email`.

```fsharp
type Email = Email of string
type Account = {
    ...
    Emails = Email list
    ...
}
```

Let's now imagine we want to ensure that every account always have at least one valid e-mail.
We could write code that always check if the list always at least contains one
element. But we also could achieve the same by changing the data-type so we get
compile-time safety that this requirement always holds true.

```fsharp
type Account = {
    ...
    PrimaryEmail    = Email
    SecondaryEmails = Email list
    ...
}
```

Now Account always must have at least one single valid `PrimaryEmail`. This idea can be extended.
In [Types + Properties = Software](https://blog.ploeh.dk/2016/02/10/types-properties-software-designing-with-types/)
Mark Seemann describes how we can embed the rules of Tennis in the Type System itself. For
example in Tennis we have the Points *Love, 15, 30, 40*. You could go on and use just a
`int` for that, and instead of `Love` you use `0`. But this is again error-prone. Because an
`int` support far more values as we need. We also could set it to `-456` for example. So
the natural idea is to use a DU for this case.

```fsharp
type Points = Love | Fifteen | Thirty
```

Why no `Forty`? Because if we define `Forty` it could happen that two players
have `Forty`, but that is an invalid state. When both Players have `Forty` the game
state is named `Deuce`. What we could do instead, we make `Forty` a whole new type
that saves which Player has `Forty` and saves the Points of the other player. With this
idea we cannot express the idea to have two Player with `Forty` because we designed
the data from the ground-up in a way that we cannot express an invalid state. When you
write the code around your data-structures you are forced to use the `Deuce`
game state instead.

<a name="summary"></a>
# Summary

An algebraic-type system is at some point simple. Because it just provides two ways in how
we can compose types. We either can use an *AND composition* or we can use an *OR composition*.

With this idea we can generate various data-structures on its own. We also can encode
various rules into the type-system itself. By using rich types we can get away with a lot
of common bugs and prevent errors from happening in the first place.

Creating rich data-types can help in readability, and it also helps to write correct code.
As Linus Torvalds once said, we should write our code around our data-structures.

> git actually has a simple design, with stable and reasonably well-documented data structures.
> In fact, I'm a huge proponent of designing your code around the data, rather than the other
> way around, and I think it's one of the reasons git has been fairly successful [...] I will,
> in fact, claim that the difference between a bad programmer and a good one is whether he
> considers his code or his data structures more important.

<a name="further"></a>
# Further Reading

1. [[Video] Domain modelling with the F# type system](https://vimeo.com/97507575)
1. [The "Designing with types" series](http://fsharpforfunandprofit.com/series/designing-with-types.html)
1. [Types + Properties = Software](http://blog.ploeh.dk/2016/02/10/types-properties-software-designing-with-types/)
1. [Avoiding Silly Mistakes with F# Unit of Measure](http://www.markheath.net/post/avoid-silly-mistakes-fsharp-units-of-measure)
1. [Power of mathematics: Reasoning about functional types](http://tomasp.net/blog/types-and-math.aspx/)
1. [Units of Measure](https://fsharpforfunandprofit.com/posts/units-of-measure/)
1. [Cardinality](https://en.wikipedia.org/wiki/Cardinality)
1. [[C#] Primitive Obsession](http://enterprisecraftsmanship.com/2015/03/07/functional-c-primitive-obsession/)
1. [[PHP] Type Safety and Money](http://verraes.net/2016/02/type-safety-and-money/)

<a name="comments"></a>
