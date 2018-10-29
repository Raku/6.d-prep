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

At the same time, a particular implementation may have had certain features
already implemented during 6.c version period. This ChangeLog concerns itself
with new features added to the specification on a language level and not
the status of their implementation in a particular compiler.

# Scope / Target Audience

This ChangeLog is targeted towards language users, to help with preparation
to use compilers supporting latest language version. Thus, it does not
contain every minute change to the specification that occurred.
Implementations wishing to ensure full compliance with the new version of the
language specification should execute the test suite and examine any failing
tests.

## Additions

These are new features that did not exist in 6.c language. For details about
them, please consult with documentation https://docs.perl6.org/

Items marked with `[6.d]` are protected by version pragma and older behaviours
can be obtained by explicitly using `use v6.c` to request an older language version.
All other changes do not conflict with the 6.c language version and implementations
may choose to make them available even when an earlier language version is requested.

#### New Behaviors

- Major clarifications and redesign of all set operators
- Loops can now produce a list of values from the values of last statements
- `next`/`last`/`redo` in a loop that collects its last statement values
    return `Empty` for the iterations they run on
- `.perl` can now be called on consumed `Seq`s, multi-dimensional arrays,
    `Date`, and `CallFrame`
- `.gist` can now be called on `Attribute`
- Numerous improvements to auto-generated `USAGE` message
- `Parameter.perl` now includes introspectable defaults
- `%*ENV` values are now allomorphic
- Trying to use variables `$;`, `$,`, `$.`, `$%`, `$\`, `$#`, `$(`, `$)`,
    `$<`, `$>`, `$/`, `$\`, `$[`, `$-`, `$+`, and `$@`
    now throws `X::Syntax::Perl5Var`
- Default `Hash.of` now returns a `Str(Any)` coercer type object
- non-ASCII numerics can be used in `:42foo` colonpair shortcut
- `StrDistance` now stringifies to its `.after` string
- More well-defined formatting of Pod tables
- `Enumeration.enums` now returns a `Map`
- `.Range` on various integer types now returns the range of values they support
- `min`/`max` routines are now work on `Hash`es
- `Signature` literals can now contain string/numeric literals as
    well as the invocant marker
- `List.invert` now maps via a required `Pair` binding, resulting in
    potential type check failures
- `:exists` now can be used with multi-dimensional associative subscripts
- Dynamically created lists can be used to define an enum
- Junctions can now be used as a matcher in `.first`
- Native attributes can now be used as bind targets in parameters
- `Proc` can now work with `IO::Pipe`s from other `Proc`s
- Typed arrays can be created with both `my SomeType @array`
    and `my @array of SomeType`
- Items with negative weights are now removed when coercing
    a `Mixy` to `Setty`/`Baggy`
- `:nth` adverb on `m//` now accepts a `Junction` as argument
- `CX::Warn` and `CX::Done` can now be caught inside `CONTROL` phaser

#### Math

- [6.d] Native `num` types now default to `0e0` instead of `NaN`
- `-Inf`, `Inf`, and `NaN` can now be round-tripped through `Rat` type
    by being represented as values `<-1/0>`, `<1/0>`, and `<0/0`> respectively.
    Zero-denominator `Rational`s are now normalized to one of those three values
- Calling `.Int` on ±`Inf` and `NaN` now throws
- Improved IEEE 754-2008 compliance in `Num` operators and math functions
- Stringification of `Num` type is now required to be roundtrippable
    to the original `Num`

#### New Parameters
- `Date.new` now accepts a `:&formatter`
- `.first` can now take `:$kv`
- `unique` and `.repeated` can now take `:&as` and `:&with`

#### New Routines

- `submethod TWEAK`: similar to `BUILD` except runs after defaults has been set
- `&duckmap`: apply `&callable` on each element that behaves in such a way
    that `&callable` can be applied
- `&take-rw` like `&take` but with a writable container
- `.hyper`/`.race`: process a list of values in parallel
- `Seq.from-loop`: generate a `Seq` from a `Callable`
- `Str.uniparse`: parse one or more Unicode character names into
    the actual characters
- `Str.parse-base`: inverse of `Int.base` operation
- `IO::Handle.slurp`: new name for `.slurp-rest`
- `IO::Path.add`: new name for `.child`; adding non-child paths explicitly allowed
- `IO::Path.sibling`: allows to reference a sibling file or directory
- `IO::Path.mode`: retrieve file mode
- `fails-like` in Test.pm6 module: allows testing for Failures
- `bail-out` in Test.pm6 module: exit out of failing test suite
- `is-approx` in Test.pm6 module: test a number is approximately like another
- `Buf` now has `.allocate`, `.reallocate`, `.append`, `.push`, `.pop`,
    `.splice`, `.prepend`, and `.unshift` methods
- `Range` now supports `.rand`
- `.antipairs` method can be used on `QuantHash` types
- `Backtrace` now has methods `.map`, `.flat`, `.concise`, and `.summary`
- `.invert` method is now available on core `QuantHash` types
- `.classify-list` method is now available on `Hash` and `Baggy` types
- `.categorize-list` method is now available on `Hash` and `Baggy` types
- `Code.of`: returns the return type constraint
- `Code.line`/`.file`: returns the line/file of definition
- XXX TODO: https://github.com/rakudo/rakudo/issues/2444
- `Proc.command`/`Proc::Async.command`: the command we're executing
- `Proc.signal`: the signal the process was killed with
- `Complex` type now provides `.reals`, `.ceiling`, `.floor`, `.round`,
    `.truncate`, and `.abs` methods and can be compared with `<=>` (as
    long as the imaginary part is negligible)
- `DateTime` type now provides `.offset-in-hours` and `.Date` and can be
    compared with other `DateTime` objects using `<=>` operator
- `Date` now provides `.DateTime`
- `&infix:<+>`/`&infix:<->` can now be using with `Duration` and `Real` types
- `Enumeration` now provides `.kv`, `.pair`, and `.Int` methods
- `.Date` can now be called on an `Instant`
- Junctions can now be created using `Junction.new` call
- `List` type now has `.to` and `.from` methods
- `Map` type now provides `Int` method, returning the number of pairs
- `Map` now provides `.clone`
- `QuantHash` types now have `.new-from-pairs` and methods to convert one
    `QuantHash` type to another (e.g. `.Bag` method on `Set` type)

#### New Types

- `IO::CatHandle`: use multiple read-only IO handles as if they were one
- Native `str` arrays
- `Semaphore`: control access to shared resources by multiple processes
- `UInt` a subset of `Int` containing non-negative numbers
- `Exceptions::JSON` an implementation of custom exceptions handler
    (can be used with `PERL6_EXCEPTIONS_HANDLER` env var)

#### New Variables

- `$*USAGE`: available inside `MAIN` subs and contains the auto-generated
  USAGE message
- `PERL6_TEST_DIE_ON_FAIL` environmental variable: stop test
    suite on first failure

#### Clarifications of Edge Case/Coersion Behaviour

- Defined behaviour of `permutations`/`combinations` on 1- and 0-item
    lists and negative and non-Int arguments
- `val`, `Str.Numeric`, and other `Str` numeric conversion methods
    throw when trying to convert Unicode `No` character group
- Numeric conversion of `Str` containing nothing but whitespace returns `0` now
- `samemark` with empty pattern argument simply returns the invocant
- `.polymod` can now be used with `lazy` but finite lists of divisors
- `.[*-0]` index is now defined
- Negative gaps in `.rotor` that are larger than the sublist now throw
- Non-`Int` arguments to `.rotor` now get coerced to `Int`
- `.hash` on `QuantHash` types does not stringify keys
- `.lines` is now defined when reading `/proc` files
- Defined behaviour of Thai numerals in postfix/prefix `++`/`--` on strings
- `map` inside sunk `for` is now treated as sunk
- Sunk `for` loop now sinks value of last statement's method call
- `.Int` on `Bool` objects now returns an `Int` object
- `splice` can now be used to extend an array
- `classify` now works with `Junction`s
- `.pairup` on a type object returns an empty `Seq`
- Synthetic codepoints are rejected from `Date`/`DateTime` constructors
- `⸨`/`⸩` pair can now be used as matching characters in quoting constructs
- `.flat` on `Array` typeobject simply returns that type object
- Mixed-level `classify` on `Hash`es now throws
- `Junction`s can now be used to specify multiple keys to `Hash`es
- The `Callable` given to `.classify-list` is now guaranteed to be
    executed only once per each item
- `:delete` on associative lookup on `Hash` type object returns `Nil`
- `is-deeply` from Test.pm6 now automatically `.cache`s `Seq`s given
    as arguments and uses the returned `List`s for testing
- `Complex.new()` now gives `<0+0i>`
- `Int.new` is now guarantied to construct a new `Int` (rather than, say,
       re-use one from a constants cache)
- 1-arg versions of `&infix:<=:=>` and `&infix:<eqv>` are now defined
- `Nil` type now throws if directly or indirectly calling
    `.BIND-POS`, `.BIND-KEY`, `.ASSIGN-POS`, `.ASSIGN-KEY`, `.STORE`, `.push`,
    `.append`, `.unshift`, `.prepend`
- `Nil.ord` returns an empty `Seq`
- `Nil.chrs` returns a `"\0"`
- `Num.new` coercers argument to `Num`
- `infix:<Z>()` returns an empty `Seq`
- Reduce with `&infix:<+>` with one item now simply returns that item
- `()[0]` returns `Nil`
- Regex smartmatching is now allowed on (possibly-infinite) `Seq`
- `Set` converted to a `Mix`/`Bag` no longer has `Bool` weights

#### Miscellaneous

- The `IO::ArgFiles` type is now just an empty subclass of `IO::CatHandle`
- Constraints on constants
    - Constraints are now fully enforced
    - Attempting to parametarized type constraints on constants
        (i.e. using `my Foo constant @int`) now throws `X::ParametricConstant` exception
- Native `num` variables now default to `0e0` instead of `NaN`
- Pod `=defn` (definition list) directive is now available
- `.^ver`, `.^auth`, and `.^name` metamethods are now available on `module`
    and are absent on a `package`, by design
- Fancy quotes (`’…’`, `“…”`, `｢…｣`, and variants) are now supported in `qww<…>`
- `&infix:< >` supports lookup of autogenerated `Callables` (e.g. `&infix:<XX>`)
- Using a named `anon` sub does no longer produces redeclaration warnings
- Extended spec of `::?MODULE`/`$?MODULE` variable
- `sub MAIN` can now accept a `where` clause on arguments
- Type smiley constraints can now be used on subsets
- `start` blocks and thunks now get fresh `$/` and `$!`
- `R` meta operator used with list-associative operators is now defined
- Type coerces can now be used in signature return type constraints
- `&infix:<x>`/`&infix:<x>` now throw with `-Inf`/`NaN` repeat arguments
- Literal constructs `put` and `put for` now throw, requiring use of parentheses

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
- `&undefine` (assign `Empty`/`Nil` directly, instead)
- `:count` argument on `&lines`/`Str.lines` routines
    (use `.elems` call on returned `Seq` instead)
- `&is_approx` in Test.pm6 (use the very similar `&is-approx` instead)

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
