---
title: "Draft Extendability Data vs Classes"
date: 2023-02-11T19:02:27+01:00
draft: true
---

How do you add behaviour to a class? You must inherit. What
happends if three different people inherit from a class to provide
additional behaviour. How do you get all three extensions into one?

Single-inheritance has a problem here. Mutliple inhertance not.
C# added this with Extension Methods.

In a functional language that separates data from functions you are
free to use any library with your data. There is no problem of combining
and using any function.

The only requirement is that you can freely create and read any data.
