
This document contains planned changes for v6.d that we postponed until v6.e
for various reasons.

## Swap IO::Path.child to use .child-secure's code

----

Postponed until 6.e because there's a huge amount of ecosystem usage
and the IO::Path.add that is the alternative is too new for module
authors to reliably switch to using it.

----

`.child-secure`'s code is commented out below .child's code. Per IO Grant work.
Discussion: https://irclog.perlgeek.de/perl6-dev/2017-04-17#i_14439386
Docs for .child-secure: https://github.com/perl6/specs/blob/master/v6d.pod

See if there's a way to make even securer operations:

> open the file first and then use readlink syscall on it

https://irclog.perlgeek.de/perl6/2017-07-21#i_14904782

### Stakeholder

Zoffix

### Time Required to Implement

40 hours

