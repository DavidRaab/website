---
layout: post
title:  "Mutability vs. Immutability: Valid objects"
slug:   mutabaility-vs-immutability-validation
date:   2017-02-27
tags:   [FSharp,CSharp,immutability,comparison]
description: "At any time we want to maintain valid objects. In this article we compare how hard/easy it is with mutability and immutability."
keywords:    f#, c#, fsharp, csharp, mutability, immutability, comparison
---

I already wrote [an article that explains immutability]({{< ref 2016-03-14-immutability-and-pure-functions >}}),
but one thing I hand-waved was the benefits of immutability and why you should
program with immutable values.

In this article I talk about those benefits by trying to maintaining valid
objects at all time and show how we can achieve it with mutability and
immutability.

One question might be why I'm not just showing the immutable part. I could do
this, but the problem I see is that it isn't so obvious how hard the mutable
part really is.

Because of this, first I show all the things you have to keep in mind if you work
with mutability. Then we see how immutability helps us.

# About this article

Throughout this article I will use C# and F#. I use C# for the mutable examples
and F# for the immutable example. There are multiple reasons for this decision:

1. Mutability is best handled by classes and C# is built around that concepts
   and everything by default is mutable.
2. Immutability is best handled by immutable data-types and functions that
   operate on them. In F# everything is immutable by default.
3. If you are new to F#, probably this article can help a little bit if
   you see how C# code translates to F#.

Throughout this article I use some words, and you should know my definition
of those words to avoid confusion:

1. **State**: State, mutable objects or mutability is just used interchangeable.
2. **Object**: The word *object* is not limited to OOP. In general it just means
   **a thing**. Also F# data-types like records, unions or tuples are just objects.
3. **Function**: Anything that you somehow execute is a function. This includes
   class constructors, methods, static methods, F# functions and so on.
4. **Constructors**: A *constructor* is any *function* that creates a new
   *object* if you don't already have one.

With immutability a constructor might seems a little blurry because every function
returns a new object. The important detail is **if you don't already have one**.
As an example `List.map`, `List.filter` or `List.fold` are not considered as
constructors. All those functions already expect a List that they operator on.
If you don't already have a list, you cannot use those functions.

In F# there are multiple ways to create a list, like the special List syntax
`[1;2;3]` or the cons operator `1 :: 2` or functions like `List.unfold`. All
of those things create a list without that you need one beforehand and
thus are considered *constructors*.

This article is not a C# vs. F# or OO vs. FP comparison. You also can create
immutable object in C# and get the same benefits as in the F# examples. Or
you can create mutable classes in F# and get the same disadvantages as in
the C# examples.

# Mutability: The MinMax example

We start with a really small and simple example. A `MinMax` class that only has
the purpose to keep a *current* value between a defined *minimum* and *maximum*.
A C# class could look like this:

```csharp
public class MinMax {
    public int Minimum { get; private set; }
    public int Maximum { get; private set; }
    public int Current { get; private set; }

    public MinMax(int minimum, int maximum, int current) {
        this.Minimum = minimum;
        this.Maximum = maximum;
        this.Current = current;

        this.CheckMinMax();
        this.CheckCurrent();
    }

    private void CheckMinMax() {
        if ( this.Minimum > this.Maximum ) {
            throw new Exception("Minimum greater than Maximum");
        }
    }

    private void CheckCurrent() {
        if ( this.Current > this.Maximum ) {
            this.Current = this.Maximum;
        }
        else if ( this.Current < this.Minimum ) {
            this.Current = this.Minimum;
        }
    }

    public void Add(int amount) {
        this.Current += amount;
        this.CheckCurrent();
    }

    public void Subtract(int amount) {
        this.Current -= amount;
        this.CheckCurrent();
    }
}
```

We start by only providing methods to change the *current* value. When we
look closer we also could define our `MinMax` class as two distinct rules.

1. The minimum value must be smaller or equal to maximum. `CheckMinMax()`
2. The current value must be between minimum and maximum. `CheckCurrent()`

Often people argue that we don't need to re-check all rules, only those that
are somehow affected. As an example the `Add` and `Subtract` functions
only call `CheckCurrent()`. Obviously, we don't need to call `CheckMinMax()`
if we don't change those values.

