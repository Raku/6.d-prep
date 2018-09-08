
This document contains planned, but not yet completed, changes for v6.d, and who is responsible for implementing them.

If you're contemplating implementing a new feature, consider implementing it in 6.e language instead, which will be released at the end of 2019.


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

### Stakeholder

Zoffix

### Time Required to Implement

40 hours

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
