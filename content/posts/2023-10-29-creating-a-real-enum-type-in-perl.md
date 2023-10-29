---
layout: post
title: "Creating a real enum Type in Perl"
slug: creating-a-real-enum-type-in-perl
date: 2023-10-29
tags: [perl,metaprogramming,enum,data]
description: With the help of metaprogramming I create enum function to create real enums.
---

In my previous article about [Metaprogramming in Perl]({{< ref 2023-10-26-meta-programming-in-perl >}}) I show how to implement an `enum()` function that creates real special values that are distinct from each other.

When I called `enum('red', 'green', 'blue')` it created two functions for every passed string. `red()` and `is_red()` for the `red` string. `red()` returns the special value and `is_red($value)` checks if `$value` is our special value.

But currently it does not completely satisfy being an *enum*. For example when I call:

```perl
enum(qw/red blue green/);
enum(qw/true false/);
```

then it creates 5 different special values. But none of the special value are related to each other. For example I have no direct way to check if a color was passed. I could write a function like `is_color` that checks if a value satisfies one
of `is_red`, `is_blue`, `is_green`. But writing them is boilerplate.

Instead I want to create such a function automatically.

But first I think about how the API of the function `enum` should be changed. For example it would make sense to add a value to provide a type. I could for example expect that the enum function works this way.

```perl
enum('color', 'red', 'blue', 'green');
```

One feature I like in Perl is the `=>` operator. Actually it is the same as `,` but considers everything on his left-side as a string. This way we don't need to put quotes around a string. So we can write:

```perl
enum(color => 'red', 'blue', 'green');
```

This looks pretty good for defining an `color` enum. This function could for example create an `is_color()` function that returns a truish-value when we pass it `red`, `blue` or `green`.

But what happens when we have something like this?

```perl
enum(color => 'red', 'blue', 'green');
enum(error => 'red', 'yellow');
```

Now we have two different enums, but both use the name `red`. The problem is, that we have a collision of names. Maybe it would be even better, when we prefix all generated functions! So for example the first call to `enum` creates the following functions: `is_color`, `color_red`, `is_color_red`, `color_blue`, `is_color_blue`, `color_green` and `is_color_green`.

The second call to enum creates: `is_error`, `error_red`, `is_error_red`, `error_yellow` and `is_error_yellow`.

This way we can avoid collision with names. One last thing I would change, from my experience, is to not use variable arguments. Variable arguments have the problem to make the API of a function less extendable.

```perl
enum(color => ['red', 'blue', 'green']);
enum(error => ['red', 'yellow']);
```

So instead of using variable arguments I would expect an array-reference as the second argument to our `enum` function. But thinking about the design doesn't stop here. Positional arguments are not so common in Perl. We find them in the built-in functions, but most modules rather use named arguments. So we also could use the following API.

```perl
enum({ type => "color", values => ['red', 'blue', 'green'] });
enum({ type => "error", values => ['red', 'yellow']        });
```

For Perl it is not uncommon to use such kinds of APIs. The advantage? It is better to read as each parameter explains itself. On top it can be better extended. Now we have two unpositional arguments. When we feel like it, we can easily add new named arguments and provide default values for those.

This style is so common in Perl because Perl don't have method overloading. We even can go one step further. Why even bother of calling `enum` multiple times? We also could choose this kind of API.

```perl
enum([
    { type => "color", values => ['red', 'blue', 'green'] },
    { type => "error", values => ['red', 'yellow']        },
]);
```

This way we can define all enums all at once. Actually we can even allow both!

When the `enum` functions get passed a hash-reference, we assume
that we only want a single enum to be created. When we pass it an
array-reference we assume an array of hash-references that initialize multiple enums at once.

Always remember: There is more than one way to do it. Like when we have sex. We don't want only one way to fuck, we want multiple ways and not a dictator telling us how we are allowed and supposed to fuck.

# Implementing enum

I decided I want to implement the latest API I described. Here is how I start. First of, even the fact that I know what my goal is, I don't start there.

First I will try to just implement `enum(color => ['red', 'blue', 'green']);` and make the function work. That means, creating a function that creates `is_color`, `color_red`, `is_color_red` and so one ...

# First iteration