While this is true, for more complex objects it can be hard to determine
what is affected and what's not. Imagine we add a `SetMaximum` function to
mutate the maximum. Would it be enough to just call `CheckMinMax()`
because only maximum changed?

Well no, otherwise this code would create an invalid state:

```csharp
var v = new MinMax(0, 100, 80);
v.SetMaximum(50);
// Minimum=0; Maximum=50; Current=80
```

In a very easy example like this it might be obvious that you must re-check
both rules and call `CheckMinMax()` **and** `CheckCurrent()`. In a more complex
class determining what needs to be called can be a lot harder.

As our goal is to always maintain valid objects, why not make up some rules like
a coding-standard that when we strictly follow it, we can be sure objects always
are valid?

We just define our first rule as:

> First Rule: Always re-validate every rule to ensure the correctness of an object.

As a conclusion of the first rule we could create a single function like `IsValid()`
instead of multiple functions like `CheckMinMax()` and `CheckCurrent()`. But
in this example, lets just call both. Following our rule, now imagine we
implement `SetMaximum()` in this way:

```csharp
public void SetMaximum(int max) {
    this.Maximum = max;
    this.CheckMinMax();
    this.CheckCurrent();
}
```

**Question:** Is this implementation correct, or not? What is the state of `v` at the end of
this code example?

```csharp
var v = new MinMax(0, 100, 80);

try {
    v.SetMaximum(-100);
}
catch {}

v.Add(10);
```

**Answer:** `SetMaximum` is not correct and the state of `v` will be:
`Minimum=0; Maximum=-100; Current=-100`. Mutating a field before we know
a change is valid is a problem. Our next rule:

> Second Rule: You always must validate before mutating.

The interesting part is. Before we implemented `SetMaximum()` this was not
a problem! `Add` and `Subtract` both check after we mutate the `Current`
value. So what is the difference? Why does it work with `Add` and `Subtract`
but not with `SetMaximum`?

One reason is that we only called `CheckMinMax` from the constructor. The
exception in `CheckMinMax` aborts the whole creation of an object. But when
we already have an object and call it from a method that isn't enough.

We can create another rule to describe that this is okay.

> Third Rule: The second rule don't need to be followed in a constructor. In
> a constructor you always are allowed to mutate and validate afterwards.

But this still doesn't explain why first mutating and the checking is no
problem in `Add` and `Subtract`. With our current rules so far we are not
allowed to write `Add` and `Subtract` in the way it is currently written.

The reason why we can validate afterwards is because our `CheckCurrent()` not
just validates and throws an error in the case there is something invalid.
It actually **fixes** the problem.

If `Current` gets bigger than `Maximum`, and thus invalidates the object,
it fixes the problem by setting `Current` to the `Maximum` value. So whenever
we can **fix** an invalid object and there is a way to turn it back into
a valid object, we actually are allowed to mutate and check afterwards.

> Fourth Rule: If there is a way to *fix* an invalid object, you
> are allowed to mutate and validate even outside of an constructor.

We also could apply this kind of **fixing** to `Minimum` and `Maximum`.
For example I could add the following logic to the `SetMinimum` and
`SetMaximum` methods.

* If Minimum is bigger than Maximum, then set Maximum to the same value.
* If Maximum is smaller than Minimum, then set Minimum to the same value.

Every method (or say at least "a lot") could somehow fix an object.
If a string is restricted to 80 characters you could reset a string to
the empty string, or maybe cut everything off after 80 characters. The
problem with logic like these are they are very hard to remember.

There isn't a right or wrong approach. But depending which
approach you pick you have other rules you must follow. If you pick the approach
to throw/return errors then you should validate before mutating. If you fix
invalid objects you are allowed to mutate and call your **fix** function afterwards.

<div class="info">
It might be true that we cannot label one as <strong>right</strong> or <strong>wrong</strong>,
nevertheless I suggest we should avoid a solution that fixes something automatically
most of the time as they are hard to remember. Returning an error, no matter how
you do it exactly (null / Options / Results / Exceptions), is most of the time
more flexible and easier to comprehend.
</div>

