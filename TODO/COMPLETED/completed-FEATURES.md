
This document contains already implemented planned changes for v6.d.
See [FEATURES.md](../FEATURES.md) for features yet to be implemented.

# Implemented


# PROPOSED FEATURES

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

-----------------------------------------------------------------

## `use v6.d.PREVIEW` in wrong place needs to throw

For example, this code silently fails. It should throw the same style of
error as when, for example, you use `unit`... *"Too late to blah blah..."*

```perl6
    use v6.c; sub foo { use v6.d.PREVIEW; await start say 42 }()
```

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

## Make `start` blocks in sink context attach an error handler

At the moment, any exceptions leaking out will be lost. For this particular case, determined
through semantic analysis at compile time, a `then` should be attached that will `die` with
any error that occurs during the execution of the `start` block and goes unhandled by it, so
the errors will not be lost. Such an error will trigger `uncaught_handler` in the thread pool,
which by default will bring down the program. Note this is *not* a general sink operation on a
`Promise`; it applies only to the `start` syntax.

### Rationale

It's rare that people really want to ignore that some background worker they set off failed.
Most `start` blocks are `await`ed or `then`'d because people care that the work completes.
The case for an unguraded `start` is most often setting off some application-lifetime worker.
In that case, the behavior being that it crashing brings the application down is the right
thing anyway, and desirable given the alternative would be that the application silently stops
responding, if it was a `start react` that was meant to handle events over the application's
lifetime.

### Stakeholder

Jonathan Worthington

### Time Required to Implement

1 day

-----------------------------------------------------------------

## Make default defaults for DefiniteHOWs be normal types

Basically, so you could do `my Foo:D $x .= new;`, which
currently dies since you can't instantiate DefiniteHOWs.

There's an open Rakudo issue for this that also contains
a potential non-version impl in a branch:
https://github.com/rakudo/rakudo/issues/1493

### Stakeholder

Zoffix Znet

### Time Required to Implement

2 hr

-----------------------------------------------------------------

## Remove $/ magicalness from %() and @()

And perhaps $() too; Per https://rt.perl.org/Ticket/Display.html?id=131392
and https://github.com/rakudo/rakudo/issues/1946

If deferring until later language versions, fix up
https://github.com/perl6/roast/commit/57e5a7d463 and
https://github.com/perl6/roast/commit/7b568420dc


### Stakeholder

Zoffix

### Time Required to Implement

30 hours

-----------------------------------------------------------------

## Deprecate `undefine`

Per discussion in https://github.com/perl6/6.d-prep/issues/5

### Stakeholder

Zoffix

### Time Required to Implement

5 hours

----------------------------------------------------------------------

## Change default for `num` to be `0e0`

Per discussion in https://github.com/perl6/6.d-prep/issues/4

There's a also a ton of propspec in S02-types/num.t for various ops expecting
default to be a `NaN` that will need to be changed as well.

### Stakeholder

Zoffix

### Time Required to Implement

10 hours

----------------------------------------------------------------------

## Make `$*ARGFILES` := `$*IN` or `IO::ArgFiles.new($*IN)` inside MAIN

It being based on `@*ARGS` is virtually never useful when you handle any sort of non-file arguments (which is what using MAIN implies).
To allow lines(), get(), words(), and slurp() to Do The Right Thing when called and use `$*IN`, we should swap `$*ARGFILES` to `$*IN`
(or to `IO::ArgFiles.new($*IN)`, to maintain `$*ARGFILES` type consistency),
when we're inside `sub MAIN(){}` and only maintain the current behaviour of considering `@*ARGS` when outside of MAIN.

Relevant discussions:

* https://rt.perl.org/Ticket/Display.html?id=131703#txn-1471796
* https://irclog.perlgeek.de/perl6/2017-07-05#i_14827209
* https://irclog.perlgeek.de/perl6-dev/2017-07-05#i_14827981
* https://github.com/rakudo/rakudo/issues/1946

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

----------------------------------------------------------------------

## Rename `RAKUDO_EXCEPTIONS_HANDLER` to `PERL6_EXCEPTIONS_HANDLER`

