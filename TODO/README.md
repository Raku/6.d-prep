
# List of TODO items required for 6.d language release

## FEATURES

The [FEATURES.md](FEATURES.md) lists the proposed features that need to be
implemented by their stakeholders.

## Documentation Website

Implement ability to mark features with the language version they first
appeared in.

## Roast

### Remove dead files

Remove all the files that are currently skipped.

### Passing Tests

Ensure Roast stresstest passes on Windows. Currently three open Issues
exists, showing a bunch of failing tests [#320](https://github.com/perl6/roast/issues/320),
[#232](https://github.com/perl6/roast/issues/232), and [#197](https://github.com/perl6/roast/issues/197)

### Review 

Review [all the new tests since
6.c](https://github.com/perl6/roast/compare/6.c-errata...HEAD) to ensure they
do belong in 6.d and test desired behaviour.

#### Review Process

The goal of the review is to ensure all of the commits added to `master` since
`6.c` specify behaviour we actually want to include in `6.d` language release.

One way to do that is to go through all the commits and see if any of the
newly added tests look off. To assist with that, you can create a handy alias:

```bash
    alias glop="git log --pretty=format:'%C(yellow)https://github.com/perl6/roast/commit/%H | %Cred%ad | %Cgreen%d %Creset%s' --date=short --reverse"
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

If you spot any questionable tests, bring it up in
[#perl6-dev](https://webchat.freenode.net/?channels=#perl6-dev)

#### Reviewers

* Zoffix (reached commit `5fb2feb9f68f8ea208754bb8f68372e9def63b35`)

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

## Feature List

Collect all the features that will be new in 6.d. Many are already implemented,
documented, and are in use, but they're not part of 6.c language
(e.g. `Str.parse-base`)

## Experimental Features

See which features that currently require `use experimental` are solid enough to
make them non-experimental in 6.d

## See if we can nail down the META6.json spec

And codify it as tests in roast. Even if entire spec won't be done, would be nice to
standardize [`"auth"` key](https://irclog.perlgeek.de/perl6/2017-08-09#i_14991431) because
we're basically ready to have multi-source ecosystem that supports same-name-multiple-author
naming, but without a nailed down `"auth"` it's hard to make use of that system.

---------

## NAMING

### Release Name

The name "Diwali", planned in 2015 to go along with "Christmas" is LTA IMO (Zoffix), as it
encodes in itself a specific date we're unlikely to meet, and if we meet it, we set a
precendent we might not want to adhere to.

So instead of "Diwali" I propose we use butterfly-related themes instead. e.g. butterly
specifies, genera, or other related names.

So far my (Zoffix's) favourite is: **6.d "Dismorphia"**
It sounds really cool and not butterfly-y at all, so when the
listener learns that we have this whole them of doing butterfly stuff
it kicks up the coolness a notch :)

Dismorphia is a [genus of butteflies](https://en.wikipedia.org/wiki/Dismorphia)
and despite my huge distaste for insects, Dismorphias actually look pretty cool:

![](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1f/Dismorphiapraxinoemale.jpg/1920px-Dismorphiapraxinoemale.jpg)

Other names that start with D are butterfly common names: "Dreamy Dusky-Wing", "Dotted Blue", and "Dainty Sulphur"

### Language Extended Naming

As elaborated [here](https://rakudo.party/post/The-Hot-New-Language-Named-Rakudo) with comments
[here](https://www.reddit.com/r/perl/comments/6lstqu/the_hot_new_language_named_rakudo/) and
[here](https://www.reddit.com/r/perl6/comments/6lstq3/the_hot_new_language_named_rakudo/), there's
a push to improve the language name.

The current mood, as I (Zoffix) perceive it, is: there's room for a name extension (e.g. "Foo Perl 6")
where "Foo" can be a standalone word people who don't wish to use "Perl" can use. However, the
change for an entirely new name is not desired.

To resolve this issue, a month before release date, Zoffix is to:

- Collate all the feedback on the matter in presentable, easy to digest manner
- Collect all of the suggested name extensions offered by various sources
- Prepare any marketing plans, if time/skill allows, to show the benefits of adjusting the name
- Give the things to TimToady and ask for:
    - Executive decision on whether the name extension can be officially made
    - If yes, decide what the official name extension is

TimToady will have a month to think on the matter and is to give the final decision prior to the 6.d language release.

As this Naming Issue is contentious, please try to place your feedback not into this repo, but
preferably in [the reddit thread](https://www.reddit.com/r/perl6/comments/6lstq3/the_hot_new_language_named_rakudo/),
or, if you don't consider yourself a regular member of the Perl 6 Community, in the
[perl 5 reddit thread](https://www.reddit.com/r/perl/comments/6lstqu/the_hot_new_language_named_rakudo/) instead.
If you don't have access to reddit, you can DM your feedback [to Zoffix on Twitter](https://twitter.com/zoffix)
or send a private message to user `Zoffix` on [IRC](https://webchat.freenode.net/?channels=#perl6)

