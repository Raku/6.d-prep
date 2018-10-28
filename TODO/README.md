
# List of TODO items required for 6.d language release

## FEATURES

The [FEATURES.md](FEATURES.md) lists the proposed features that need to be
implemented by their stakeholders.

## Roast

### Throw out / Appendicize Exception Tests

The way we created/thrown new exceptions is very ad-hoc and needs more thought.
Don't spec particular exceptions in new tests until we have the time to
properly evaluate Exceptions hierarchy.
http://colabti.org/irclogger/irclogger_log/perl6-dev?date=2018-09-21#l142


### Remove dead files

Remove all the files that are currently skipped. (e.g. S24-testing/1-basic.t,
which probably needs to be fixed, as its tests are wrong and I don't see any
other tests testing these routines)

### Add Missing `plan`s in substests

[Some subtests](https://github.com/perl6/roast/commit/991398bb7) are missing
a `plan` call inside of them. Some programatic method should be developed
to detect this issue and fix it.

### Passing Tests

Ensure Roast stresstest passes on Windows. Roast Issue
[#320](https://github.com/perl6/roast/issues/320) showing a bunch of failing tests.

## Define More Concrete Policies For Implementation of New Features

Work in progress available in [`d-docs/New-Features-Policy.md`](d-docs/New-Features-Policy.md)

## Feature List

Collect all the features that will be new in 6.d. Many are already implemented,
documented, and are in use, but they're not part of 6.c language
(e.g. `Str.parse-base`)

Note, some routines are meant to be deprecated in 6.d but
[we lack the means to emit a deprecation
warning](https://github.com/rakudo/rakudo/issues/1289) based on language version.
These routines should be logged in release docs as deprecated even though they
might stick around for awhile and start to warn only in 6.e. These are listed
as planned for deprecation in [`deferred-to-6.e`](deferred-to-6.e/) files.

## Experimental Features

See which features that currently require `use experimental` are solid enough to
make them non-experimental in 6.d
