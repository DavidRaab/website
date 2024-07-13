---
layout:  post
title:   "Debian Tool: Purging config of removed packages"
slug:    debian-tool-purging-config-of-removed-packages
date:    2024-07-13T00:00:00
lastmod: 2024-07-13T00:00:00
tags:    [linux,debian,tool,perl,ipc]
description: A small tool for purging the configs of all removed packages. With some explanation of doing IPC in Perl.
---

In Debian the following "problem" sometimes arrives. Let's say you install
for example *Apache2*, you do something with it but shortly after you
don't need it anymore. By default Debian always keeps all configuration files
alive. So after installing Apache2 once, you are left with some configuration
files in `/etc/apache2/`.

This is usually a good idea, because removing a package and re-installing
shouldn't delete all your configuration. But what do you do if you really
want to delete the configuration because you really don't need them anymore?

Or even better, how can we get a list of removed packages that still has
some configuration file left in the system and possible remove it?

Here is a shell one-liner you could use:

    dpkg --purge $(dpkg -l | grep "^rc" | perl -ane 'print "$F[1] "')

for some time I used this line above, but it has some disadvantages.

1. When there is nothing to remove it throws a weird error-message. (Beause nothing is passed to `dpkg --purge`).
2. You cannot see what will be removed, it immediately starts purging the config. (eliminating the `dpkg --purge` will show).

So i thought of improving that one liner and make it into its own script.
Here is the the full result of it. After that i wanna shortly talk about
how to execute sub-process in Perl and a little bit of how everything works.

```perl
#!/usr/bin/env perl
use v5.36;
use open ':std', ':encoding(UTF-8)';
use Getopt::Long::Descriptive;

my ($opt, $usage) = describe_options(
    'Usage: %c %o',
    ['force|f', 'force action, otherwise does nothing', {default => 0}],
    ['help|h',  'Print this message',                   {shortcircuit => 1}],
);

$usage->die if $opt->help;

open my $fh, '-|', 'dpkg -l';
# read only "rc" (removed config) packages - means packages that
# are removed but configuration files are still left in the system
my @packages = map { m/\A rc \s+ (\S+)/xms ? $1 : () } <$fh>;
close $fh;

if ( @packages ) {
    printf "Configuration from already removed packages found:\n";
    printf "  + %s\n", $_ for @packages;
    if ( $opt->force ) {
        system('dpkg', '--purge', @packages);
    }
    else {
        printf "\nNothing removed. Pass --force to command.\n";
    }
}
else {
    say 'No packages with remaining configs are found.';
}
```

Executing this script without any arguments now prints me a list of packages
telling me which configuration for which packages are gonna be removed, but
by default does nothing. If nothing needs to be done, it tells me instead of
giving me some kind of error message. Only passing `--force` to the script will do
the action and starts removing the configs of the packages.

# IPC - Interprocess communication in Perl

IPC is a broad topic in general but here let's focus on spawning other process
from Perl. The language itself offers three easy ways on achiving typical
tasks you might wanna do. Those are using `system`, **backticks** and using
`open` for creating a pipe.

## system

`system` is probably sometimes the most used one. `system` just spawns a process,
waits for it to finish and then the execution of the perl code continues. When
you just spawn a linux command and don't care for whatever it outputs, or
the output of the command is what the user should see, then this is what you
should do.

There is only one best practice I would offer. Instead of

    system("command -a $var1 -b $var2");

you should call it this way.

    system("command", '-a', $var1, '-b', $var2);

The first version has the typical, I call it *Bash-crapiness* in it. It suffers
from all kind of escaping problems. When a variable has spaces in it, newlines
and all other things the whole command usually breaks. It is much familiar
to SQL injection security flaws.

The second version don't have that problem. Here the content of `$var1` is passed
as the argument to `-a`. Whatever `$var1` will contain. All kind of escaping is
done for you. Even if the string itself contains `"` and so on it will do what
you probably expect.

Also using the second version usually does not spawn a *shell* (like bash/dash).
It immediately executes the command by `fork` making it faster and consuming
less memory.

## Backticks

Backticks is the right choice if you wanna spawn a command, wait for its execution
and get the whole content of whatever that program printed into a variable.
It usually looks like this.

    my $output = `ls -lha`;

Now `$output` is a single string containing the output of `ls -lha`. But consider
that it only captures what the command printed to its own `STDOUT` channel. By
default it doesn't capture the output of `STDERR`. But usually that is what
you want when using this command.

My best practice is only that you should use `qx` instead.

    my $output = qx(ls -lha);

`qx` is just much easier to read. For the human eye it is much better to distinguish
compared to a normal string with a single tick. Compare ' to `. Hard to
distinguish, right?

One disadvantage of `qx` is that it only allows you to pass a single string
like the first `system` call to it. So when you execute it with variables in it
you end up with the possible problem of quoting characters again.

Another disadvantage is still that the program does not run in *parallel*.
Means the program has to completely run and finish until you get the result
of it as a single string.

Or maybe it is not a disadvantage. Depends on the command you run. If you
need to run a command that completely needs to finish and the program uses its
output to pass information on what it have done, this is the right command
for you.

## Pipes

But usually most commands, especially in a Unix like environment, works in
the way that it immediately prints result and each line is considered a result.

This is the typical piping you often see in bash. For example you could
write.

    find -type f | grep 'ao'

and you immediately see all files that contain an `ao` in it. In this example
`find` and `grep` actually somehow works in parallel. The operating
system usually spawn two processes. `find` prints each file to its STDOUT
and the STDOUT of this programm is pushed into the STDIN of `grep`. `grep`
reads each line from its STDIN.

Here `grep` immediately prints the result of each line and does some filtering.
It doesn't need to wait for `find` to finish or read the whole output into
its own gigantic variable.

