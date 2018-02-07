
This document contains planned changes for v6.d, and who wants to do them.

Please list yourself as stakeholder so we'd know whom to contact if we need
clarification. If possible, find a volunteer willing to implement your
proposal (it could be you).

# IN NEED OF IMPLEMENTATION

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

## Rename `.parse-names` to `.uniparse`

*UPDATE: `.uniparse` is now implemented: https://github.com/rakudo/rakudo/commit/2a8287cf89 
https://github.com/perl6/roast/commit/3efe6cb8da https://github.com/perl6/doc/commit/bff42f80b1 ;
Still need to add a deprecation warning
in `.parse-names` in 6.d, once machinery allowing that exists.*

-------

Altough `.parse-names` was never part of `6.c`, there's some use of it
in ecosystem and likely elsewhere, so it should undergo a deprecation period.

The original name was chosen to align with `.parse-base` that parses out base-X numbers out
strings. However, we have a whole block of more closely related routines all named in `.uni*`
format, so it makes sense for this routine's naming to align with those. They are: 
`infix:<unicmp>`, `unimatch`, `uniname`, `uninames`, `unival`, `univals`,
`uniprop`, `uniprop-bool`, `uniprop-int`, `uniprop-str`, `uniprops`.

### Stakeholder

Zoffix

### Time Required to Implement

3 hours

## Remove $/ magicalness from %() and @()

And perhaps $() too; Per https://rt.perl.org/Ticket/Display.html?id=131392

### Stakeholder

Zoffix

### Time Required to Implement

30 hours


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
methods consumers of IO::Handle would use. Perhaps, `.WRITE`, `.READ`,
and `.EOF`, but would it be confusing with the lower-case methods of the same name?

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

## Make `start` blocks in sink context attach an error handler

At the moment, any exceptions leaking out will be lost. For this particular case, determined
through semantic analysis at compile time, a `then` should be attached that will `die` with
any error that occurs during the execution of the `start` block and goes unhandled by it, so
the errors will not be lost. Such an error will trigger `uncaught_handler` in the thread pool,
which by default will bring down the program. Note this is *not* a general sink operation on a
`Promise`; it applies only to the `start` syntax.

### Rationale

It's rare that people really want to ignore that some background worker they set off failed.
Most `start` blocks are `await`ed or `then`'d because people care that the work completes.
The case for an unguraded `start` is most often setting off some application-lifetime worker.
In that case, the behavior being that it crashing brings the application down is the right
thing anyway, and desirable given the alternative would be that the application silently stops
responding, if it was a `start react` that was meant to handle events over the application's
lifetime.

### Stakeholder

Jonathan Worthington

### Time Required to Implement

1 day

## Make `.?` safecall actually safe

By performing a `.cando` (or some cheaper alternative) in `dispatch:<.?>`

### Rationale

If I use the "safe call" method I expect a call done if possible, or if not
a `Nil` to be returned. It's irrelevant to me whether an object `.can`
a method; I care about whether my call with the given arguments can actually
be performed.

The current implementation suffers issues even with core `Nil` implementation,
since it inherits methods from `Cool` and many of them have a `:D` constraint
on the invocant. This causes usecases such as this:

    "x".?meow.?starts-with: "foo"

To crash because `.starts-with` is called on a `Nil:U` returned by previous
safe-call. This sort of crashes make safe-call op rather useless.

By making safecall perform a `.cando`-equivalent check to ensure the call
with the given args can be dispatched will resolve the `Nil:U` issue
shown above as well as make the safecall a lot more safer.

### Stakeholder

Zoffix Znet

### Time Required to Implement

1 day

## Make method matcher forms not set `$/`

For rationalle and discussion see: https://github.com/rakudo/rakudo/issues/1235

This change depends on how the discussion in that Issue concludes.

### Stakeholder

Zoffix Znet

### Time Required to Implement

1 day

## Make default defaults for DefiniteHOWs be normal types

Basically, so you could do `my Foo:D $x .= new;`, which
currently dies since you can't instantiate DefiniteHOWs.

There's an open Rakudo issue for this that also contains
a potential non-version impl in a branch:
https://github.com/rakudo/rakudo/issues/1493

### Stakeholder

Zoffix Znet

### Time Required to Implement

2 hr