```perl
sub enum($type, $values) {
    no strict 'refs';

    my @is_functions;
    for my $name ( @$values ) {
        # this creates the function name like "color_red"
        my $fn_name = sprintf("%s_%s", $type, $name);
        # this then creates "color_red" and "is_color_red"
        create_special($fn_name);
        # This way i get a reference to the created "is_*" function
        # and collect them in @is_functions
        push @is_functions, *{"is_".$fn_name}{CODE};
    }

    # Now i want to create a new function `is_$type` that checks if any
    # value passed to it will return true for any function inside
    # @is_functions

    *{ "is_" . $type } = sub {
        my ($value) = @_;
        # we start with false
        my $is_type = 0;
        for my $func ( @is_functions ) {
            # we pass the $value to function
            $is_type = $func->($value);
            # and if it is true we abort the loop
            last if $is_type;
        }
        return $is_type;
    }
}
```

Now we can test our function:

```perl
BEGIN {
    enum(color => [qw/red green blue/]);
    enum(error => [qw/red yellow/]);
}

# We create a color_red and an error_red value
my $cred = color_red;
my $ered = error_red;

printf("is_color \$cred: %d\n", is_color($cred)); # 1
printf("is_color \$ered: %d\n", is_color($ered)); # 0
printf("color red: %d\n", is_color_red($cred));   # color red: 1
printf("color red: %d\n", is_color_red($ered));   # color red: 0
printf("error red: %d\n", is_error_red($cred));   # error red: 0
printf("error red: %d\n", is_error_red($ered));   # error red: 1
```

As we can see the functions are created. Now we have an additional function to check the enum type itself.

# Second iteration

The function works, now I want to implement the named arguments. I could change the `enum` function, but instead I decide to create a new one first. I name it `enum_named` and should be called like this:

```perl
enum_named({
    type   => 'color',
    values => [qw/red green blue/],
});
```

This is the implementation and the tests for it.

```perl
sub enum_named($args) {
    my $type   = $args->{type}   // die "Argument 'type' not passed.\n";
    my $values = $args->{values} // die "Argument 'values' not passed.\n";
    enum($type, $values);
}

# create our special values
BEGIN {
    enum_named({
        type   => 'color',
        values => [qw/red green blue/],
    });
    enum_named({
        type   => 'error',
        values => [qw/red yellow/],
    });
}

my $cred = color_red;
my $ered = error_red;

printf("color red: %d\n", is_color_red($cred)); # color red: 1
printf("color red: %d\n", is_color_red($ered)); # color red: 0
printf("error red: %d\n", is_error_red($cred)); # error red: 0
printf("error red: %d\n", is_error_red($ered)); # error red: 1
```

The `enum_named` actually don't do much. I just add checks that ensures a user passes the required `type` and `values` key. If not, the function throws an exception aborting the program.

When we want, we could add additional checks. For example if the user passed a string to the `type` argument. Or the argument to `values` is an array-ref. Or the string is valid and so on. But I think it is good enough.

# Third iteration

Now I want to implement the final `enum` that i can either pass a hash-reference or an array of hash-references. Again, I create a new function for this. But before doing so I will rename the `enum` function i have so Far. I will rename `enum` to `enum_positional`. For this I also need to update `enum_named`. But once done I have two functions i can use. `enum_positional` and `enum_named`.

Then I implement the new `enum` function to use those. Here is my implementation of it.

```perl
sub enum($data) {
    my $type = reftype($data) // "";

    # when user pass a single hash, just call enum_named
    if ( $type eq 'HASH' ) {
        enum_named($data);
    }
    # when user passed an array
    elsif ( $type eq 'ARRAY' ) {
        # we call enum_named for every hash in the array
        for my $hash ( @$data ) {
            enum_named($hash);
        }
    }
    else {
        die "You need to either pass a hashref or arrayref to enum.\n";
    }
}
```

The implementation is pretty easy for it. Now a user have multiple
ways to create *enums*.

