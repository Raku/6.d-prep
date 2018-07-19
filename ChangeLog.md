```
##################################################################################
##################################################################################
##################################################################################
###
### NOTE: this is an "internal", core dev only file that is a work in
###       progress. It is likely INCOMPLETE and INACCURATE. To avoid
###       confusion, please do not distribute this file to users as
###       an indication of upcoming language changes.
###       
###       If you are a user reading this, you probably should not modify
###       your code based on this file, until a user copy without this
###       notice is publicized. This will avoid unnecessary or inaccurate changes.
###       
###
##################################################################################
##################################################################################
##################################################################################
```

# Introduction

This document lists changes in Perl 6.d (Diwali) language from Perl 6.c (Christmas)
version. A particular implementation of the language may contain additional
changes; please consult with the changelog for your implementation.

## Additions

These are new features that did not exist in 6.c language. For details about
them, please consult with documentation https://docs.perl6.org/

Items marked with `[6.d]` are protected by version pragma and older behaviours
can be obtained by explicitly using `use v6.c` to request an older language version.
All other changes do not conflict with the 6.c language version and implementations
may choose to make them available even when an earlier language version is requested.

#### New Methods

- `Str.uniparse`: parse one or more Unicode character names into
    the actual characters
- `Str.parse-base`: inverse of `Int.base` operation
- `IO::Handle.slurp`: new name for `.slurp-rest`
- `IO::Path.add`: new name for `.child`; adding non-child paths explicitly allowed
- `IO::Path.sibling`: allows to reference a sibling file or directory

#### New Types

- `IO::CatHandle`: use multiple read-only IO handles as if they were one

#### New Variables

- `$*USAGE`: available inside `MAIN` subs and contains the auto-generated
  USAGE message

#### Miscellaneous

- The `IO::ArgFiles` type is now just an empty subclass of `IO::CatHandle`
- Constraints on constants
    - Constraints are now fully enforced
    - Attempting to parametarized type constraints on constants
        (i.e. using `my Foo constant @int`) now throws `X::ParametricConstant` exception

## Deprecations

These methods are deprecated in 6.d language and will be removed in 6.e.
Implementations may choose to emit deprecation warnings or to offer these
methods for a longer period than 6.e release.

- `IO::Handle.slurp-rest` (use `.slurp` instead)
- `Any.flatmap` (use combination of `.flat` and `.map` methods instead)
- `Cool.path` (use `.IO` instead)
- `Pair.freeze` (use `Pair.new` with decontainerized arguments instead)
- `Str.subst-mutate` (use `Str.subst` with `.=` methodcall assign metaop instead)
- `Rational.norm` (`Rational` types are required to be normalized on creation now)
- `IO::Path.child` (use `.add` instead)

## Removals

- The 6.c specification contained a number of "fudged" tests, which use special
  directive to mark a test as "TODO" (or "SKIP"). Any tests that applied a fudge
  to all implementations (i.e. they did not have at least one working
  implementation of the behaviour) have been deemed void and have been removed
  from 6.d language. All new feature proposals for the language now require
  for at least one language implementation to have that feature implemented
  as a proof-of-viability, before they're accepted to be a part of a released
  language specification.

## MISC

- On subroutine names, the colonpair with key `sym` (e.g. `:sym<foo>`) is now reserved,
  in anticipation of possible future use.
