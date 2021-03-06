# Deferred to 6.e

**NOTE: see also [DEPRECATIONS.md](DEPRECATIONS.md) file that contains features deprecated in earlier language versions and possibly need to be removed
in 6.e or later**

This file contains features that are being deferred to 6.e or later language versions. The
primary reason is [R#1289](https://github.com/rakudo/rakudo/issues/1289) and that we're currently
unable to change behaviour of methods based on caller's language. A lot more thought needs to
be given to that Issue and it was decided it's best to release 6.d language with the features
we can implement right now, rather than have that Issue hold everything up.

# POLICIES

## Define More Concrete Spec Errata Rules

* Define specific protocol for how past specs can be changed
* What is a "test that's wrong"?
    * How is that wrongness measured?
    * Who and how many people decide that it's wrong?
    * Which code is considered the test itself and which code is merely
        supporting code that isn't explicitly being tested by the test?
* How do we make users aware of spec errata changes?
* What does the core dev do with code they wrote that turned out to break possibly-wrong tests?
    * Specify specific branch where such commits should be kept in
    * Avoid committing this stuff to `nom` until decision is reached that tests are indeed wrong
* Make it clear to all core devs that this protocol must be adhered to

## Define More Concrete Policies For Deprecations/Changes with Large Impact

* What's our deprecation period for features?
* What's our user notification chain for changes that might impact them?
    * ~~Define where users can watch for critical notifications~~ exists now as alerts.perl6.org
    * Define how much notice we must give users before we introduce potentially
       breaking changes, which aren't necessarily have to do with language, but
       even things like infrastructure changes (e.g.
       [how well was the nom to master rename handled?](https://irclog.perlgeek.de/perl6-dev/2017-10-27#i_15360590)
       should we have given bleed users more notice?)


# FEATURES

## Use IEEE 754-2008 semantics for num/Num infix:</>, infix:<%>, and infix:<%%>

**(Sidenote: be sure to check log(42, 1) does not explode when this is implemented.
If it's decided not to implement this; change log(42, 1) to give a better error**

Relevant Issue: https://github.com/rakudo/rakudo/issues/2502

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
