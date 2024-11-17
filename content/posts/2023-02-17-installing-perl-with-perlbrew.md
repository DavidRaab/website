---
layout: post
title: "Installing perl with perlbrew"
slug: installing-perl-with-perlbrew
date: 2023-02-17
lastmod: 2024-06-22
tags: [perl,linux]
description: Howto manage multiple Perl version in home directory with perlbrew.
---

Today every Linux system mostly already ships Perl including a lot of modules.
While this can be totally fine sometimes maybe you want to switch to a newer
Perl version. Or maybe even an older version to test a module
with an older Perl version for compatibility?

[`perlbrew`](https://metacpan.org/pod/App::perlbrew) is a tool that helps
installing multiple Perl environments and gives you the ability to switch between
them. Installing your own Perl also means you don't mess with the system
installed Perl and you are able to install any new module directly from CPAN.
The Perl environments are managed in your `$HOME` directory.

First you need to install `perlbrew` itself. The best option is to install
`perlbrew` from your system ressources if available. In Debian Bullseye you
just can type

```bash
apt install perlbrew
```

It also installs dependency like `gcc`, `make`, `patch` and `libc-dev` that
you need to compile your own Perl version. Maybe your favorite Linux distribution
has a package too. If not you should at least install the same dependencies
and maybe install [App::perlbrew](https://metacpan.org/pod/App::perlbrew) with
the help of [local::lib](https://metacpan.org/pod/local::lib).

Once installed as a user (not root) you now can type.

```bash
perlbrew init
```

the command tells you to add a line to `.profile` (if you are using bash).

```
# Add perlbrew environment
source ~/perl5/perlbrew/etc/bashrc
```

You must add the line and then either restart the terminal or type the `source`
command once.

To install the newest Perl version (at the point of writting this article) i did
the following.

```bash
perlbrew install-patchperl
perlbrew install perl-5.36.0 --thread -j 10
```

The `-j 10` tells how many CPU cores should be used. This installs the new
Perl version but don't use it as the default. You can now switch between Perl
version with the `perlbrew` command.

You can see all installed Perl version with

```bash
user@machine:~$ perlbrew list
  perl-5.36.0
```

You can permanently switch to the new Perl version by executing.

```bash
user@machine:~$ perlbrew switch perl-5.36.0
user@machine:~$ perlbrew list
* perl-5.36.0
```

`perlbrew list` will mark the active Perl with a `*`. You also can check with
`perl --version` if the correct version is being used.

You also can switch temporary just for the terminal session with

```bash
perlbrew use perl-5.36.0
```

Once the newest Perl version is installed i recommand to install
[`App::cpanminus`](https://metacpan.org/pod/App::cpanminus).

```bash
cpan App::cpanminus
```

Its up to you to install additional libraries as you need/want, here is a
complete set of useful ones I installed (on Debian). First I usually install some
system dependencies for C/C++ compiler and some headers for some libraries.

```bash
apt install build-essential  # C and C++ compiler
apt install libev4 libev-dev # AnyEvent
apt install libgd-dev        # GD
apt install libxml2-dev      # XML::LibXML
apt inszall libcairo2-dev    # Cairo / Chart::Clicker
```

Then here some modules you maybe wanna try out.

```bash
cpanm EV
cpanm AnyEvent
  cpanm Try::Tiny # if not explicitly installed Perl::LanguageServer will fail
  cpanm Perl::LanguageServer
cpanm Catalyst
cpanm Data::Printer
cpanm DBI
cpanm DBD::SQLite
cpanm SQL::Abstract
cpanm DBIx::Class
cpanm Text::Table
cpanm Term::ProgressBar
cpanm Moo
cpanm Moose
cpanm MooseX::StrictConstructor
cpanm Path::Tiny
cpanm Type::Tiny
cpanm Type::Tiny::XS
cpanm JSON::XS
cpanm Class::Accessor
cpanm Class::Accessor::Fast
  cpanm File::Copy::Recursive # if not explicitly installed; DateTime will fail
  cpanm Class::Tiny           # if not explicitly installed; DateTime will fail
  cpanm DateTime
cpanm Devel::NYTProf
cpanm Perl::Critic
cpanm Perl::Tidy
cpanm namespace::autoclean
cpanm LWP::UserAgent
cpanm WWW::Mechanize
cpanm Template
cpanm List::MoreUtils
cpanm List::MoreUtils::XS
cpanm Chart::Clicker
cpanm Curses
cpanm File::Slurp
cpanm IPC::Run
cpanm Devel::Symdump   # PerlNavigator
cpanm Sub::Util        # PerlNavigator
cpanm App::perlimports # PerlNavigator
cpanm PPI              # PerlNavigator
cpanm Class::Inspector # PerlNavigator
cpanm Module::Starter
```

Don't forget to use the following shebang line in your scripts.

```
#!/usr/bin/env perl
```

instead of directly picking a path to a specific perl version, or otherwise
your scripts will not use your newly installed Perl.

Have fun coding!