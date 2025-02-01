---
layout:  post
title:   "Quick: Useless Classes"
slug:    quick-useless-classes
date:    2025-02-01T00:00:00
lastmod: 2025-02-01T00:00:00
tags:    [perl,quick]
description:
---

Here is a quick reminder. When you have a class/object that only has a single
method.

```perl
my $obj = Class->new(
    arg1 => "whatever",
    arg2 => 10,
);

my $result = $obj->one_method(
    arg3 => [],
    arg4 => {},
);
```

then just make a single function out of it.

```perl
my $result = one_method(
    arg1 => "whatever",
    arg2 => 10,
    arg3 => [],
    arg4 => {},
);
```

this also works with multiple methods, but with just a single method it defenitely
should just be a single function. Also consider that Perl has great ability
for passing arguments. For example you also can do.

```perl
my %default = (
    arg1 => "whatever",
    arg2 => 10,
    arg3 => [],
);

my $res1 = one_method(%default, arg4 => { ... });
my $res2 = one_method(%default, arg4 => { ... });
```

# Related

* [OOP is Partial Application]({{< ref "2024-06-10-oop-is-partial-application.md" >}})

In that example, the hash `%default` basically acts as your *object*.