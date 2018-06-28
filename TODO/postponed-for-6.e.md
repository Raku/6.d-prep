
This document contains planned changes for v6.d that we postponed until v6.e
for various reasons.

## Implement secure way of opening a path

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

