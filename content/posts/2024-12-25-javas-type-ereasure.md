---
layout:  post
title:   "Java's type ereasure"
slug:    javas-type-ereasure
date:    2024-12-25T00:00:00
lastmod: 2024-12-25T00:00:00
tags:    [java,types]
draft:   true
description:
---

Here is some knowledge that i gathered how Java's typing system works. Don't
know which version. But it's an older one. Somwhere around Java 6 to Java 8.
Maybe it changed now.

Consider that generics was added to Java, and it was not a default of the language.
When Java would have started with Generics, it might have evolved different.

So a common problem you have is storing data in an Array. That's also basically
when you start C. You not only want to store one thing. But many. But in C
an array is of fixed-size. You cannot grow and shrink it.

In Perl we use

```perl
push @array, $x;
```

to push one element to an array. But Perl is written in C. So how does C solve
that problem? Well here are things you can do.

* Allocate an array of the size of the element you already have + 1
* Double the size. When you have an array with 4 entries. Allocate 8. And so on.

So which one do we pick?

The first solution has the advantage of being very memory efficent. No extra
space is wasted. Back in the old days that was very important. My first computer
had 4 MiB of RAM. Now 16 GiB are the "normal" even for consumer PC.


