---
layout:  post
title:   "Algorithms: Filtering an Array (Benchmarks)"
slug:    algorithms-filtering-an-array-benchmarks
date:    2024-10-12T00:00:00
lastmod: 2024-11-07T00:00:00
tags:    [algorithms,data-structures,array]
description:
---

```perl
#!/usr/bin/env perl
use v5.36;
use open ':std', ':encoding(UTF-8)';
use Sq;
use Benchmark qw(cmpthese);

# It started by bechmarking and looking at different filtering implementation
# especially looking at the performance of a full mutable version that
# mutates an array and removes entries instead of createing a new on. This
# version is `by_splice`. Because it mutates and correct benchmarking behaviour
# must be ensured there are versions that copies the array first to really
# compare the difference of the algorithms. But there are also version without
# copies that shows a more practical version.

# creates array with 10_000 random integer numbers
my $amount  = 10_000;
my $numbers = Array->init($amount, sub($idx) {
    return int (rand(100_000));
});

# evens with perl built-in grep
sub by_grep {
    my $numbers = $numbers->copy;
    my @evens   = grep { $_ % 2 == 0 } @$numbers;
    return;
}

# filtering but c-style
sub by_manual {
    my $numbers = $numbers->copy;
    my @evens;
    for my $num ( @$numbers ) {
        if ( $num % 2 == 0 ) {
            push @evens, $num;
        }
    }
    return;
}

# full mutable version that changes array
sub by_splice {
    my $numbers = $numbers->copy;
    my $idx     = 0;
    while ( $idx < @$numbers ) {
        if ( $numbers->[$idx] % 2 == 1 ) {
            splice @$numbers, $idx, 1;
            next;
        }
        $idx++;
    }
    return;
}

# using lazy sequence
sub by_seq {
    my $numbers = $numbers->copy;
    my $evens =
        Seq
        ->from_array($numbers)
        ->filter(sub($x) { $x % 2 == 0 })
        ->to_array;
    return;
}

###- no copy versions

# like grep but without copies (nc = no copy)
sub by_grep_nc {
    my @evens = grep { $_ % 2 == 0 } @$numbers;
    return;
}

sub by_seq_nc {
    my $evens =
        Seq
        ->from_array($numbers)
        ->filter(sub($x) { $x % 2 == 0 })
        ->to_array;
    return;
}

# this is like the grep version, but uses Sq
sub array_filter {
    my $evens = $numbers->filter(sub($x) { $x % 2 == 0 });
    return;
}

# same as filter but uses string-eval to build up query instead of lambda
sub array_filter_e {
    my $evens = $numbers->filter_e('$_ % 2 == 0');
    return;
}

# this creates an "immutable linked list" from array
my $list = List->from_array($numbers);

sub list_filter {
    my $evens = $list->filter(sub($x) { $x % 2 == 0 });
    return;
}

# same, but with functional-style call
sub list_filter_nm {
    my $evens = List::filter($list, sub($x) { $x % 2 == 0 });
    return;
}

###- first_5_* versions

# only getting first 5 even numbers
sub first_5_manual {
    my $numbers = $numbers->copy;
    my $count   = 0;
    my @evens;
    for my $x ( @$numbers ) {
        if ( $x % 2 == 0 ) {
            push @evens, $x;
            last if ++$count >= 5;
        }
    }
    return;
}

sub first_5_manual_nc {
    my $count = 0;
    my @evens;
    for my $x ( @$numbers ) {
        if ( $x % 2 == 0 ) {
            push @evens, $x;
            last if ++$count >= 5;
        }
    }
    return;
}

# getting first 5 even numbers, but code is still abstract like using grep
sub first_5_seq {
    my $numbers = $numbers->copy;
    my $evens =
        Seq
        ->from_array($numbers)
        ->filter(sub($x) { $x % 2 == 0 })
        ->take(5)
        ->to_array;
    return;
}

# when @numbers changes, then $evens will evaluate to the latest updates
# on trying to fetch data from it.
sub first_5_seq_nc {
    my $evens =
        Seq
        ->from_array($numbers)
        ->filter(sub($x) { $x % 2 == 0 })
        ->take(5)
        ->to_array;
    return;
}

sub first_5_list {
    my $evens =
        $list
        ->filter(sub($x) { $x % 2 == 0})
        ->take(5);
    return;
}

sub first_5_array {
    my $evens =
        Array::filter($numbers, sub($x) { $x % 2 == 0})
        ->take(5);
    return;
}

# doing grep and then just picking first 5
sub first_5_grep {
    my @evens   = grep { $_ % 2 == 0 } @$numbers;
    my @first_5 = @evens[0..4];
    return;
}


printf "Benchmarking versions with array copies.\n";
cmpthese(-1, {
    'seq'            => \&by_seq,
    'splice'         => \&by_splice,
    'manual'         => \&by_manual,
    'grep'           => \&by_grep,
    'first_5_seq'    => \&first_5_seq,
    'first_5_manual' => \&first_5_manual,
});

printf "\nFiltering all with different data-structures. No Array copies.\n";
cmpthese(-1, {
    'list'    => \&list_filter,
    'list_nm' => \&list_filter_nm,
    'seq'     => \&by_seq_nc,
    'array'   => \&array_filter,
    'array_e' => \&array_filter_e,
    'grep'    => \&by_grep_nc,
});

printf "\nGetting only first 5 even numbers.\n";
cmpthese(-1, {
    'first_5_list'      => \&first_5_list,
    'first_5_array'     => \&first_5_array,
    'first_5_grep'      => \&first_5_grep,
    'first_5_seq'       => \&first_5_seq_nc,
    'first_5_manual_nc' => \&first_5_manual_nc,
});

print(
    ($numbers->count == $amount)
    ? "\nok - correct \$numbers count\n"
    : "\nnot ok - \$numbers count should be 10000\n"
);
```

Benchmark results on my machine

    Benchmarking versions with array copies.
                    Rate     seq  splice  manual    grep first_5_seq first_5_manual
    seq             333/s      --     -9%    -72%    -73%        -86%           -86%
    splice          366/s     10%      --    -69%    -71%        -85%           -85%
    manual         1188/s    257%    225%      --     -5%        -50%           -51%
    grep           1255/s    277%    243%      6%      --        -48%           -49%
    first_5_seq    2399/s    620%    556%    102%     91%          --            -2%
    first_5_manual 2438/s    632%    567%    105%     94%          2%             --

    Filtering all with different data-structures. No Array copies.
            Rate    list list_nm     seq   array    grep array_e
    list     282/s      --     -0%    -27%    -59%    -89%    -89%
    list_nm  282/s      0%      --    -27%    -59%    -89%    -89%
    seq      384/s     36%     36%      --    -44%    -85%    -85%
    array    684/s    142%    142%     78%      --    -73%    -73%
    grep    2535/s    798%    798%    559%    270%      --     -1%
    array_e 2560/s    806%    806%    566%    274%      1%      --

    Getting only first 5 even numbers.
                        Rate first_5_list first_5_array first_5_grep first_5_seq first_5_manual_nc
    first_5_list          277/s           --          -61%         -89%       -100%             -100%
    first_5_array         704/s         154%            --         -71%       -100%             -100%
    first_5_grep         2465/s         789%          250%           --        -99%             -100%
    first_5_seq        183794/s       66186%        26016%        7356%          --              -90%
    first_5_manual_nc 1908815/s      688325%       271126%       77332%        939%                --

    ok - correct $numbers count