If we now decide for one way and follow the rules, is it then impossible
that an object will never be in an invalid state? The answer is no. But
we need another example to demonstrate this.

The `MinMax` class uses three mutable fields. While the fields itself are mutable
this is not true for the `int` itself. An `int` is an immutable type. So we also
must consider an example where the objects themselves are mutable.

For the next example let's consider a `Product` class with two fields `Name`
and `Price`. To make it easy, we just assume every `Name` and `Price` is valid.
Instead we focus on a `ProductsPriceOver` class. The purpose of this class
is to maintain a list of `Product`s with only one rule. Every `Product` must
be more expensive then a defined minimum.

The final usage of those two classes could look like this:

```csharp
var a = new Product("A", 9.99);
var b = new Product("B", 19.99);
var c = new Product("C", 49.99);

var ppo  = new ProductsPriceOver(10.00);
ppo.add(a);
ppo.add(b);
ppo.add(c);
```

First we create an `ProductsPriceOver` object that only accepts Products more expensive than `10.00`.
When we implement `ProductsPriceOver` it means the `add` method must check the `Price`
of every `Product`. When the above code gets executed we assume only product "B"
and "C" are inside `ProductsPriceOver`.

**Question:** If we assume `add` is the only method `ProductsPriceOver` will ever
implement. Will `ProductsPriceOver` always be valid?

**Answer:** No. We can invalidate `ProductsPriceOver` by just changing the Price
of any Product directly.

```csharp
b.Price = 5.00
```

The problem is that we reference Product B directly in `ProductsPriceOver` and we
also never get a notification if one of the products changes.

There are two ways how we can fix that. We could choose only one so I consider
both as the fifth rule.

> Fifth Rule A: Mutable objects must have some kind of notification mechanism
> once they changed.

As an example. We could add a `Changed` event to every mutable object that gets
fired as soon an object changes.

This way the `ppo.add(x)` function can add an event handler to every product.
If a product changes it price, it re-checks if the new price is high enough
to still be part of the `ppo` object.

> Fifth Rule B: Mutable objects must have a `Copy` function that can create
> deep copies of an object.

This technique is often named *defensive copies*. When `ppo.add(x)` is
called it doesn't save a reference to the same object. It creates a copy
of the whole object and only keeps the copy in its internal list.

When we do this, even after we execute `b.Price = 5.00` the product in `ppo`
will still be `19.99` instead of `5`. This way every `ProductsPriceOver` object
stay valid, but we need a way to update the price in `ProductsPriceOver`.
We look at this problem more precisely later in the immutability part.

In some way *defensive copies* and *immutability* are the same. Because
sharing a mutable object can cause problems, defensive copying creates
copies of objects and try to avoid sharing. With immutable objects
sharing is no problem, but we create copies of objects when we want to
change something.

The difference is at what time we create copies. Defensive copies
creates copies before-hand, immutability creates copies only in the exact
moment something is changing.

But after all, we are still not safe from invalid objects! Rule Five only
covers things we could describe as *Input*. It only covers those cases
when our current object receives another object from outside of our
current object.

Another problem that can occur is when we return mutable objects from
methods. Or in some sense when we return them as *Output*.

Up so far The `ProductsPriceOver` class only provides an `add` method
and is pretty useless at this time. We assume that the `ProductsPriceOver`
object has an internal mutable list to keep track of all its Products.
This list is usually created when we create an `ProductsPriceOver` object
and is not passed from the outside.

But when we return those internal mutable object, then our object can be
easily invalidated. Let's assume our `ppo` has a `Products` field that
directly returns the products as a `List<Product>`. We could then write
something like this:

```csharp
var a = new Product("A", 9.99);
var b = new Product("B", 19.99);
var c = new Product("C", 49.99);

var ppo  = new ProductsPriceOver(10.00);
ppo.add(a);
ppo.add(b);
ppo.add(c);

// At this time ppo contains only "B" and "C"

List<Product> products = ppo.Products
products.Add(a)

// ppo now contains "A", "B" and "C"!!!
```

