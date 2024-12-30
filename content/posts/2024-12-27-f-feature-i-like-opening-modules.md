---
layout:  post
title:   "F# Feature I Like: Opening Modules"
slug:    fsharp-feature-i-like-opening-modules
date:    2024-12-27T00:00:00
lastmod: 2024-12-27T00:00:00
tags:    []
description:
draft: true
---

One thing I Like is the following. Let's say i have that new Array function
that is pretty useful. In F# i just can open any module, and just add functions
to it.

```fsharp
module List =
    let printArray = ...
```

absolutely fine.

Why is this so complicated and horrible in other languages? I can pick modules
that each extend. But you cannot overwrite functions. Not without hacks.

It's better as

    List.func1()
    Lst.func1()
    Maga.List.func3()
    xx.List.func4()
    MOOOOARList.func5()
    ListExt3.func6()

you know what i mean?

C# brought back this Feature with Extension Methods. Now you could extend
every object by loading some libraries that all extend that single object.

You get the same when you just allow it is a feature in the language to
open every module.

Perl has that too.