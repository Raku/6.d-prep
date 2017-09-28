
This document contains already implemented planned changes for v6.d.
See [FEATURES.md](FEATURES.md) for features yet to be implemented.

# Implemented

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
