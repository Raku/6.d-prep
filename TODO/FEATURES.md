
This document contains planned, but not yet completed, changes for v6.d, and who is responsible for implementing them.

If you're contemplating implementing a new feature, consider implementing it in 6.e language instead.


# IN NEED OF IMPLEMENTATION

## `use v6.d.PREVIEW` in wrong place needs to throw

For example, this code silently fails. It should throw the same style of
error as when, for example, you use `unit`... *"Too late to blah blah..."*

```perl6
    use v6.c; sub foo { use v6.d.PREVIEW; await start say 42 }()
```

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

