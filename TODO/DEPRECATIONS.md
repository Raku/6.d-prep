This document contains (a possibly incomplete) list of deprecations of language
features, for which Rakudo implementation cannot currently issue deprecation
warnings. We'll still deprecate these feature on language level and convey those
deprecations through documentation and will possibly have a longer deprecation
period for these features than for those for which we *can* emit deprecation
warnings.

----------------------------------------------

## Deprecate `.parse-names`

It was renamed to `.uniparse`.

Altough `.parse-names` was never part of `6.c`, there's some use of it
in ecosystem and likely elsewhere, so it should undergo a deprecation period.

The original name was chosen to align with `.parse-base` that parses out base-X numbers out
strings. However, we have a whole block of more closely related routines all named in `.uni*`
format, so it makes sense for this routine's naming to align with those. They are:
`infix:<unicmp>`, `unimatch`, `uniname`, `uninames`, `unival`, `univals`,
`uniprop`, `uniprop-bool`, `uniprop-int`, `uniprop-str`, `uniprops`.

### Stakeholder

Zoffix

### Time Required to Implement

3 hours

-----------------------------------------------------------------

## Deprecate .flatmap

Make use of `.flatmap` to issue a deprecation warning in 6.d, to be removed in 6.e. The method
saves a single character of typing, but hides the behaviour (is the .flat done first or is the .map
done first), which can easily lead to a trap that the user isn't getting anything flattened.

See also: https://rt.perl.org/Ticket/Display.html?id=130520
See also: https://github.com/perl6/doc/issues/1428

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

-----------------------------------------------------------------

## De-magicalize '-' in IO::Handle.open

**N.B.: we can't actually change the behaviour in Rakudo ATM (and without
a POV we can't spec it), but we should still document it as a deprecated
functionality IMO**

Currently, `'-'.IO` will operate on a filesystem object (e.g. `'-'.IO.mkdir`
will create a directory named `-`). However, trying to C<.open> such a path
causes special-cased behaviour in `IO::Handle.open` where it return `$*OUT`
if the open was done for reading or `$*IN`, if the open was done for write.

This is a holdover from Perl 5 and is a lot less useful in Perl 6 (since you can
just use `$*IN/$*OUT` directly or create your own IO::Handle with IO::Special as paths).
While the special-cased behaviour can be unwanted and surprising.

The proposal is to remove this feature in 6.d, so that `'-'.IO.spurt: 'foo'` would
spurt into a file named `-` rather than to `$*IN` handle.

Relevant discussion: https://irclog.perlgeek.de/perl6-dev/2017-06-23#i_14777024

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

-----------------------------------------------------------------

## Deprecate Cool.path

We already have `Cool.IO`. Also, `IO::Path.path` exists and `IO::Path`
is `Cool`, yet that `.path` returns a `Str` object, not `IO::Path`.

Issue deprecation warning in 6.d and remove when 6.e comes out.

This isn't actually part of 6.c language, but when I
[removed it](https://github.com/rakudo/rakudo/commit/b212fc5e20), `zef` crashed.

May as well go through full deprecation cycle and keep it around, just in case.

### Stakeholder

Zoffix Znet

### Time Required to Implement

1 hour

-----------------------------------------------------------------

## Deprecate Pair.freeze

*(Added by Zoffix, based on commit
[c229022cb0](https://github.com/rakudo/rakudo/commit/c229022cb09413e48b7e3d8343a823463f48cb71)'s
message)*

My (Zoffix) :+1: for its removal is because it doesn't actually "freeze" much:

```perl6
   with foo => [<a b c>] { .freeze; .value = 100; .say } # OUTPUT: foo => [100]
```

So it's a method that overpromises on what it does and merely deconts the value—something
that likely should've been done during the Pair's creation.

### Stakeholder

lizmat

### Time Required to Implement

???

-----------------------------------------------------------------

## Deprecate Str.subst-mutate

This routine is misdesigned and unneeded:

1. This method does not fit the language. We don't offer `-mutate` versions of any
    other routines and there's a really good reason: the `.=` methodop and infixop
    provide that feature for all the methods automatically.
2. The current implementation is broken in that it doesn't set `$/` if `Str` matcher
    was given, yet it returns `Match` objects in all cases.
3. Worst of all this routine has poor performance, detailed in
   [R#1668](https://github.com/rakudo/rakudo/issues/1668). The current unoptimized version
   is 44x slower than `.=subst` variant, yet even after `Str, Str` candidate was added,
   the performance was still 3x slower, due to the need of fetching caller's `$/` in one
   extra level. I'm speculating it'll never match the speed of `.=subst` because `.=subst`
   can do the assignment with a local-scoped variable, while `.subst-mutate` requires a mutable
   container actually gets passed to it.

In summation, it's an outlier as far as language's design is concerned, it still has bugs even
after we fixed a ton of bugs with it already, and it has dismal performance resulting in
us having to actively discourage its use. I call we deprecate it in 6.d and remove entirely in 6.e.

### Stakeholder

Zoffix

### Time Required to Implement

5 hours

-----------------------------------------------------------------

## Deprecate Rational.norm

There's absolutely no need for this method to exist as (after
[CaR Grant](http://news.perlfoundation.org/2018/04/grant-proposal-perl-6-bugfixin.html)) all core Rationals
are always normalized upon creation.

It's actually required to perform normalization on creation to be able to maintain
Rationals as immutable, while still having valid object identity tests (e.g. `<4/2> === <2/1> # => True`)

### Stakeholder

Zoffix

### Time Required to Implement

5 hours