By directly accessing the internal mutable array we bypass the `ppo.add()` method
including its enforcement of the rules. We can solve this problem in the exact
same way we did with Rule Five.

> Sixth Rule A: Every mutable object we return must have a Changed event
> that gets fired when an object was mutated.

If the internal mutable list has an event that gets fired whenever the list
somehow mutates. Then our `ProductsPriceOver` class could add an event-handler
that re-checks if all elements inside the list are valid.

> Sixth Rule B: Never return mutable objects directly. Return defensive
> copies instead.

If `ppo.Products` just returns a copy then the caller can manipulate
the returning list in any possible way, but it doesn't affect `ppo`.

But for the output case, there exists a third way.

> Sixth Rule C: Don't allow access to internal mutable objects at all.

This rule is probably the most used one in practice. And in my opinion it
is the worst rule of all. This rule is so bad that in my opinion most problems
of OO programming are connected to this idea. This rule alone deserves a
whole article on its own to describe its evilness. Yes, I'm serious and this
is not a joke!

Currently I left it to he reader to figure out how many implications this has,
otherwise this article will get too long. You can start with the question:
*When you cannot access the `Products` list of an `ProductsPriceOver` object.
How do you implement new functionality?*

Now that we have Rule Five and Six, lets talk about these. Actually you cannot
freely decide if you either use events or defensive copies. Events are
*reactive*. They get fired *after* a mutation happened. So you only can use
events if there is a way to fix an object after it became invalid.

If there is no way to fix an invalid object, you must use defensive copies!
This is not really a rule you must follow, more a reminder which
previously rule you must choose. But because of its important I still
consider it as a new rule.

> Seventh Rule: Events can only be used if there is always a way to fix
> an invalid object. If there is no way to fix an invalid object, use
> defensive copies.

Straight away, here is my last rule without much explanation as
it should be self-explanatory.

> Eighth Rule: If your mutable objects are accessed by multiple threads
> (mutable shared state). You also must add synchronization primitives
> to avoid race conditions that can bring an object into an invalid
> state.

And here is a bonus rule for the eighth rule.

> Bonus Rule: Just because every method of an object has synchronization
> primitives doesn't mean it is thread-safe. Because of this, you probably
> want to ignore Rule Eight.

Also this is worth its own article I will write about in the near future.

Let's get an overview of all rules we have to follow so we can be sure state
will always be valid.

# All Rules to ensure a valid State

1. **First Rule:** Always re-validate every rule to ensure the correctness of an object.
1. **Second Rule:** You always must validate before mutating.
1. **Third Rule:** The second rule don't need to be followed in a constructor. In
   a constructor you always are allowed to mutate and validate afterwards.
1. **Fourth Rule:** If there is a way to *fix* an invalid object, you
   are allowed to mutate and validate even outside of an constructor.
1. **Fifth Rule A:** Mutable objects must have some kind of notification mechanism
   once they changed.
1. **Fifth Rule B:** Mutable objects must have a `Copy` function that can create
   deep copies of an object.
1. **Sixth Rule A:** Every mutable object we return must have a Changed event
   that gets fired when an object was mutated.
1. **Sixth Rule B:** Never return mutable objects directly. Return defensive
   copies instead.
1. **Sixth Rule C:** Don't allow access to internal mutable objects at all.
1. **Seventh Rule:** Events can only be used if there is always a way to fix
   an invalid object. If there is no way to fix an invalid object, use
   defensive copies.
1. **Eighth Rule:** If your mutable objects are accessed by multiple threads
   (mutable shared state). You also must add synchronization primitives
   to avoid race conditions that can bring an object into an invalid
   state.
1. **Bonus Rule:** Just because every method of an object has synchronization
   primitives doesn't mean it is thread-safe. Because of this, you probably
   want to ignore Rule Eighth.

In fact, even now I'm not sure if I really covered everything! Besides
the amount of rules you should follow to ensure an object is always valid,
the problem is that nearly every rule either has an exception or special
requirements when you should/can use them.

This overall makes mutability pretty hard. Now that we covered the mutability
part lets see how immutability helps us.

# Designing with Immutability

Our MinMax class can be easily represented by a record in F#. Records in F#
are immutable by default and can group data together like classes do.

