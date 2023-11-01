---
layout: post
title: "Object-Oriented Programming in C"
slug: object-oriented-programming-in-c
date: 2023-11-01
tags: [oop,c,csharp,design-pattern]
description: Explains how object-orientation is done in C. And what object-orientation is all about.
---

Let's do object-oriented programming in C. First of I am creating an easy example for an Vector2 type. To define a Vector2 in C we use a *struct*.

```c
struct Vector2 {
    int X;
    int Y;
};
```

When you know C#, you are already familiar with a `struct`. A `struct` is a value-type in C. It always gets copied as a whole. In C# we have a class that is a *reference-type* but this doesn't exist in C. But, we can make one. In C we always can create a *Pointer* to any value. A *pointer* is like a *reference* you are used in more "modern" languages.

In C we can do *pointer-arithmetic* and add, subtract or change a *pointer*, but here we will not do this. On a *reference* this is forbidden. But still the concept behind it is the same.

Creating an instance of `Vector2` becomes:

```c
struct Vector2 v1 = { 3, 4 };
```

Now I want to add an easy way to just print an vector. This is how the method `vector2_show()` will look.

```c
void vector2_show(struct Vector2 v) {
    printf("X=%d; Y=%d\n", v.X, v.Y);
}
```

Now we can do:

```c
struct Vector2 v1 = { 3, 4 };
vector2_show(v1);
```

it will print ...

```
X=3; Y=4
```

... to the console.

Let's add an setter to change `X` value of an `Vector2`. We implement it like this.

```c
void vector2_setX_1(struct Vector2 v, int x) {
    v.X = x;
}
```

but somehow this doesn't work as expected. When we write the following code.

```c
struct Vector2 v1 = { 3, 4 };
vector2_show(v1);

vector2_set_x(v1, 4);
vector2_show(v1);
```

no change has been happening. It will just print the same vector twice.

```c
X=3; Y=4
X=3; Y=4
```

The reason for this is as you guessed. Because `Vector2` is a *value-type* and not a reference type. So instead of directly passing a `Vector2` as a copy, we just want to pass it a *pointer* to a `Vector2` instead. A pointer-type in C are declared with an additional `*`.

We can get a pointer to any variable by just adding a `&` (*address-operator*) before it. So we can do the following.

```c
struct Vector2   v1 = {3, 4};
struct Vector2 *pv1 = &v1;
```

Here `pv1` is now a pointer to `v1`. But we cannot pass a pointer to the `vector2_set_x()` or `vector2_show()` methods we implemented so far because `vector2_show()` expected a `Vector2` not a *pointer* to a `Vector2`.

Additionaly when we pass a pointer, we need to *de-reference* the pointer to the actual value. How do we do this? Perl programmers are familiar with this. We *de-reference* a pointer with the arrow `->` operator.

```c
void vector2_show(struct Vector2 *v) {
    printf("X=%d; Y=%d\n", v->X, v->Y);
}

void vector2_set_x(struct Vector2 *v, int x) {
    v->X = x;
}
```

Now we can do the following.

```c
struct Vector2   v1 = {3, 4};
struct Vector2 *pv1 = &v1;

vector2_set_x(pv1, 4);
vector2_show(pv1);
```

Running this code will print us.

```c
X=3; Y=4
X=4; Y=4
```

But when I want to add a constructor this way, it will not work.

```c
struct Vector2* vector2_new(int x, int y) {
    struct Vector2 v = { x, y };
    return &v;
}
```

`gcc` produces the following warning.

```
vector2.c: In function ‘vector2_new’:
vector2.c:11:12: warning: function returns address of local variable [-Wreturn-local-addr]
   11 |     return &v;
      |            ^~
```

Trying to run the compiled program produces a *Segmentation Fault* on my machine. But behavior could be different. C is actually **not** a very strongly typed language.

Why does the constructor not work? Because we are creating a local variable. And those local variables are created on the *Stack* not in the *Heap*. A *Stack* variable will be freed after the function finish. So we cannot return a pointer to a *local* variable. This will cause problems.

Instead we need to create a `Vector2` in the *Heap*. Then we are allowed to return a pointer. Actually the function `malloc` in C does that. It allocates memory on the *Heap* and return a pointer to the allocated memory. Here are all the corrected constructor and methods so far.