```perl
BEGIN {
    enum_positional(decision => [qw/yes no/]);
    enum({
        type   => "bool",
        values => [qw/true false/],
    });
    enum([
        { type => 'color', values => [qw/red green blue/] },
        { type => 'error', values => [qw/red yellow/]     },
    ]);
}

my $cred = color_red;
my $ered = error_red;

printf("color red: %d\n", is_color_red($cred)); # color red: 1
printf("color red: %d\n", is_color_red($ered)); # color red: 0
printf("error red: %d\n", is_error_red($cred)); # error red: 0
printf("error red: %d\n", is_error_red($ered)); # error red: 1

printf("is_bool: %d\n", is_bool(bool_true));    # is_bool: 1
printf("is_bool: %d\n", is_bool(bool_false));   # is_bool: 1

printf("is_decision: %d\n", is_decision(bool_true));    # is_decision: 0
printf("is_decision: %d\n", is_decision(decision_yes)); # is_decision: 1
```

With Perl I learned the following design principle. The function that you want a user to use, should be the shortest named function. All others have longer names indicating to the user he probably shouldn't use them.

# Fourth iteration

While I have written the examples I somehow was annoyed about the **Bool** case. We now can create a boolean type with:

```perl
enum({type => "bool", values => [qw/true false/]});
```

but isn't it annoying that we have to write `bool_true` and `bool_false` instead of just being able to write `true` and `false`?

There was a reason for it why I prefixed all created functions with the type. By default for all kind of enums this is probably useful. But sometimes I want more control about the function name that is created. So i want to add this feature.

At the moment we pass an arrayref to `values`. And this arrayref just contain every case we want to create. Instead we also could allow to pass an hash instead. When an hash is passed the function could be called like this.

```perl
enum({
    type   => "bool",
    values => {
        true  => { value => 'true',  check => 'is_true'  },
        false => { value => 'false', check => 'is_false' },
   },
});
```

When the user passes a hash to *values* instead of an array then the keys of the hashes are the cases. One reason I don't like this is because the hash keys are actually never used and don't matter.

Instead of an hash I can expect an Array of Hashes, like this:

```perl
enum({
    type   => "bool",
    values => [
        { value => 'true',  check => 'is_true'  },
        { value => 'false', check => 'is_false' },
    ],
});
```

First i thought it is annoying to check if all values in an array are either a string or an hashref, but actually we can allow to pass either a string or an hashref. For example.

```perl
enum({
    type   => "bool",
    values => [
        "true",
        "false",
        { value => 'true',  check => 'is_true'  },
        { value => 'false', check => 'is_false' },
    ],
});
```

This would actually create four special values. Passing it a string works like we are used so far. Passing a hash allows to configure the function names being created.

In this case the *bool* type would have four values instead of just two.

The advantage of this design is that most of the time we just can use the default, and only for special reason we can overwrite the function names.

How about defining multiple functions for a case? Could also be possible. Maybe like:

```perl
enum({
    type   => "bool",
    values => [
        { value => [qw/t true/], check => 'is_true'  },
        { value => 'false',      check => 'is_false' },
    ],
});
```

In that case we would create the function `t` and `true` for the
same value. Could you come up with other descriptions and features?

For my case I think i will stick with the idea that instead of just being able to pass strings I also can pass a hashref.

To implement this I have done multiple changes.

1. `create_special` expect both function names to be passed.
2. I renamed `enum_positional` to `enum_hash` and changed it to
only accept array of hashes for the `values` parameter.
3. I created a new function `enum_positional` that also allows *strings* to be passed. Strings will be converted to the hash that are then passed to the `enum_hash` function call.

This is my final result.