```fsharp
type MinMax = {
    Minimum: int
    Maximum: int
    Current: int
}
```

The problem is that up to this point we have no validation and can create a lot
of invalid objects like:

```fsharp
let a = {Minimum=0; Maximum=100; Current=1000}
let b = {Minimum=100; Maximum=0; Current=-1000}
```

We solve that problem by creating a module and make the record constructor private.

```fsharp
module MinMax =
    type T = private {
        Minimum: int
        Maximum: int
        Current: int
    }
```

Now we cannot create a `MinMax` object outside of the `MinMax` module. So we need at
least one constructor. Because we want to eliminate the ability to create invalid
`MinMax` objects we also add any validation we want to this constructor. Our
final constructor `create` looks like this:

```fsharp
let create min max cur =
    let create min max cur = {Minimum=min; Maximum=max; Current=cur}

    if   min > max then failwith "Minimum greater than Maximum"
    elif cur > max then create min max max
    elif cur < min then create min max min
    else create min max cur
```

I also create a small `show` function that can display the content of a `MinMax` object.

```fsharp
let show mm = sprintf "%d range %d/%d" mm.Current mm.Minimum mm.Maximum
```

Outside of the `MinMax` module the only way to create a new `MinMax` object is
by using the `MinMax.create` constructor. As an example the previously invalid
state cannot be created anymore.

```fsharp
MinMax.show (MinMax.create 0 100 1000) // 100 range 0/100
MinMax.create 100 0 -1000              // Exception: Minimum greater than Maximum
```

Inside the module it is a little bit different. Every function still has access
to the record constructor, so there is a possibility of creating an invalid object.
This leads to the **only** rule you will ever need with immutability!

> Golden Rule: Create a constructor function with the name `create` that contains
> all rules and validation logic. Only use this function to create new
> objects from now on.

Here are the `add` and `subtract` functions:

```fsharp
let add x mm      = create mm.Minimum mm.Maximum (mm.Current + x)
let subtract x mm = create mm.Minimum mm.Maximum (mm.Current - x)
```

Consider that these functions don't contain any validation logic and
still work as intended. With mutability we had rules if we
need to validate or mutate first. Because we cannot mutate in the first
place we must create a new object and we do that by using the `create`
function that contains all validation logic. We also can say, we are
falling into the pit of success.

```fsharp
MinMax.create 0 100 50
|> MinMax.add 200
|> MinMax.show // 100 range 0/100

MinMax.create 0 100 50
|> MinMax.subtract 200
|> MinMax.show // 0 range 0/100
```

Because functions return a new `MinMax` object we often end up with a fluid-interface.
The only thing you have to consider is that the `MinMax` object should be
the last argument of a function.

Next look at `setMinimum` and `setMaximum`. In our mutation version it throws
an exception, but it could leave an object in an invalid state if we first
mutated an object. Needless to say that we never get this problem with
immutability.

```fsharp
let setMaximum max mm = create mm.Minimum max mm.Current
let setMinimum min mm = create min mm.Maximum mm.Current
```

As you can see, everything always stays valid:

```fsharp
let v = MinMax.create 0 100 80

try
    let x = MinMax.setMaximum -100 v
    printfn "%s" (MinMax.show x)
with
    | _ -> ()

MinMax.show v
// 80 range 0/100
```

This should be obvious as `MinMax.setMaximum` returns a new object and don't
mutate an object. `v` never can become invalid.

You can choose if you want to *fix* or throw an error exactly like we did
with the mutable version. But there are no special rules you must consider
how something can become invalid. Nevertheless it still might be a good idea
to avoid *fixing*.

<div class="info">
I only use exceptions to throw errors. In functional language this is usually
not considered good. Instead of throwing an exception I also could return an
<strong>Option</strong> or a <strong>Result-type</strong>. Functional error
handling is yet another topic on its own. But for this topic, none of those
matter as long the validation stays inside the constructor.
</div>

Up to this point I already covered the whole `MinMax` example, and covered
everything up to the Fifth Rule in our mutation based code. Here is the full
code for our `MinMax` module.

