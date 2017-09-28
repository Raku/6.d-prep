
This document contains already implemented planned changes for v6.d.
See [FEATURES.md](FEATURES.md) for features yet to be implemented.

## [IMPLEMENTED] Consistify Naming of `$*INITTIME`

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


## [IMPLEMENTED] Remove dummy precision parameters from Rational/Int .Rat and .FatRat coercers

**Since we can't modify methods between languages, the params were deprecated instead,
for removal in 6.e**

They're dummy parameters that offer more confusion than usefulness. The roast itself
seems confused. There are a whole bunch of trig tests that use these coercers with a
precision arg for no good reason; almost feels like the writer assumed `1.5` is a Num and
not a Rat.

Done in: https://github.com/rakudo/rakudo/commit/4c337e8ef9


## [IMPLEMENTED] Deprecate IO::Path.chdir and IO::Handle.slurp-rest

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