There are actually erroneous proptests in `S04-exceptions/exceptions-json.t`
that set `RAKUDO_EXCEPTIONS_HANDLER` variable.

The var should be renamed (with some deprecation support for old name for some time)
to a variant without a specific implementation name in it.

The docs should also be amended to move this feature out from implementaion-specific
features.

### Stakeholder

Zoffix Znet

### Time Required to Implement

3 hr

----------------------------------------------------------------------

## Properly reserve all `:sym<>` colonpairs on subroutines

This is mostly a reminder so we don't forget. As
[previously discussed](https://irclog.perlgeek.de/perl6/2017-01-25#i_13988093),
We want to make `:sym<>` colonpairs on subroutines reserved. However,
there is [a 6.c-errata test](https://github.com/perl6/roast/blob/dfe905a8ce84d09b5b0536fcc151e798488a9289/S32-exceptions/misc.t#L132)
that expects `sub foo:sym<bar> {}` `X::Syntax::Extension::Category`
exception instead of `X::Syntax::Reserved` we desire. The
[code implementing this](https://github.com/rakudo/rakudo/commit/48abeeef26)
already exists. It just needs to be uncommented for 6.d and
[corresponding tests](https://github.com/perl6/roast/commit/53d6e8491d)
unfudged.

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

-----------------------------------------------------------------


## Spec IO::Handle's `.write-internal`, `.read-internal`, `.eof-internal`

The method names should be changed as currently they imply Rakudo-internal methods
rather than methods users should be overriding to affect all read/write methods.

They should probably use different casing to differentiate them from regular IO::Handle
methods consumers of IO::Handle would use. Perhaps, `.WRITE`, `.READ`,
and `.EOF`, but would it be confusing with the lower-case methods of the same name?

Spec and document the methods as the proper way to create custom IO::Handle
subclasses.

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

-----------------------------------------------------------------

## Non-blocking await and react

In v6.c, using `await` currently blocks a real thread. In v6.d it will, provided the `await`
takes place in code running on the thread pool, take a continuation and scheduler its
resumption after the awaited operation completes.

A react block also becomes non-blocking in the same way (so `start react ...` does not block
a thread pool thread in v6.d also).

### Rationale

It is quite easy to write code that waits for multiple promises in parallel,
which can lead to the full thread pool begin tied up in blocking waits,
leading to a deadlock. Examples of this happening have showed up in the past. Even if
that doesn't happen, it still can increase the number of threads needed, thus making
for higher memory use.

### Stakeholder

Jonathan Worthington

### Time Required to Implement

It's implemented, though we should do a spectest run and ecosystem toast with 6.d as
the default to check there's no surprises.

## Consistify Naming of `$*INITTIME`

All of our other multi-word dynvars use kebobcase, so `$*INITTIME`
should be re-named ~~`$*INIT-TIME`~~
[`$*INIT-INSTANT`](https://github.com/perl6/roast/issues/296#issuecomment-324719710)

There is some mention of it on 5to6 docs and IRC logs, so there
should be some deprecation period for the old name.

See also:
- https://github.com/perl6/roast/issues/296
- https://github.com/perl6/doc/issues/1462
- https://github.com/perl6/doc/issues/510
- https://github.com/perl6/roast/issues/296#issuecomment-324719710


## Remove dummy precision parameters from Rational/Int .Rat and .FatRat coercers

**Since we can't modify methods between languages, the params were deprecated instead,
for removal in 6.e**

They're dummy parameters that offer more confusion than usefulness. The roast itself
seems confused. There are a whole bunch of trig tests that use these coercers with a
precision arg for no good reason; almost feels like the writer assumed `1.5` is a Num and
not a Rat.

Done in: https://github.com/rakudo/rakudo/commit/4c337e8ef9


## Deprecate IO::Path.chdir and IO::Handle.slurp-rest

Per IO Grant work.

* `IO::Path.chdir` - the name is a poor choice, as it doesn't **ch** hange
    any **dir** ectories. Functionality with relative-path argument is directly
    replaced by `IO::Path.add` method and functionality with absolute-path is
    directly replaced by `IO::Path.new` or `Cool.IO` methods.

* `IO::Handle.slurp-rest` - removed to make interface consistent with
    the rest of the language that uses `slurp` as the name for the routine.
    The `IO::Handle.slurp` was already implemented as part of the IO Grant.

Done in: https://github.com/rakudo/rakudo/commit/6d2adb20f2
     and https://github.com/rakudo/rakudo/commit/3341384bfe


## Remove deprecated Test::is_approx

This is mostly a reminder so we don't forget. Test::is_approx is currently marked as deprecated
(replaced by `is-approx`). It was decided to leave it in until 6.d, so that we can still use it
in 6.c-errata without any changes.

###

Made to die in 6.d in https://github.com/rakudo/rakudo/commit/cd043f2ae4

## Remove Str.lines :$count parameter

Marked as deprecated instead of complete removal in
https://github.com/rakudo/rakudo/commit/01d4939c38

An entirely pointless, undocumented, unused, and unloved parameter whose removal is blocked by
[two 6.c tests](https://github.com/perl6/roast/blob/5c703c355d0457f78f1f32a0f3af394ab54be256/S32-str/lines.t#L59-L60)


## Remove `$*MAIN-ALLOW-NAMED-ANYWHERE`

It was never specced and currently exists for backwards compatibility with older panda. By 6.d it should not matter and could be
removed.

Removed in https://github.com/rakudo/rakudo/commit/9cb4b167f5

## Formal Rules for Defining Matched Delimiters/Brackets

It was decided that due to Ornate Parenthesis being unused and the test in actuality being *wrong* (and it was an accident it was codified that way) we would
be able to implement this before v6.d.

### Status

Resolved

#### Fri Oct 6 2017

[NQP 02a426e0e](https://github.com/perl6/nqp/commit/02a426e0e)
* Fri Oct 6 2017, Support for using ornate parenthesis for quoting constructs is removed

[NQP 576d78eef](https://github.com/perl6/nqp/commit/576d78eef)
* A warning message is added, notifying the user:
```
Ornate parentheses U+FD3E '﴾' + U+FD3F '﴿', have been removed
Please use another type of bracket.
See https://github.com/perl6/nqp/commit/02a426e0e for removal reasons.
```

### Thu Jan 4 2018

[NQP 277cfcb2d](https://github.com/perl6/nqp/commit/277cfcb2d)
* The warning message was removed as planned.


## Formal Rules for Defining Matched Delimiters/Brackets

In v6.d we should formalize which brackets we support.
How do we decide which delimiters should be added on future updates
to the Unicode standard? We should look to the Unicode standard to help
us define matching delimiters and brackets for Perl 6.

All delimiters we support should conform to two simple rules for the sake of
uniformity, elegance and clairity.

### Rules

1. Delimiter's Unicode General_Category must match one of these:

    Pi -> Pf ( Punctuation, initial quote -> Punctuation, final quote)
    Ps -> Pe (Punctuation, start -> Punctuation, end)

2. The delimiters must be matching BidiBrackets and/or BidiMirroring characters.

Bidirectional brackets are specified
[here](http://www.unicode.org/Public/UCD/latest/ucd/BidiBrackets.txt)

Non brackets have their matching glyph specified in this
[file](http://www.unicode.org/Public/UCD/latest/ucd/BidiMirroring.txt)

### Possible issues

The only possible issue, is what to do with ornate parens.

**BidiBrackets.txt** states:

“For legacy reasons, the characters U+FD3E ORNATE LEFT PARENTHESIS and
U+FD3F ORNATE RIGHT PARENTHESIS do not mirror in bidirectional display
and therefore **do not form a bracket pair.**”

In v6.c, roast includes tests for 'ornate left parens' and 'ornate right parens'
for doing things like `q[ ]` type contructs and such. I think that we should not
allow these parenthesis because firstly, Unicode states they do not form a matching pair
of brackets. Secondly, the ornate parenthesis also do not have mirror glyphs.
To make matters even worse, their Unicode general categories are the opposite of every
matched bracket we support, the opening brackets tested for in v6.c open with
"Pe"(End) and close with is "Ps"(Start).
They break both of these proposed rules.

In practice this is already implement with the exception of the ornate parenthesis,
but I propose this be made an official part of the Perl 6 standard.

### Stakeholder

Samantha McVey (samcv)