```fsharp
module MinMax =
    type T = private {
        Minimum: int
        Maximum: int
        Current: int
    }
    let minimum mm = mm.Minimum
    let maximum mm = mm.Maximum
    let current mm = mm.Current

    let create min max cur =
        let create min max cur = {Minimum=min; Maximum=max; Current=cur}

        if   min > max then failwith "Minimum greater than Maximum"
        elif cur > max then create min max max
        elif cur < min then create min max min
        else create min max cur

    let show mm = sprintf "%d range %d/%d" mm.Current mm.Minimum mm.Maximum

    let add x mm      = create mm.Minimum mm.Maximum (mm.Current + x)
    let subtract x mm = create mm.Minimum mm.Maximum (mm.Current - x)

    let setMaximum max mm = create mm.Minimum max mm.Current
    let setMinimum min mm = create min mm.Maximum mm.Current
```

In the mutation based code I introduced the `ProductsPriceOver` example
and we discussed what happens if mutable objects are either passed
or returned from an object.

We had two solution for this problem. We either fired events as soon
something was changed or we created defensive copies. Using events
doesn't make any sense with immutable objects. As they cannot change those
events will never be fired.

And as discussed previously the idea of creating defensive copies is very
similar to immutability. In fact it created another problem that if we mutated
a product it didn't affected the same product in a `ProductsPriceOver` object.
So lets address this problem.

First, we create an immutable `Product`. It contains no validation and I also
could just used the Record definition (3 lines of code). But I anyway decided
to create another module with functions so you could get an idea how to convert
a similar mutable `Product` class to an immutable `Product` module.

```fsharp
module Product =
    type T = private {
        Name:  string
        Price: decimal
    }
    // Constructor
    let create name price =
        {Name=name; Price=price}

    // Getters
    let name mm  = mm.Name
    let price mm = mm.Price

    // Setter
    let setName name mm   = create name mm.Price
    let setPrice price mm = create mm.Name price

    // other functions
    let toString p = sprintf "%s -- %.2f" p.Name p.Price
```

Second, we create our `ProductsPriceOver` type. This time I just show
the whole code.

```fsharp
module ProductsPriceOver =
    type T = private {
        Price:    decimal
        Products: Product.T list
    }
    // Constructor
    let priceOver x product =
        (Product.price product) > x
    let create price products =
        let products = List.filter (priceOver price) products
        {Price=price; Products=products}

    // Getter
    let price ppo    = ppo.Price
    let products ppo = ppo.Products

    // Setter
    let setPrice price ppo       = create price ppo.Products
    let setProducts products ppo = create ppo.Price products

    // Other Functions
    let toString ppo =
        sprintf "%A" (List.map Product.toString ppo.Products)

    let add product ppo =
        create ppo.Price (product :: ppo.Products)

    let update product ppo =
        let mapper old =
            if   (Product.name old) = (Product.name product)
            then product
            else old
        let updatedList = List.map mapper ppo.Products
        create ppo.Price updatedList
```

Here is a short summary of the stuff you already should now.

Instead of a class we create a module. We put all private mutable fields
inside an immutable record. We make this record private. A constructor
function that we name `create` has the purpose of creating a new object and
contains all validation. In this case it filters the list and ensures that only
products over a specified price will be saved in our object. The Getter and
Setter part are identical to what you know from OO. Because the record is private
we need getters so we can access the `Price` and `Products` outside of the module.
The setter just sets the value of `Price` or `Products` to a new value. But it does
it by creating new objects instead of mutating. The `toString` and `add`
functions are normal functions like `MinMax.show`, `MinMax.add` or
`MinMax.subtract` with more logic.

For a moment lets forget about the `update` function. Lets see what we can
do with our `ProductsPriceOver` module so far:

```fsharp
// returns a list of product names from a ProductsPriceOver object
let getProductNames ppo =
    List.map Product.name (ProductsPriceOver.products ppo)

// Our Products
let a = Product.create "A" 9.99m
let b = Product.create "B" 19.99m
let c = Product.create "C" 49.99m

// Initializing and adding our products
let ppo =
    ProductsPriceOver.create 10.00m []
    |> ProductsPriceOver.add a
    |> ProductsPriceOver.add b
    |> ProductsPriceOver.add c
printfn "%A" (getProductNames ppo) // ["C"; "B"]
// Correct: only contains B and C as price of A is below 10.00

// Replace the whole products
let ppo2 = ProductsPriceOver.setProducts [a;c] ppo
printfn "%A" (getProductNames ppo2) // ["C"]
// Correct: only contains C as price of A is below 10.00

// Increase minimum price
let ppo3 = ProductsPriceOver.setPrice 20.00m ppo
printfn "%A" (getProductNames ppo3) // ["C"]
// Correct: Contained B and C before. B was removed after
//          increasing the needed price a product must have.

// Decrease Price and then reset all products
let ppo4 =
    ppo
    |> ProductsPriceOver.setPrice 5.00m
    |> ProductsPriceOver.setProducts [a;b;c]
printfn "%A" (getProductNames ppo4) // ["A"; "B"; "C"]
// Correct: Should be obvious.
```

Again, we can see how we got a fluid-syntax or in general an data-flow API
very easily. Amazing is that none of our functions from `add`, `setProducts`
or `setPrice` contains any validation logic and we still always get valid objects.

The final part is the problem we already saw with *defensive copies*. If we
change the price of any product, it doesn't effect any `ProductsPriceOver`
object.

```fsharp
// Set Price of A to "1.00"
let newA = Product.setPrice 1.00m a

// A in ppo4 is still 9.99
printfn "%s" (ProductsPriceOver.toString ppo4)
// ["A -- 9.99"; "B -- 19.99"; "C -- 49.99"]
```

This should be obvious and is the reason why we need an `update` function.
When we update the price of a product, we need to create a new `ProductsPriceOver`
object and we pass it the Product that changed. We could pass `newA`
that we created and pass it to the `update` function. But actually, there
is no reason to create `newA` separately. We also could write:

```fsharp
let p1 = ProductsPriceOver.update (Product.setPrice 8.0m a) ppo4
ProductsPriceOver.toString p1
// ["A -- 8.00"; "B -- 19.99"; "C -- 49.99"]

let p2 = ProductsPriceOver.update (Product.setPrice 1.0m a) ppo4
ProductsPriceOver.toString p2
// ["B -- 19.99"; "C -- 49.99"]
```

Instead of creating a new object, and then pass it to update, we just
inline the whole function call. [In general immutability changes the way how
we write code either into a more sequential-style or a nested-style]({{< ref 2016-09-25-function-application-and-composition >}})

If you follow this idea further there are probably even more changes you want
to do, but all of those are outside the scope of this article. The important
aspect is if our objects stay valid.

We must create an `update` function that is not needed in an mutable version.
But even that can be considered as good. We didn't need to create an `update`
function with a mutable version because mutable Products already had this
feature. In fact the problem is more that you can forget this feature, and this
is the reason why an `ProductsPriceOver` in the mutable version can become
invalid, because you can forget a feature that msomehow must be handled.

The rules to create either events or defensive copies forces you to think about
this cases so you hopefully don't forget them. With immutability you cannot forget
anything that later on can invalidate anything. Your code only has those
features you also implemented!

# Conclusion

Working with immutability sure is different. I guess some people will also claim
that updating with immutability is harder. With mutability you just need to
change the Product and that's it.

```csharp
a.Price = 5.0;
```

The problem is that those things often miss the bigger picture. As just mutating
the price can cause problems in other code like we have seen previously. If we
start adding `copy` functions for defensive copying or need to add event handling
then this is not really simpler compared to immutability.

It is sure possible that you can create mutable objects that are always valid,
but doing so is a lot harder. Even now I'm still not 100% sure if I covered
ll possibilities how mutation can somehow lead to an invalid object.

There are more reasons for immutability, advantages and techniques we didn't
looked at. But there are also reasons against immutability. But in this article we
only looked at mutability vs. immutability in the context of maintaining
valid objects.

In my opinion, most of the time, this is the most important aspect we usually
care for. As a general thumb of rule I would claim that by default everything
should be immutable. Until of course there are other reasons why this
shouldn't be the case. What this other reasons could be are topics for other
articles.