In Perl you can achieve the same thing with the [open](https://perldoc.perl.org/functions/open) command.

for example someone could write.

    open my $fh, '-|', "find -type f";

This will spawn the `find` command and gives you a `$fh` file-handle. You then
can read from it by line like you are used to reading from a typical file.

For example this code emulates the same behaviour as the linux command.

```perl
open my $fh, '-|', "find -type f";
while ( my $line = <$fh> ) {
    print $line if $line =~ m/ao/;
}
close $fh;
```

Each line that contains an `ao` is printed and the results are immediately shown,
you don't need to wait for `find` to finish.

`open` supports the two open methods `-|` or `|-`.

When the `-` is on the left-side consider it the way that the output flows
from right to left. On the right-side you finde the command `find -type f` and
its output goes into `$fh`.

When you use `|-` then it like `$fh` is a file-handle that writes into the
right command. This is when you want to spawn a command and give it values.

For example this spawns a `grep` command that filter on `ao`.

```perl
open my $fh, '|-', "grep 'ao'";
print $fh "hello\n";
print $fh "maor\n";
print $fh "world\n";
close $fh;
```

it will only print `maor`. When you have a more complex use case, reading
from sockets, pipes, fork and any other cases you should read [perlipc](https://perldoc.perl.org/perlipc)
and look at modules like [IPC::Open2](https://perldoc.perl.org/IPC::Open2),
[IPC::Open3](https://perldoc.perl.org/IPC::Open2) or [IPC::Run](https://metacpan.org/pod/IPC::Run)

# map

Sometimes perl is critizes by those who don't understand it as being too short.
Like: Hey that one-liner is hard to read, can you not do the same
in 100 lines of code instead? So lets' dissect the `map` line above.

```perl
my @packages = map { m/\A rc \s+ (\S+)/xms ? $1 : () } <$fh>;
```

the full version would be something like this.

```perl
my @packages;
for my $line ( <$fh> ) {
    if ( $line =~ m/\A rc \s+ (\S+)/xms ) {
        push @packages, $1;
    }
}
```

First I don't explain the regex `\A rc \s+ (\S+)`. Regular expressions always
look cryptic to those who don't understand it. Much like chinese. They look cryptic
in all languages, not just Perl.

Usually I would recommand using a `while` loop instead a `for` loop in that case.
The difference? In a `for` loop the whole file-handle `$fh` must be executed
and read from. While a `while` loop works iteratively line by line. But in that
case it doesn't make much of a difference.

Perl has a special underscore variable. It is often used as the default variable.
So in a lot of cases when we omit a variable this default variable is used
by commands. This is also the case in a for loop. When we omit the variable
binding `$line` it just puts its result into `$_`. So we also can write.

```perl
my @packages;
for ( <$fh> ) {
    if ( $_ =~ m/\A rc \s+ (\S+)/xms ) {
        push @packages, $1;
    }
}
```

This is also the case when we do pattern matching with regular expression. When
we don't define on which variable we do regex matching, it uses `$_` by default.
So code can be further shortened too.

```perl
my @packages;
for ( <$fh> ) {
    if ( m/\A rc \s+ (\S+)/xms ) {
        push @packages, $1;
    }
}
```

and technically, whenever you have some kind of loop with the goal to `push`
some values onto on array, you just can use `map` instead.

An important aspect of `map` in Perl is that you also can use it to filter (grep).
Whenever you return *nothing* in a `map` call then nothing is added to the final
array. So we could write.

```perl
my @packages = map {
    if ( m/\A rc \s+ (\S+)/xms ) {
        $1;
    }
    else {
        ();
    }
} <$fh>;
```

Here the empty parenthesis `()` indicates the idea of an *empty list* in Perl.
This can be further reduced by using the *ternary operator*.

```perl
my @packages = map {
    m/\A rc \s+ (\S+)/xms
    ? $1
    : ()
} <$fh>;
```

or one a single line

```perl
my @packages = map { m/\A rc \s+ (\S+)/xms ? $1 : () } <$fh>;
```

## Why not use backticks?

Maybe you ask why I don't use backticks instead of `open`? We anyway process
all the content of `$fh`, so using the `open` approach has no benefits, right?

Well, that's not quite right. First, if i would have used backticks then the
whole content of `dpkg -l` would have read into memory as a single string.
This is *quite large*. Okay *large* is really relative. Calling `dpkg -l | wc -c`
tells me that the output is around 400 KiB on my system. Maybe large for a
string. And could be large if we would be in the MS-DOS age and only had
640 KiB of conventional memory. But in todays age of memory its like nothing.

But still, why read so much memory if it is not needed? We only need to keep
track of the package names that are flagged **rc** in the output of `dpkg -l`. I guess
usually when calling the command there are only like 0 to 10 packages. Maybe 100
when you never purged the configs and you worked for a system for some years. Still
maybe not many.

I could have used **backticks** to read the whole output of the program into
a single string. But then for processing I would actually need to call a `split`
on it to process it by line.

But calling `map { ... } <$fh>` already processes it by line.

Second, Perl also has some optimization. As far as I know Perl itself doesn't
work by creating the full output of `<$fh>` into an array than process it,
it internally uses an iterator to go through it line by line. So from
a memory perspective no more than a single line is read into memory.

Combined with the idea that I don't need to `split` the result it makes it
the far superior version compared to using backticks.

Even if in this small program it will probably never matter what you do,
but maybe you have to write a program where this will matter, now you know
the difference between the different approaches and its advantages and
disadvantages.

Always remember: There is always more than one way to do it. And every way usually
has its advantages and disadvantages.

# Related

* [Searching for a linux command]({{< ref 2023-11-04-searching-for-a-linux-command >}})
