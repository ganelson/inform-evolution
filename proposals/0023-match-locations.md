# (IE-0023) Retaining text/regexp match start/end locations

* Proposal: [IE-0023](0023-match-locations.md)
* Discussion PR link: [#23](https://github.com/ganelson/inform-evolution/pull/23)
* Author: Zed Lopez
* Language feature name: None
* Status: Accepted in principle
* Related proposals: None
* Implementation: Implemented but unreleased

## Summary

Retain the start and end text indices of text/regexp matches and provide phrases
to make them available to Inform 7.

## Motivation

During processing of text and regular expression matches, TEXT_TY_Replace_REI
determines the beginning and ending string indices of the matches, on a per
subexpression basis for the latter. In the text case, this information was
discarded immediately; for the regular expression case, it was retained, but
not made available to Inform 7.

This proposal is for a trivial change to RegExp.i6t so that TEXT_TY_Replace_REI
stores the information for the text matching case (in two new global variables)
and to add phrases to Basic Inform to make the start and final indices of the
matches and their lengths available to Inform 7.

Having this information makes it straightforward to implement some useful things
that would otherwise be cumbersome:

- splitting a text on a given substring or regexp, returning a list of texts
- replacing just the *first* occurrence of a matched text or regular expression

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor change to BasicInformKit.
- [x] Minor changes Basic Inform.
- [x] Minor addition to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, since this feature is entirely additive.

## Specifics

The new phrases are:

- first index of the/-- text match ... number
- last index of the/-- text match ... number
- length of the/-- text match ... number
- first index of subexpression (number) ... number
- last index of subexpression (number) ... number
- length of subexpression  (number) ... number

After `if "shinto" matches the text "hint"`, `first index of the text match`
would be 2, and `last index of the text match` would be 5, i.e., these are
the same 1-indexed values one would use with `character number <n> of <t>`
to get "h" and "t" in "shinto".

Every text, including the empty text "", matches the text "". In this case,
the first index, last index, and length are all 0. This is the only case where
any of those would be 0 following a successful text match.

Following `if "educate" matches the regular expression "du(cat)"` the `first
index of subexpression 1` would be 4 and `the last index of subexpression 1`
would be 6. The `first index of subexpression 0` would be 2; `the last index
of subexpression 0` would also be 6. (The documentation will reveal that
subexpression 0 can be used to mean the whole of the text matching the regexp,
including what was outside of any capture groups: this was already true, but
undocumented.)

In many languages (Perl, Python, Ruby, Javascript), any string matches an
empty regular expression; Inform hews to the arguably more correct position
that only the empty string matches the empty regular expression. If the empty
string successfully matches *any* regular expression, the results for first index,
last index, and length of subexpression 0 will all be 0.

Unlike with the text match case, it is possible for the results to all be 0
when a non-empty text successfully matches a regular expression. For instance,
`if "Q*bert" matches the regular expression "x?"` is true, and subsequent to
it, first index, last index, and length of subexpression 0 will be 0.

Subsequent to any successful text or regular expression matches, it should
always be the case that either all of the values for first index, last index,
and length should be 0, or that none of them are. In the non-zero case, the
length is always the 1 + the last index - the first index.

As with `text matching the regular expression` or `text matching subexpression
<n>`, these phrases should be invoked immediately after a successful match: the
next matching operation will clobber their values. The values of these phrases
after an unsuccessful match is undefined behavior. The documentation will make
clear that they're only meaningful after a successful match and must be used
promptly.
