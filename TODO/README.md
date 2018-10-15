
# List of TODO items required for 6.d language release

## FEATURES

The [FEATURES.md](FEATURES.md) lists the proposed features that need to be
implemented by their stakeholders.

## Documentation Website

Implement ability to mark features with the language version they first
appeared in.

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

### Review

Review [all the new tests since
6.c](https://github.com/perl6/roast/compare/6.c-errata...HEAD) to ensure they
do belong in 6.d and test desired behaviour.

#### Review Process

The goal of the review is to ensure all of the commits added to `master` since
`6.c` specify behaviour we actually want to include in `6.d` language release.

If you need to make some changes in the code you're reviewing, put
`[v6.d REVIEW] ` into the commit title. This way we'll be able to filter these
out and tell how many "actual" new commits are in the 6.d language spec.

One way to do a review is to go through all the commits and see if any of the
newly added tests look off. To assist with that, you can use the
[spec view tool](https://github.com/perl6/roast/commit/1c5da5181b) or
create a handy alias:

(shell version)
```bash
    alias glop="git log --pretty=format:'%C(yellow)https://github.com/perl6/roast/commit/%h | %Cred%ad | %Cgreen%d %Creset%s' --date=short --reverse"
```

(html version)
```bash
alias glop="git log --pretty=format:'<a href='\''https://github.com/perl6/roast/commit/%h'\'' style='\''font-family: monospace'\'' target=_blank>%H | %ad | %s</a><br><br>' --date=short --reverse"
```

Then, in checkout of [the roast repo](https://github.com/perl6/roast/) run
`glop` alias giving the last commit you reviewed (or `6.c-errata` if you
haven't yet reviewed any commits. E.g.:

```bash
    git clone https://github.com/perl6/roast/ ~/roast-6.d-review
    cd ~/roast-6.d-review
    git checkout 6.c-errata; git checkout master
    glop 6.c-errata...HEAD
```

Say, you stopped at commit `4a59ba39a8bec2c746d3ae34cd67fde3bcca25cd`. When
you come back later to review some more, run:

```bash
    cd ~/roast-6.d-review
    glop 4a59ba39a8bec2c746d3ae34cd67fde3bcca25cd...HEAD
```

With HTML version, save it to a file and view the file in the browser:

```bash
    cd ~/roast-6.d-review
    glop 4a59ba39a8bec2c746d3ae34cd67fde3bcca25cd...HEAD > 6.d.html
    chromium-browser 6.d.html
```

Note that with the above one liners, some times you'd still end up with
a couple of old commits at the top of the list, even after re-filtering to
a newer commits. There's probably some `git` reason behind that, but
I don't know what it is.

If you spot any questionable tests, bring it up in
[#perl6-dev](https://webchat.freenode.net/?channels=#perl6-dev)

#### Reviewers

* Zoffix (reached commit: *review COMPLETED*)
    - Does only superficial review of Unicode, CompUnit, Pod, and QuantHash op tests due to limited knowledge of those features
    - Does not do an in-depth review of many of own tests on assumption the tests
        were already well-thought and well-researched when they were written

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

## See if we can nail down the META6.json spec

And codify it as tests in roast. Even if entire spec won't be done, would be nice to
standardize [`"auth"` key](https://irclog.perlgeek.de/perl6/2017-08-09#i_14991431) because
~~we're basically ready to have multi-source ecosystem that supports same-name-multiple-author naming, but without a nailed down "auth" it's hard to make use of that system.~~ *we already have it and we [do have problems](https://github.com/perl6/modules.perl6.org/issues/106)*

A roast Issue for the `auth` key has been filed as well: https://github.com/perl6/roast/issues/450

