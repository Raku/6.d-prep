
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

## Use IEEE 754-2008 semantics for num/Num infix:</>, infix:<%>, and infix:<%%>

**(Sidenote: be sure to check log(42, 1) does not explode when this is implemented.
If it's decided not to implement this; change log(42, 1) to give a better error**

Currently, division and related modulus operations with Nums return Failure if the
divisor is zero. By IEEE rules, those would instead produce a NaN for 0e0/0e0 and
Inf with the sign of the divident. Note: Division and related modulus operations where
at least one operanad is a Num or num coerce both operands to Num

The proposed behaviour has [TimToady's nod of approval](https://irclog.perlgeek.de/perl6-dev/2017-02-08#i_14066067)
but is blocked by [three 6.c-errata tests](https://github.com/perl6/roast/blob/e73bb67f64c26926aa2665e64477d9a084821b48/S03-operators/div.t#L9-L11)

Untested, but the implementation likely just involves removing all of the checks for
0 divisors, as NQP ops already Do The Right Thing for nqp::div_n(). For nqp::mod_n()
more examination is needed, the primary problem being that we don't do IEEE's
remainder() operation with it, so what it's supposed to do in these edge cases is
not declared by IEEE.

```perl6
    multi sub infix:</>(Num:D \a, Num:D \b) {
        nqp::p6box_n(nqp::div_n(nqp::unbox_n(a), nqp::unbox_n(b)))
    }
    multi sub infix:</>(num $a, num $b) returns num {
        nqp::div_n($a, $b)
    }

    multi sub infix:<%>(Num:D \a, Num:D \b) {
        nqp::p6box_n(nqp::mod_n(nqp::unbox_n(a), nqp::unbox_n(b)))
    }
    multi sub infix:<%>(num $a, num $b) returns num {
        nqp::mod_n($a, $b)
    }
```

Similar treatment is needed for `Complex` numerics which currently partially follow IEEE:
don't explode, but always produce a `NaN` instead of producing `+/-Inf` in cases where
it's meant to be produced.


### Stakeholder

Zoffix

### Time Required to Implement

6 hours

-----------------------------------------------------------------

## Remove $/ magicalness from %() and @()

And perhaps $() too; Per https://rt.perl.org/Ticket/Display.html?id=131392

### Stakeholder

Zoffix

### Time Required to Implement

30 hours

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