```perl
#!/usr/bin/env perl
use v5.36;
use open ':std', ':encoding(UTF-8)';
use Scalar::Util qw(refaddr reftype);

# create_special('red', 'is_red');
#
# Creates a special value and we pass it the function names it should create
sub create_special($name, $is_name) {
    # disable strict refs inside this function
    no strict 'refs';
    # the special value
    my $special = [];

    # create $name function that returns the special value
    *{ $name } = sub {
        return $special;
    };

    # create function to check if any value is $special
    *{ $is_name } = sub {
        my ($value) = @_;
        my $type = reftype($value) // "";
        # check if arrayref was passed
        if ( $type eq 'ARRAY' ) {
            # check if adresses are the same
            if ( refaddr($value) == refaddr($special) ) {
                return 1;
            }
            else {
                return 0;
            }
        }

        return 0;
    };
}

# enum_hash(color => [{value => "red", check => "is_red"}]);
#
# $values must be an array of hashes (AoH)
sub enum_hash($type, $values) {
    no strict 'refs';

    my @is_functions;
    for my $hash ( @$values ) {
        # this creates both functions
        create_special($hash->{value}, $hash->{check});
        # This way i get a reference to the created check function
        # and collect them in @is_functions
        push @is_functions, *{ $hash->{check} }{CODE};
    }

    # Now i want to create a new function `is_$type` that checks if any
    # value passed to it will return true for any function inside
    # @is_functions

    *{ "is_" . $type } = sub {
        my ($value) = @_;
        # we call every check function and return 1 if one
        # of those function returns a truish value
        for my $func ( @is_functions ) {
            return 1 if $func->($value);
        }
        return 0;
    }
}

# enum_positional(color => ["red", "green", {value => "yellow", check => "is_yellow"}])
#
# In the arrayref of enum_positional there can be passed a string or
# a hash. strings that are passed are converted to the hash-call
sub enum_positional($type, $arrayref) {
    # Create a new arrayref with all string converted to a hash
    my $args = [map {
        my $ref = reftype($_);
        # when not defined - the value $_ is a string/float
        if ( not defined $ref ) {
            {
                value => sprintf("%s_%s",    $type, $_),
                check => sprintf("is_%s_%s", $type, $_),
            }
        }
        # when already a hash - keep it unchanged
        elsif ( $ref eq 'HASH' ) {
            $_
        }
        # throw error in all other cases
        else {
            die "enum_positional only expect string or hashref.\n";
        }
    } @$arrayref];

    # call enum_hash with the newly created $args
    enum_hash($type, $args);
}

sub enum_named($args) {
    my $type   = $args->{type}   // die "Argument 'type' not passed.\n";
    my $values = $args->{values} // die "Argument 'values' not passed.\n";
    enum_positional($type, $values);
}

sub enum($data) {
    my $type = reftype($data) // "";

    if ( $type eq 'HASH' ) {
        enum_named($data);
    }
    elsif ( $type eq 'ARRAY' ) {
        for my $hash ( @$data ) {
            enum_named($hash);
        }
    }
    else {
        die "You need to either pass a hashref or arrayref to enum.\n";
    }
}
```

And here is how all of it will be used:

```perl
# create our special values
BEGIN {
    enum_positional(decision => [qw/yes no/]);
    enum_positional(option => [
        { value => "some", check => 'is_some' },
        "none",
    ]);

    # Creates one enum
    enum({
        type   => "bool",
        values => [
            { value => 'true',  check => 'is_true'  },
            { value => 'false', check => 'is_false' },
        ],
    });

    # Creates multiple enums at once
    enum([
        { type => 'color', values => [qw/red green blue/] },
        { type => 'error', values => [qw/red yellow/]     },
        { type => 'fail',  values => [
            { value => 'ok', check => 'is_ok' },
            'err',
        ]},
    ]);
}

my $cred = color_red;
my $ered = error_red;
printf("is_color \$cred: %d\n", is_color($cred));     # 1
printf("is_color \$ered: %d\n", is_color($ered));     # 0
printf("is_cred \$cred: %d\n",  is_color_red($cred)); # 1
printf("is_cred \$ered: %d\n",  is_color_red($ered)); # 0
printf("is_ered \$cred: %d\n",  is_error_red($cred)); # 0
printf("is_ered \$ered: %d\n",  is_error_red($ered)); # 1


printf("is_bool true:  %d\n",  is_bool(true));  # 1
printf("is_bool false: %d\n",  is_bool(false)); # 1
printf("is_bool \$cred: %d\n", is_bool($cred)); # 0


printf("is_decision true: %d\n",         is_decision(true));         # 0
printf("is_decision decision_yes: %d\n", is_decision(decision_yes)); # 1


my $some = some;
my $none = option_none;
printf("is_some \$some: %d\n",   is_some($some));        # 1
printf("is_none \$none: %d\n",   is_option_none($none)); # 1
printf("is_option \$some: %d\n", is_option($some));      # 1
printf("is_option \$none: %d\n", is_option($none));      # 1


my $ok  = ok;
my $err = fail_err;
printf("is_fail \$ok: %d\n",    is_fail($ok));  # 1
printf("is_fail \$error: %d\n", is_fail($err)); # 1

```

I hope you liked the content. Are there things you would have done differently? What did you like, what do you don't like? Other suggestions or comments? Share your opinion!