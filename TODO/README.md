
# List of TODO items required for 6.d language release

## FEATURES

The [FEATURES.md](FEATURES.md) lists the proposed features that need to be
implemented by their stakeholders.

## Documentation Website

Implement ability to mark features with the language version they first
appeared in.

## Roast

### Remove dead files

Remove all the files that are currently skipped. (e.g. S24-testing/1-basic.t,
which probably needs to be fixed, as its tests are wrong and I don't see any
other tests testing these routines)

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
newly added tests look off. To assist with that, you can create a handy alias:

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

* Zoffix (reached commit `c010174ac7027b2b6bb2b541f01158cb1a6a0b9d`)

## Define More Concrete Spec Errata Rules

* Define specific protocol for how past specs can be changed
* What is a "test that's wrong"?
    * How is that wrongness measured?
    * Who and how many people decide that it's wrong?
    * Which code is considered the test itself and which code is merely
        supporting code that isn't explicitly being tested by the test?
* How do we make users aware of spec errata changes?
* What does the core dev do with code they wrote that turned out to break possibly-wrong tests?
    * Specify specific branch where such commits should be kept in
    * Avoid committing this stuff to `nom` until decision is reached that tests are indeed wrong
* Make it clear to all core devs that this protocol must be adhered to

## Define More Concrete Policies For Deprecations/Changes with Large Impact

* What's our deprecation period for features?
* What's our user notification chain for changes that might impact them?
    * ~~Define where users can watch for critical notifications~~ exists now as alerts.perl6.org
    * Define how much notice we must give users before we introduce potentially
       breaking changes, which aren't necessarily have to do with language, but
       even things like infrastructure changes (e.g.
       [how well was the nom to master rename handled?](https://irclog.perlgeek.de/perl6-dev/2017-10-27#i_15360590)
       should we have given bleed users more notice?)

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

---------

## NAMING

### Release Name

A popular proposed name for the release is the name "Diwali", planned in 2015 to go along with "Christmas".

### Language Extended Naming

As elaborated [here](https://rakudo.party/post/The-Hot-New-Language-Named-Rakudo) with comments
[here](https://www.reddit.com/r/perl/comments/6lstqu/the_hot_new_language_named_rakudo/) and
[here](https://www.reddit.com/r/perl6/comments/6lstq3/the_hot_new_language_named_rakudo/), there's
a push to improve the language name.

The current mood, as I (Zoffix) perceive it, is: ~~there's room for a name extension (e.g. "Foo Perl 6")
where "Foo" can be a standalone word people who don't wish to use "Perl" can use~~ (UPDATE: we're now [aiming for a stand-alone *alias*](https://rakudo.party/post/6lang-The-Naming-Discussion-Update#the6lang) rather than extension). However, the
change for an entirely new name and dropping "Perl 6" currently does not have sufficient support.

To resolve this issue, a month before release date, Zoffix is to:

- Collate all the feedback on the matter in presentable, easy to digest manner
- Collect all of the suggested name aliases offered by various sources
- Prepare any marketing plans, if time/skill allows, to show the benefits of adjusting the name
- Give the things to TimToady and ask for:
    - Executive decision on whether the name alias can be officially made
    - Decide what the official name alias is going to be

TimToady will have a month to think on the matter and is to give the final decision prior to the 6.d language release.

As this Naming Issue is contentious, please try to place your feedback not into this repo, but
preferably in [the reddit thread](https://www.reddit.com/r/perl6/comments/6lstq3/the_hot_new_language_named_rakudo/),
or, if you don't consider yourself a regular member of the Perl 6 Community, in the
[perl 5 reddit thread](https://www.reddit.com/r/perl/comments/6lstqu/the_hot_new_language_named_rakudo/) instead.
If you don't have access to reddit, you can DM your feedback [to Zoffix on Twitter](https://twitter.com/zoffix)
or send a private message to user `Zoffix` on [IRC](https://webchat.freenode.net/?channels=#perl6)

UPDATE: more info on the naming issue: https://6lang.party/post/6lang-The-Naming-Discussion-Update
