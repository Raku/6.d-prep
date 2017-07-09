
# List of TODO items required for 6.d language release

## FEATURES

The [FEATURES.md](FEATURES.md) lists the proposed features that need to be
implemented by their stakeholders.

## Documentation Website

Implement ability to mark features with the language version they first
appeared in.

## Roast

Review [all the new tests since
6.c](https://github.com/perl6/roast/compare/HEAD...6.c-errata) to ensure they
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

## Feature List

Collect all the features that will be new in 6.d. Many are already implemented,
documented, and are in use, but they're not part of 6.c language
(e.g. `Str.parse-base`)
