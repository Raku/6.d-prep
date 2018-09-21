
This document contains planned, but not yet completed, changes for v6.d, and who is responsible for implementing them.

If you're contemplating implementing a new feature, consider implementing it in 6.e language instead.


# IN NEED OF IMPLEMENTATION

## `use v6.d.PREVIEW` in wrong place needs to throw

For example, this code silently fails. It should throw the same style of
error as when, for example, you use `unit`... *"Too late to blah blah..."*

```perl6
    use v6.c; sub foo { use v6.d.PREVIEW; await start say 42 }()
```

# PROPOSED FEATURES

## Sort out normalization of ZDRs

There's a whole bunch of propspec that defines stuff for ZDRs with
different numerators, like [`&[===]`](https://github.com/perl6/roast/commit/fd7c11bfc).

Need to decide whether there's a away to avoid LTAness with normalized ZDRs
or to fix all the bugs with non-normalized ZDRs and ensure all the propspec
concerning this is as it should be.

UPDATE: TimToady OKed making throwage exceptions less precise and then we can 
normalize ZDRs and be happy: http://colabti.org/irclogger/irclogger_log/perl6-dev?date=2018-09-21#l248

```
15:41 	TimToady 	.
15:41 	yoleaux 	09:17Z <Zoffix> TimToady: what's your opinion on changing 6.c 
spec's test that tests for "4" in this exception: `my $x = 4/0; say 42 + $x; => 
Attempt to divide 4 by zero using div`. There are a couple of bugs to fix and 
some performance benefits to reap if we don't report any particular value. We'd 
do so by normalizing all zero-denominator Rationals to <-1/0>, <0/0>, and <1/0>
15:41 	yoleaux 	09:26Z <Zoffix> TimToady: well, bugs could be fixed with 
other means, by basically slowing down the common Rational ops with edge-case 
logic to handle zero-denominator Rationals (ZDRs). Basically by normalizing 
them the check for ZDRs becomes just a denominator check (and I tried marking 
them with a role instead and doing MMD in the past, but dispatch ambiguities 
creep up due to Rational role)
15:41 		p6bannerbot set mode: +v ExtraCrispy
15:41 	TimToady 	I'm fine with normalizing the inv/nan rats
15:41 	Zoffix 	\o/
15:42 	TimToady 	it's not like nums carry that info along either
```


### Stakeholder

Zoffix

### Time Required to Implement

40 hours

-----------------------------------------------------------------

## Remove all proptests for IO::Handle.new's attribute-setting

Continuing from [R#2039](https://github.com/rakudo/rakudo/issues/2039),
we'll likely want to diminish what IO::Handle.new does or even forbid
calling the method entirely.

Right now only `:$path` argument to `.new` is required for operation
of the handle. All other attributes can be set from `.open` call. There
are some proptests in `S32-io/io-handle.t` that test attributes can be
set via `.new`. Those proptests need to be removed (check `S32-io/io-cathandle.t`
for anything related that needs to be removed too).

### Stakeholder

Zoffix Znet

### Time Required to Implement

1 hr

----------------------------------------------------------------------

## Sort out meaning of `.bool-only`/`.count-only` after receiving IterationEnd

See https://github.com/rakudo/rakudo/issues/2075

After Issue has a resolution, this roast work in `range-iterator-improvements`
branch needs to be amended and merged to master https://github.com/perl6/roast/commit/ab6ade455b
as well as commit from `moaaar-iterator-cover` https://github.com/perl6/roast/commit/09f561314fd100c28c5da76bfb7e72f9b15d7d56

### Stakeholder

Zoffix Znet

### Time Required to Implement

7 hr