```c
struct Vector2* vector2_new(int x, int y) {
    struct Vector2 *obj = malloc(sizeof(struct Vector2));
    obj->X = x;
    obj->Y = y;
    return obj;
}

void vector2_show(struct Vector2 *this) {
    printf("X=%d; Y=%d\n", this->X, this->Y);
}

void vector2_set_x(struct Vector2 *this, int x) {
    this->X = x;
}
```

Now we can write:

```c
struct Vector2 *p_v = vector2_new(1,1);
vector2_show(p_v);

vector2_set_x(p_v, 4);
vector2_show(p_v);
```

and it will print:

```
X=1; Y=1
X=4; Y=1
```

But remember. C does not have *automatic memory-management*. It doesn't automatically free any memory. You must explicitly free memory calling the `free()` function. This is an example I prefer to be implicit instead of being *explicit*. Other programmers will have a different opinion about this.

If you don't free the memory, you will have [Memory Leaks](https://en.wikipedia.org/wiki/Memory_leak) in your program. Also be aware if you `free()` memory and still try to access (Read/Write) to a pointer, bad things will happen.

Here is a full example.

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

struct Vector2 {
    int X;
    int Y;
};

struct Vector2* vector2_new(int x, int y) {
    struct Vector2 *obj = malloc(sizeof(struct Vector2));
    obj->X = x;
    obj->Y = y;
    return obj;
}

void vector2_show(struct Vector2 *this) {
    printf("X=%d; Y=%d\n", this->X, this->Y);
}

void vector2_set_x(struct Vector2 *this, int x) {
    this->X = x;
}

void vector2_set_y(struct Vector2 *this, int y) {
    this->Y = y;
}

double vector2_length(struct Vector2 *this) {
    return sqrt((this->X * this->X) + (this->Y * this->Y));
}

int main (int argc, char **argv) {
    struct Vector2 *v = vector2_new(1,1);

    // This prints X=1; Y=1
    vector2_show(v);

    // Setting X to 3
    vector2_set_x(v, 3);

    // prints X=3; Y=1
    vector2_show(v);

    // calculating length of vector2
    double length = vector2_length(v);

    // Prints: Length=3.162278
    printf("Length=%f\n", length);

    // Free memory -- when v not needed anymore
    free(v);

    return 0;
}
```

You can compile this file with `gcc vector2.c -o vector -lm`. When running, it will print.

```c
X=1; Y=1
X=3; Y=1
Length=3.162278
```

# What is object-orientation?

Object-Orientation was basically just a pattern in C. At some time people decided that instead of a *Design Pattern* it should be a language feature.

For better or for worse. I think *object-orientation* made programming a lot more worse. People acted as as if it would be something never seen. Fanatics all over the world started to came upon there graves telling that everything not object-oriented must be bad, and must be converted. The dark ages of programming started.

*Object-orientation* is useful and has it purpose. It fits the time perfectly. When you have limited memory and computational resources, it makes sense to pass pointers instead of copying structs over and over again.

And still, when you look to the C# community as an example, those programmers start to realize more and more the importance of allocating things on the Stack and why it is important to also be able to allocate classes on the *Stack* instead of always
allocating them in the *Heap*. Bypassing the garbage collector for performance.

C# started by making the distinction not be seen by a user. But still the distinction exists. Starting programmers **must** learn the distinction between *Value-Type* and a *Reference-type* in C#. Otherwise there will have a really bad time.

C# nowadays adds back a lot of the stuff to create a class on the stack. Returning reference on structs and so on. After they dismissed the features from `C`, decades later they started to re-implement then again. [How ironic](https://www.youtube.com/watch?v=Jne9t8sHpUc).

Doesn't this make you think. Wasn't it better how `C` made it? I really think that making stuff *explicit* is at some cases better. Handling of pointers is such a case.

It also shows you one of the most important aspect of what *object-oriented* programming really is.

**Object-oriented** programming is nothing more than passing a data-structure to a function and directly mutate the structure.

"Modern" languages like C# just have a different syntax. Instead of

```C
vector2_length(obj);
```

now it becomes

```csharp
obj.length();
```

but under the hood, nothing really changed at all. Both are functions getting `obj` as the first argument. That's why in *Perl* or *Python* you explicitly have `$self` as an argument on every method.

`C#` hides it, but it gives you the special `this` variable instead. Did you ever implemented a *Extension Method* in C#? For example.

```csharp
public static bool IsGreaterThan(this int i, int value) {
    return i > value;
}
```

Now you probably understand why you write `this int i` as the first argument. In C, you don't even **need** this *feature*.