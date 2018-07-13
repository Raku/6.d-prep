# Deferred to 6.e

This file contains features that are being deferred to 6.e or later language versions. The
primary reason is [R#1289](https://github.com/rakudo/rakudo/issues/1289) and that we're currently
unable to change behaviour of methods based on caller's language. A lot more thought needs to
be given to that Issue and it was decided it's best to release 6.d language with the features
we can implement right now, rather than have that Issue hold everything up.

# FEATURES

## Implement secure way of opening a path

i.e. having a guarantee that `$fh` is open to a file in `/foo/bar/file-I-asked/`
and not `/foo/bar/../../your-secrets.txt`. This idea originally started as
simply a way of ensuring the opened file is guaranteed to be within a certain
directory, but should explore all the security concerns with opening
paths that are [partially] generated from user's input.

----

There was a proposal for `.child-secure`, but it's likely not secure enough.
`.child-secure`'s code was [this](https://github.com/rakudo/rakudo/commit/587cd4f9e53efc63d8694cb7770caed2b3d53410).
Per IO Grant work.
Discussion: https://irclog.perlgeek.de/perl6-dev/2017-04-17#i_14439386
Docs for .child-secure: https://github.com/perl6/specs/blob/master/v6d.pod

#### See if there's a way to make even securer operations:

> open the file first and then use readlink syscall on it

https://irclog.perlgeek.de/perl6/2017-07-21#i_14904782

### Stakeholder

Zoffix

### Time Required to Implement

40 hours

-----------------------------------------------------------------

## Need a way to know caller's language

So we could deprecate methods based on it. See
https://irclog.perlgeek.de/perl6-dev/2017-09-29#i_15233995

Some of the previoud 6.d-fying work was reverted from `nom` in:
https://github.com/rakudo/rakudo/commit/22d3d933b3
https://github.com/rakudo/rakudo/commit/8ed7adf1a6
https://github.com/rakudo/rakudo/commit/68fdeff3b6
https://github.com/rakudo/rakudo/commit/142f772e32
https://github.com/rakudo/rakudo/commit/a65d5f922a
https://github.com/rakudo/rakudo/commit/44d5256cd2

### Stakeholder

???

### Time Required to Implement

???

-----------------------------------------------------------------

## `@`, `%`, (and possibly `&`) Sigils Only Accept DEFINITE Objects

In a declaration, the `@` sigil for example implies a type constraint to
`Positional`. I want the sigil to imply the type constraint `Positional:D`
instead, so that it constrains the variable to defined values.

The same applies to the `%` sigil, and possibly the `&` sigil.

Current behavior:

```perl6
    sub f(@x) { say @x.perl }; f Array;     # Array
```

New behavior:

```perl6
    sub f(@x) { say @x.perl }; f Array;
    # dies with
    # Parameter '@x' requires an instance of type Array, but a type object was passed.  Did you forget a .new?
```

### Rationale

I've never seen anybody write code as if they expected an array or hash
variable to contain a type object, yet the type checker silently allows this,
which usually leads to much worse error messages when using the type object as
if it were an instance.

Since normal variables are typically used with assignment, not binding,
constraining the types of parameters is of higher importance.

### Stakeholder

Moritz Lenz

### Time Required to Implement

???

-----------------------------------------------------------------

## Make all redeclarations fatal

Currently, some redeclarations are fatal, such us trying to declare two `only` subs
of the same name. However, redeclarations of variables aren't fatal and are merely warnings.

        $ perl6 -e 'my $x; my $x; say "hi"'
        Potential difficulties:
            Redeclaration of symbol '$x'
            at -e:1
            ------> my $x; my $x⏏; say "hi"
        hi

        $ perl6 -e 'sub foo {}; sub foo {}; say "hi"'
        ===SORRY!=== Error while compiling -e
        Redeclaration of routine 'foo' (did you mean to declare a multi-sub?)
        at -e:1
        ------> sub foo {}; sub foo {}⏏; say "hi"
            expecting any of:
                horizontal whitespace

I propose ALL redeclarations to be made fatal errors, as I can't think of when
code that warns would be more desirable than one that dies and warnings can
be hard to spot sometimes with large-output programs,
[like test suites](https://github.com/perl6/roast/commit/7b592f5e1203cf20525b3ff515f53662e1ea9693)

### Stakeholder

Zoffix

### Time Required to Implement

10 hours

-----------------------------------------------------------------

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

## De-magicalize '-' in IO::Handle.open

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

## Make method matcher forms not set `$/`

For rationalle and discussion see: https://github.com/rakudo/rakudo/issues/1235

This change depends on how the discussion in that Issue concludes.

As part of this work, we'd need to figure out how `$/` is set for
`s///`/`S///` ops, since currently they use method forms under the hood.
One of the bugs pointed out with method forms was `.subst` with `:g` not
setting `$/` to an empty `List` when no matches occured. This would need
to be fixed for the `S///` operator. (Original issue
[R#1523](https://github.com/rakudo/rakudo/issues/1523#issuecomment-365447388))

### Stakeholder

Zoffix Znet

### Time Required to Implement

1 day

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

## Remove Pair.freeze

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

-----------------------------------------------------------------

## Decoding/Encoding windows-1252, windows-1251 and Latin1 throw on invalid input/output

Currently, these three encodings will decode and encode invalid input. This
allows us to create files in these encodings that cannot be opened by
practically any other programming language.

Proposal will make these encodings throw on invalid input/output by default,
although you will be able to optionally be "permissive" and allow invalid
decoding/encoding as long as it is technically feasible. (i.e. even though
codepoint 129 doesn't exist in windows-1252, it *does* fit in one byte, so can
be encoded, despite it being invalid windows-1252).

### Rationale

Python, Perl 5, iconv will all not decode files with unmapped codepoints.
This is a horrible default and goes against the way our other encoders/decoders
work. Given that this is a big change and implementing it before 6.d would
cause previously working script to all of a sudden break. They may have been
utilizing this "permissive" decoding/encoding either intentionally or
unintentionally.

This will be added in 6.d and it will be optional in 6.c

### Stakeholder

Samantha McVey (samcv)

### Time Required to Implement

???