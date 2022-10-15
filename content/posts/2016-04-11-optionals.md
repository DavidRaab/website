---
layout: post
title: Optionals
slug: optionals-app
date: 2016-04-11
tags: [FSharp,"null",option]
description: "Describes Optionals as an alternative to null"
keywords: f#, fsharp, functional, option, null
---

In this post I want to talk about *Optionals* more deeply. I already wrote
[about null]({{< ref 2016-03-20-null-is-evil >}}), but I noticed that it is still
not immediately clear on why *Optionals* are better. Instead of focusing why `null`
is bad, this time I want to focus why *Optionals* are good. For this purpose I also wrote
a small application that I will cover. But first, let's go over *Optionals* and see
which benefits they have.

## `null` is not the problem

At first, I will state that `null` itself is not even the problem.

Let's assume we write a application and we need to read some *Product* entries from a
database. We just have a simple *id* as an `int` to identify a product.

Now let's assume we get some user input that tells us to show a Product with the id `12345`.
Sure, as long we only get request to products that exists, we don't run into any kind of problems.
But let's imagine we write a function that returns a *Product*, and the requested *Product* doesn't
exists. What do we do in such a case?

The usual approach, at least in OO programming, is to either return `null`, or throwing an
*Exception*. But i [don't think that Exceptions are better compared to null]({% post_url 2016-03-25-exceptions-are-evil %}).

So, why is returning `null` bad? Because a client calling our function is not forced to check
for `null`. And if he works with `null` as if he had a *Product*, the program will crash. What
happens if we throw an *Exception*? Well, our program still crashes!

The thing is. No matter if we return `null` or we throw *exceptions*. We must handle the case when
we don't get a *Product*. But `null` or *exceptions* don't force you to handle both cases.
Neither `null` or *exceptions* forces you to add checks. Both concepts just crashes your program
at runtime. The only *hope* you have is that you *hopefully* have a *test-suite* that *hopefully*
handles those cases. For me, that is too much of *hope*.

Yes, throwing an *Exception* is a *little bit* better. But think on why it is *a little bit better*.
We see it as an advantage as we *hopefully* (*gosh!*) already see any kind of error during development,
so we can add the needed `try/catch` statements.

But wouldn't it be better if we are forced to add the checks because the language forces us to
do it? Because we otherwise get a compile-time error? If we somehow could get this kind of behaviour
it would mean we never can forgot to add a `null` check or a `try/catch` statements. It will become
impossible to write programs that unexpectedly crashes at runtime.

Our program either compiles fine, and we handled all places where a **no value** could be returned,
or if we forgot to handle such a place we just get a compile-time error that gives us the exact place
and line where we forgot to handle such a case.

It seems like a dream. But this is exactly what *Optionals* gives us!

## Optionals

*Optionals* fixes that problem because it makes the idea of **No value** it's own type. The `option`
type in F# is defined as followed:

```fsharp
type option<'a> =
| Some of 'a
| None
```

What does that mean? You can compare it to a `bool` or an *enum* type. We have two cases. Like
a `bool` that either can be `true` or `false`. Here we have an `option` that either can be `Some`
or `None`. The difference is that the `Some` case can carry an additional value. Something
that an *enum* or a `bool` cannot do.

The `Some` or `None` are basically constructors. In the same sense that `true` or `false` are
constructors that creates a `bool`. As `None` don't carry any value, you just can write `None`,
the same you just can write `null`. But, if a function has a path that returns `None`, you
must ensure that all code paths return an `option` type. As we cannot create functions with
mixed return types. So you cannot write something like this:

```fsharp
let containsE (str:string) =
    if str.Contains("E")
    then str  // string
    else None // option
```

This would either return a `string` or an `option`. So what you must do is wrap your value in a `Some`.

```fsharp
let containsE (str:string) =
    if str.Contains("E")
    then Some str // option<string>
    else None     // option<string>
```

This has some implications:

1. Now you have a function that returns an `option` containing a `string`.
1. A user can see that `containsE` not always returns a value.
1. A user that wants to work with the result of `containsE` first must check which case he got.

The last implication means you must check if you either got `Some` or `None`

```fsharp
let opt = containsE "foo"
match opt with
| None     -> printfn "No E"
| Some str -> printfn "%s contains an e" x
```

But we also have a lot of helper functions in the `Option` module like `Option.map`,
`Option.iter`, `Option.bind` and so on that helps us working with `option` types.

For example it is quite common that we want to check if we have `Some value` and call a
function with `value`. But what do we do if we have `None`? Then we can't call
our function. This kind of stuff is what a `Option.map` does. So instead of

```fsharp
let x = Some 10
let y =
    match x with
    | None   -> None
    | Some x -> Some (x + 5)
```

we also can just write

```fsharp
let y = Option.map (fun x -> x + 5) x
```

Now, we are forced to handle `option` values. But `option` itself is it's own type and we have
a lot of functions that helps us working with `option` values.

## The Application

The best way to show the benefits and the difference is to go through a small example. For that
purpose I created a small in-memory database and a CLI program with basic CRUD operations. You
can see the full source code at the end. But i don't want to cover every detail. I want to focus on
the *Optional* part. First, let's see how we can use the program

    Command: show
      Id |                      Name | Price
       1 |                        TV | 499.99
       2 |                    A Book | 29.99
       3 |              Game Console | 349.99
    Command: asdasdfkhjb
    Error: invalid command -- [asdasdfkhjb]
    Command: delete 2
    Command: show 2
    Product with id 2 doesn't exists.
    Command: insert "Zelda: Skyward Swords" 49,99
    Command: show
      Id |                      Name | Price
       1 |                        TV | 499.99
       3 |              Game Console | 349.99
       4 |     Zelda: Skyward Swords | 49.99
    Command: name 1 "Television"
    Command: price 3 299,99
    Command: show
      Id |                      Name | Price
       1 |                Television | 499.99
       3 |              Game Console | 299.99
       4 |     Zelda: Skyward Swords | 49.99
    Command: exit

The program just contains the commands `show`, `insert`, `name`, `price`, `delete` and `exit` as
valid commands. As you can imagine we need to handle a lot of failures.

As usual, all kind of user input can be invalid. A command that doesn't exists. Parsing of
a number failed, in general a user just enters garbage. Or a user tries to change or show
a specific entry that doesn't exists.

As i don't want to cover the whole program, just let's look first at the modules and functions
and their signatures to get a overview of the code.

So let's just go through some of the interesting parts. At first, i added a `Option.IfNone`
function. It either returns a value if present or the provided value. It is just a call
to `defaultArg`. I created `ifNone` because the default argument order of `defaultArg`
doesn't work nicely with piping.

```fsharp
module Option =
    let ifNone x opt = defaultArg opt x
```

Here is a brief overview of the modules and functions i created.

```fsharp
module Product = begin
    val create    : int -> string -> decimal -> Product
    val withName  : string -> Product -> Product
    val withPrice : decimal -> Product -> Product
end

module DB = begin
    val getById    : 'a -> Map<'a,'b> -> 'b option
    val getAll     : Map<'a,'b> -> 'b list
    val containsId : 'a -> Map<'a,'b> -> bool
    val nextId     : Map<int,'a> -> int
    val insert     : 'a -> 'b -> Map<'a,'b> -> Map<'a,'b>
    val delete     : 'a -> Map<'a,'b> -> Map<'a,'b>
    val update     : 'a -> 'b -> Map<'a,'b> -> Map<'a,'b>
    val updateId   : ('a -> 'a) -> 'b -> Map<'b,'a> -> Map<'b,'a>
end

module CLI = begin
    val parseCommand   : string -> Command
    val printProduct   : Product -> unit
    val show           : Map<'a,Product> -> Map<'a,Product>
    val showProduct    : Map<int,Product> -> int -> Map<int,Product>
    val insert         : Map<int,Product> -> string -> decimal -> Map<int,Product>
    val updateName     : Map<'a,Product> -> 'a -> string -> Map<'a,Product>
    val updatePrice    : Map<'a,Product> -> 'a -> decimal -> Map<'a,Product>
    val executeCommand : Map<int,Product> -> Command -> Map<int,Product> option
    val eval           : Map<int,Product> -> string -> Map<int,Product> option
end
```

For the *in-memory* database i just used a `Map` without creating a new type,
that's why you see a lot of `Map` types. But overall you still see that we don't have
so few functions returning `option`.

At first we have the `Product` module. It just provides some helper functions to create and
modify the following `Product` Record type.

```fsharp
type Product = {
    Id:    int
    Name:  string
    Price: decimal
}
```

The `Product` type should contain all our Business logic, validation and so on. It should
only contain *pure* functions, usually this modules should contain the most code, but
in this example `Product` doesn't really do much, so it is the smallest module.

`DB` is basically an *Application layer* or *Service layer*. It just provides the code
to save things to a database or read things from a database. It is just a simple
*Key/Value* interface. As you can see from the types. It is fully *generic* and could
work with any type.

From the types, only `getById` returns an `option`. But that doesn't mean it is the only
function that has some `option` handling in it.

Finally, on top we have the `CLI` module. The purpose of it is to provide the logic
for the `CLI`. We parse our input string, interpret the commands and update the database.

## DB

`Map` itself is an *immutable* type. So the general idea is that operations like
`insert`, `delete`, `update` return the new state of `Map`.

```fsharp
let getById id db =
    Map.tryFind id db
```

Our `getbyid` function is fairly simple, as it is just a call to `tryFind`. But it makes
sense to talk about `tryFind` and compare it with a `Dictionary`. If you use a `Dictionary`
in C# and you try to fetch an entry you usually have two different behaviours.

Either you choose that retrieving an entry could throw an exception, or you use the
`TryGetValue` method. It doesn't throw an exception, but instead it returns a `bool` that
you must check, to get the value you have to pass an `out` parameter to `TryGetValue`.
I don't really like both ways. We don't want *exceptions* and wrap the code in `try/catch`.
But also `TryGetValue` is annoying as we first create a (mutable) value and pass
it to `TryGetValue`.

In F# on the other hand a function like `tryFind` just returns an `option`, we
either have `Some value` or `None`. Here we just return the `option` as-is. That is
also the reason why `getById` returns an `option`. We don't must immediately check the
`option`. We also can just pass it as a value around. We only must check it if we need
the value inside `Some`.

```fsharp
let update key value db =
    Map.add key value db

let updateId f key db =
    db
    |> getById key
    |> Option.map (fun value -> update key (f value) db)
    |> Option.ifNone db
```

Inside `DB` i added a `updateId` function. Usually i wouldn't add such a function, but
on the other hand, it is useful and at the same time interesting. The purpose of this
function is that we can fetch and update an entry at the same time. The interesting is:
`updateId` don't return an `option`.

At first we fetch the specified entry with `getById` that returns an `option`.
When we got a result we want to transform the `value`. That's why we have `(f value)`
in-there. The result of this is passed to update, that then returns a new updated
`Map`. But what happens if `getById` returns a `None` instead of a value? The `Option.map`
part will also directly return `None` instead of executing the `update` call.
But we always want to ensure to return a `Map`, so we either return a new updated `Map`
or in the case we had `None`. `Option.ifNone` returns the `Map` we started with, without
any change applied.

Assume you work in a language without `option` and you get a `null`. You theoretically could
write.

```fsharp
let updateId f key db =
    let value = db |> getById key
    update key (f value) db
```

But you will only notice that this is error-prone. As this code can just throw a
`NullReferenceException` if you try to update a non-existent value. Sure you then could add
the typical `null` checks to get it safe.

```fsharp
let updateId f key db =
    let value = db |> getById key
    if value <> null
    then update key (f value) db
    else db
```

The difference is, with an `option` you are forced to handle that case. You cannot
write error-prone code in the first-place! But in general it also shows that `updateId`
even the fact that an `id` could not be presented, still don't return an `option`. Actually
we have an operation that cannot fail here. Either it updates an entry, or it does
nothing. And you get that behaviour ensured by the type-system at compilation-time!

### CLI

The CLI handling is in general split into two parts. First I parse the input by the user
and I use a *Discriminated Union* to save the different input commands. The type is just
named `Command`. `parseCommand` do the transformation of converting the input `string`
to such a `Command`. The interesting thing is. Parsing could fail, but you don't see
a `Command option` for this function.

This is a general idea. Sure `option` is nice, but if you anyway build a custom type,
you can make the "None", "NotExistence" or "Invalid" a part of your type. Here I just
have a `Command` that contains an `Invalid` case.

```fsharp
type Command =
    | Invalid     of string
    | Show
    | ShowProduct of int
    | NewName     of int * string
    | NewPrice    of int * decimal
    | Insert      of name:string * price:decimal
    | Delete      of int
    | Exit
```

In general you can see that the *Discriminated Union* just contains a case for every CLI command,
it also contains the parsed input as `int`, `string` or `decimal`. In my code
I just use *Pattern Matching*, but with *Active Patterns* I can specify *transformation*
functions on top of it.

```fsharp
let (|Int|_|) input =
    match Int32.TryParse input with
    | false,_ -> None
    | true,x  -> Some x

let (|Decimal|_|) input =
    match Decimal.TryParse input with
    | false,_ -> None
    | true,x  -> Some x

let (|LC|) (str:string) = str.ToLowerInvariant()
```

`LC` in that example is a *Complete Active Pattern*. As it will always succeed. `LC` just
takes a `string` and turns it into a `lower-case` string. I use it like that

    | [| LC "show" |] -> Show

This not only a *Pattern Match* that checks if we have an array with "show" as the only
entry. It first transform the entry to a lower-case string and then compares it with "show".

Transforming a string to a lower-case string always succeed, but we also can use
*Partial Active Patterns* for operation that could fail. That is what `(|Int|_|)` and
`(|Decimal|_|)` stands for. Both operation try to parse a string as either `Int` or
`Decimal`. In a success case we return `Some`, otherwise `None`. But the handling of those
`option` is done for us. You also see something like that in other functions.

For example a `List.choose` is basically a `map` and then a `filter` operation in one
step. You not only can transform an entry to a new value. By returning `Some` or `None`
you also can filter. The `choose` operation only take the `Some` elements. Here we have
the same. Using *Options* for a *success/failure* or *filter* case is quite common.

So parsing and validation can be done in one-step. For example *parsing* the `price`
case looks like this.

    | [| LC "price"; Int id; Decimal price |] ->

We *Pattern Match* and only if we have an Array with three elements and the second element can
successfully transformed into an `int` and the third element can be turned into a `Decimal`,
only then the case successfully matches. `id` and `price` are also `int` and `decimal`
not `option`.

```fsharp
let showProduct db id =
    match db |> DB.getById id with
    | None   -> printfn "Product with id %d doesn't exists." id
    | Some p -> printProduct p
    db
```

Also the `CLI` functions have the idea that they just return the new `Map`. `showProduct`
is a operation that could fail, as the specified entry cannot exists. That's why we handle
both cases here. We either print a message that the Product didn't exists, or we print
the returned `product`. Because we expect the new `Map` state as a return value, but
`showProduct` never changes `Map`, we always just return `db` (the input `Map`).

```fsharp
let updateName db id newName =
    DB.updateId (Product.withName newName) id db

let updatePrice db id newPrice =
    DB.updateId (Product.withPrice newPrice) id db
```

Our `DB.updateId` already handled the `option` for us, that means we just can write
the `updateName` and `updatePrice` functions without thinking about `option`. And still
everything works as expected without any failure!

```fsharp
let executeCommand db command =
    match command with
    | Show               -> Some <| show db
    | ShowProduct x      -> Some <| showProduct db x
    | Insert(name,price) -> Some <| insert db name price
    | NewName(id,name)   -> Some <| updateName db id name
    | NewPrice(id,price) -> Some <| updatePrice db id price
    | Delete x           -> Some <| DB.delete x db
    | Invalid input      ->
        printfn "Error: invalid command -- [%s]" input
        Some db
    | Exit -> None
```

In `executeCommand` I use the `option` type for signalling if we still *continue* or want
to *stop*. The idea is once again that we just return a new `Map` after every command.
That `Map` contains the new state. But once I return `None` it marks an end. Here
you also see the mapping from the `Command` to the actual functions. The `Invalid`
case for example doesn't abort the program. We just print an error message and just return
`db` unchanged. Only the `Exit` command ends the program.

```fsharp
let eval db str =
    parseCommand str |> executeCommand db
```

Near the end I simplified the whole program into two function. Parsing a `string`
to a `Command`, and executing a `Command` that returns us the next `Map`. At this
level we just compose both operations into a single function. Now we have `eval`.

`eval` takes a `Map` and an input `string`, and will just return a new updated `Map`
for us. We already achieved a higher-level beyond error or option checking.

```fsharp
let main db =
    let rec loop db =
        printf "Command: "
        stdin.ReadLine() |> CLI.eval db |> Option.iter loop
    loop db
    ()
```

The only thing needed is the main program loop. We just print "Command: " to the terminal.
We read a string. Pass it to `Cli.eval db`. This will Parse our string, do all kind
of checking and just returns us a *eventually* a new `Map`. As long we get a new `Map`.
we just call `loop` again that recurs with the new `Map`.

```fsharp
let storage =
    Map.ofList [
        1, Product.create 1 "TV" 499.99m
        2, Product.create 2 "A Book" 29.99m
        3, Product.create 3 "Game Console" 349.99m
    ]

main storage
```

We create an *immutable* `Map` as our starting database. With `main storage` we finally
start our whole application loop.

## Further Reading

 * [The Option Type](http://fsharpforfunandprofit.com/posts/the-option-type/)
 * [Wikibook - Option Type](https://en.wikibooks.org/wiki/F_Sharp_Programming/Option_Types)
 * [[C#] Some optional, functional goodness in C#](http://www.davesquared.net/2012/12/optional-fp-in-csharp.html)

## Full Code

```fsharp
open System
open System.Text.RegularExpressions

module Option =
    let ifNone x opt = defaultArg opt x

type Product = {
    Id:    int
    Name:  string
    Price: decimal
}

[<CompilationRepresentation(CompilationRepresentationFlags.ModuleSuffix)>]
module Product =
    let create id name price =
        {Id=id; Name=name; Price=price}

    let withName newName product =
        {product with Name=newName}

    let withPrice newPrice product =
        {product with Price=newPrice}

module DB =
    let getById id db =
        Map.tryFind id db

    let getAll db =
        db |> Map.toList |> List.sortBy fst |> List.map snd

    let containsId id db =
        Map.containsKey id db

    let nextId db =
        (Map.fold (fun acc k v -> max acc k) 0 db) + 1

    let insert key value db =
        // Check if we have that key
        if   db |> containsId key
        then db                      // yes -- return "db" unchanged
        else db |> Map.add key value // no  -- add product

    let delete id db =
        db |> Map.remove id

    let update key value db =
        Map.add key value db

    let updateId f key db =
        db
        |> getById key
        |> Option.map (fun value -> update key (f value) db)
        |> Option.ifNone db

module CLI =
    // The CLI Commands
    type Command =
        | Invalid     of string
        | Show
        | ShowProduct of int
        | NewName     of int * string
        | NewPrice    of int * decimal
        | Insert      of name:string * price:decimal
        | Delete      of int
        | Exit

    // Parsing string to Command
    let (|Int|_|) input =
        match Int32.TryParse input with
        | false,_ -> None
        | true,x  -> Some x

    let (|Decimal|_|) input =
        match Decimal.TryParse input with
        | false,_ -> None
        | true,x  -> Some x

    let (|LC|) (str:string) = str.ToLowerInvariant()

    let parseCommand (input:string) =
        let args =
            Regex.Matches(input, "(?:\"(?<str>[^\"]+)\"|(?<str>\S+))")
            |> Seq.cast<Match>
            |> Seq.map (fun m -> m.Groups.["str"].Value)
            |> Seq.toArray
        match args with
        | [| LC "show" |]        -> Show
        | [| LC "show"; Int x |] -> ShowProduct x
        | [| LC "insert"; name; Decimal price |] ->
            Insert (name,price)
        | [| LC "name"; Int id; name |] -> NewName (id,name)
        | [| LC "price"; Int id; Decimal price |] ->
            NewPrice (id,price)
        | [| LC "delete"; Int x |]      -> Delete x
        | [| LC "exit" |] -> Exit
        | _               -> Invalid input

    // Helper functions
    let printProduct product =
        printfn "%4d | %25s | %4.2f" product.Id product.Name product.Price

    // CLI Commands
    let show db =
        printfn "%4s | %25s | %s" "Id" "Name" "Price"
        db |> DB.getAll |> List.iter printProduct
        db

    let showProduct db id =
        match db |> DB.getById id with
        | None   -> printfn "Product with id %d doesn't exists." id
        | Some p -> printProduct p
        db

    let insert db name price =
        let id      = DB.nextId db
        let product = Product.create id name price
        DB.insert id product db

    let updateName db id newName =
        DB.updateId (Product.withName newName) id db

    let updatePrice db id newPrice =
        DB.updateId (Product.withPrice newPrice) id db

    // Execution of CLI Commands
    let executeCommand db command =
        match command with
        | Show               -> Some <| show db
        | ShowProduct x      -> Some <| showProduct db x
        | Insert(name,price) -> Some <| insert db name price
        | NewName(id,name)   -> Some <| updateName db id name
        | NewPrice(id,price) -> Some <| updatePrice db id price
        | Delete x           -> Some <| DB.delete x db
        | Invalid input      ->
            printfn "Error: invalid command -- [%s]" input
            Some db
        | Exit -> None

    let eval db str =
        parseCommand str |> executeCommand db

let main db =
    let rec loop db =
        printf "Command: "
        stdin.ReadLine() |> CLI.eval db |> Option.iter loop
    loop db
    ()

// Start with some default entries
let storage =
    Map.ofList [
        1, Product.create 1 "TV" 499.99m
        2, Product.create 2 "A Book" 29.99m
        3, Product.create 3 "Game Console" 349.99m
    ]

// Start program loop
main storage
```