
This document contains planned changes for v6.d, and who wants to do them.

Please list yourself as stakeholder so we'd know whom to contact if we need
clarification. If possible, find a volunteer willing to implement your
proposal (it could be you).

## [PARTIALLY IMPLEMENTED] Non-blocking await

*Note: appears to be implemented, but [based on
this comment](https://irclog.perlgeek.de/perl6-dev/2017-07-06#i_14835079) looks
like more work is needed*

In v6.c, waiting for a promise, either with an explicit `await` or by using its
`.result` method, currently uses up a thread just for the blocking wait.

After this proposed change has been implemented, waiting doesn't tie up
a whole thread for each wait.

### Rationale

It is quite easy to write code that waits for multiple promises in parallel,
which can lead to the full thread pool begin tied up in blocking waits,
leading to a deadlock.

Examples of this happening have showed up in the past.

### Stakeholder

Jonathan Worthington

### Time Required to Implement

???

# IN NEED OF IMPLEMENTATION

## `use v6.d.PREVIEW` in wrong place needs to throw

For example, this code silently fails. It should throw the same style of
error as when, for example, you use `unit`... *"Too late to blah blah..."*

```perl6
    use v6.c; sub foo { use v6.d.PREVIEW; await start say 42 }()
```

# PROPOSED FEATURES

## Sigils imply :D

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

## Formal Rules for Defining Matched Delimiters/Brackets

In v6.d we should formalize which brackets we support.
How do we decide which delimiters should be added on future updates
to the Unicode standard? We should look to the Unicode standard to help
us define matching delimiters and brackets for Perl 6.

All delimiters we support should conform to two simple rules for the sake of
uniformity, elegance and clairity.

### Rules

1. Delimiter's Unicode General_Category must match one of these:

    Pi -> Pf ( Punctuation, initial quote -> Punctuation, final quote)
    Ps -> Pe (Punctuation, start -> Punctuation, end)

2. The delimiters must be matching BidiBrackets and/or BidiMirroring characters.

Bidirectional brackets are specified
[here](http://www.unicode.org/Public/UCD/latest/ucd/BidiBrackets.txt)

Non brackets have their matching glyph specified in this
[file](http://www.unicode.org/Public/UCD/latest/ucd/BidiMirroring.txt)

### Possible issues

The only possible issue, is what to do with ornate parens.

**BidiBrackets.txt** states:

“For legacy reasons, the characters U+FD3E ORNATE LEFT PARENTHESIS and
U+FD3F ORNATE RIGHT PARENTHESIS do not mirror in bidirectional display
and therefore **do not form a bracket pair.**”

In v6.c, roast includes tests for 'ornate left parens' and 'ornate right parens'
for doing things like `q[ ]` type contructs and such. I think that we should not
allow these parenthesis because firstly, Unicode states they do not form a matching pair
of brackets. Secondly, the ornate parenthesis also do not have mirror glyphs.
To make matters even worse, their Unicode general categories are the opposite of every
matched bracket we support, the opening brackets tested for in v6.c open with
"Pe"(End) and close with is "Ps"(Start).
They break both of these proposed rules.

In practice this is already implement with the exception of the ornate parenthesis,
but I propose this be made an official part of the Perl 6 standard.

### Stakeholder

Samantha McVey (samcv)

### Time Required to Implement

???

## Remove deprecated Test::is_approx

This is mostly a reminder so we don't forget. Test::is_approx is currently marked as deprecated
(replaced by `is-approx`). It was decided to leave it in until 6.d, so that we can still use it
in 6.c-errata without any changes.

### Stakeholder

Oh, I thought you said steak... (Zoffix)

### Time Required to Implement

10 hours

## Properly reserve all `:sym<>` colonpairs on subroutines

This is mostly a reminder so we don't forget. As
[previously discussed](https://irclog.perlgeek.de/perl6/2017-01-25#i_13988093),
We want to make `:sym<>` colonpairs on subroutines reserved. However,
there is [a 6.c-errata test](https://github.com/perl6/roast/blob/dfe905a8ce84d09b5b0536fcc151e798488a9289/S32-exceptions/misc.t#L132)
that expects `sub foo:sym<bar> {}` `X::Syntax::Extension::Category`
exception instead of `X::Syntax::Reserved` we desire. The
[code implementing this](https://github.com/rakudo/rakudo/commit/48abeeef26)
already exists. It just needs to be uncommented for 6.d and
[corresponding tests](https://github.com/perl6/roast/commit/53d6e8491d)
unfudged.

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

## Use IEEE 754-2008 semantics for num/Num infix:</>, infix:<%>, and infix:<%%>

**(Sidenote: be sure to check log(42, 1) does not explode when this is implemented.
If it's decided not to implement this; change log(42, 1) to give a better error**

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

### Stakeholder

Zoffix

### Time Required to Implement

6 hours

## Deprecate IO::Path.chdir and IO::Handle.slurp-rest

Per IO Grant work.

* `IO::Path.chdir` - the name is a poor choice, as it doesn't **ch** hange
    any **dir** ectories. Functionality with relative-path argument is directly
    replaced by `IO::Path.add` method and functionality with absolute-path is
    directly replaced by `IO::Path.new` or `Cool.IO` methods.

* `IO::Handle.slurp-rest` - removed to make interface consistent with
    the rest of the language that uses `slurp` as the name for the routine.
    The `IO::Handle.slurp` was already implemented as part of the IO Grant.

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

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

## Remove Str.lines :$count parameter

An entirely pointless, undocumented, unused, and unloved parameter whose removal is blocked by
[two 6.c tests](https://github.com/perl6/roast/blob/5c703c355d0457f78f1f32a0f3af394ab54be256/S32-str/lines.t#L59-L60)

### Stakeholder

Zoffix

### Time Required to Implement

2 hours

## Remove $/ magicalness from %() and @()

And perhaps $() too; Per https://rt.perl.org/Ticket/Display.html?id=131392

### Stakeholder

Zoffix

### Time Required to Implement

30 hours

## Remove `$*MAIN-ALLOW-NAMED-ANYWHERE`

It was never specced and currently exists for backwards compatibility with older panda. By 6.d it should not matter and could be
removed.

### Stakeholder

Zoffix

### Time Required to Implement

1 hour

## Make `$*ARGFILES` := `$*IN` or `IO::ArgFiles.new($*IN)` inside MAIN

It being based on `@*ARGS` is virtually never useful when you handle any sort of non-file arguments (which is what using MAIN implies).
To allow lines(), get(), words(), and slurp() to Do The Right Thing when called and use `$*IN`, we should swap `$*ARGFILES` to `$*IN`
(or to `IO::ArgFiles.new($*IN)`, to maintain `$*ARGFILES` type consistency),
when we're inside `sub MAIN(){}` and only maintain the current behaviour of considering `@*ARGS` when outside of MAIN.

Relevant discussions:

* https://rt.perl.org/Ticket/Display.html?id=131703#txn-1471796
* https://irclog.perlgeek.de/perl6/2017-07-05#i_14827209
* https://irclog.perlgeek.de/perl6-dev/2017-07-05#i_14827981

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

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


## Spec IO::Handle's `.write-internal`, `.read-internal`, `.eof-internal`

The method names should be changed as currently they imply Rakudo-internal methods
rather than methods users should be overriding to affect all read/write methods.

They should probably use different casing to differentiate them from regular IO::Handle
methods consumers of IO::Handle would use. Perhaps, `.WRITE-SOURCE`, `.READ-SOURCE`,
and `.EOF-SOURCE`; as the methods are source for all the other read/write methods.

Spec and document the methods as the proper way to create custom IO::Handle
subclasses.

### Stakeholder

Zoffix

### Time Required to Implement

4 hours

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
