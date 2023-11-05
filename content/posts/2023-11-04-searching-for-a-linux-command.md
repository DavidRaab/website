---
layout: post
title: "Searching for a Linux Command"
slug: searching-for-a-linux-command
date: 2023-11-04
tags: [perl,linux,tool]
description:
---

Lately I wanted to search for a Linux Command.

I didn't knew the exact command, only that there must be a "wacom" in it. A
configuration tool to set some more details of my wacom tablet. So I thought.

Hey, just collect all binary files in all the directores in the PATH Environment
of Linux. The Path environment is divided with colons. Just let me give it a
Regex it can match against.

During development I realized that there can be multiple overlapping binaries
in different folders with the same name. Usually that makes sense as only the
first binary that is found is executed. This way you can control with PATH
which command you wanna override.

Here is my solution.

```perl
#!/usr/bin/env perl
use v5.36;
use open ':std', ':encoding(UTF-8)';
use Data::Printer;
use Getopt::Long::Descriptive;
use List::Util qw/uniqstr/;
use Path::Tiny;

my ($opt, $usage) = describe_options(
    'Usage: search_command REGEX [--full]',
    ['full|f', 'Print Full Paths',   {default => 0}],
    ['help|h', 'Print this message', {shortcircuit => 1}],
);

print($usage->text) && exit if $opt->help;
print($usage->text) && exit if not defined $ARGV[0];

# Precompile regex
# - This way program aborts when regex is mailformed instead of continuing
# - and running the program
my $search = qr/$ARGV[0]/i;

# Get all binaries in PATH that match passed Regex
my @binaries =
    sort { $a->[1] cmp $b->[1]                  } # Sort by basename
    map  { [$_, $_->basename]                   } # Schwartzian Transformation
    grep { $_->is_file && $_->stat->mode & 0111 } # only files and executables
    map  { path($_)->children($search)          } # get children of every PATH that match $search regex
        split /:/, $ENV{PATH};                    # split PATH on :

# prints full-path, usually useful when command exists multiple times
if ( $opt->full ) {
    say $_->[0] for @binaries;
}
# by default - only print basename of binaries (single-time)
else {
    say for uniqstr map { $_->[1] } @binaries;
}

=pod

=head1 search_command

Searches for a command in your PATH Environment by giving it a REGEX. Its useful
when you don't know the exact command but parts of it. REGEX is case-insensitive
by default.

=head1 EXAMPLES

=begin text

    $ search_command wayland
      Xwayland
      es2gears_wayland
      es2gears_wayland.x86_64-linux-gnu
      wayland-scanner

    $ search_command wacom
      xsetwacom

    $ search_command wacom --full
      /usr/bin/xsetwacom
      /bin/xsetwacom

=end text
```
