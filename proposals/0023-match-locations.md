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

- start index of the/-- text match ... number
- final index of the/-- text match ... number
- length of the/-- text match ... number
- start index of subexpression (number) ... number
- final index of subexpression (number) ... number
- length of subexpression  (number) ... number

Every text (including "") matches the text "". In this case, the start index,
final index, and length are all 0.

In many languages (Perl, Python, Ruby, Javascript), any string matches an
empty regular expression; Inform hews to the arguably more correct position
that only the empty string matches the empty regular expression. They all
agree that only the empty string matches `^$`. If the empty string successfully
matches any regular expression, the results for start index, final index,
and length are all 0.

Any text will match the regular expression `(?:)`; the resulting start index,
final index, and length are all 0 for that case. But in the general case
for non-empty texts, the values will all be 1 or more, as one would expect
given Inform's 1-indexed texts.

The documentation will reveal that subexpression 0 can be used to mean the
whole of the text matching the regexp, outside of any capture groups. (This
was already true, but undocumented.)

As with `text matching the regular expression` or `text matching subexpression
<n>`, these phrases should be invoked immediately after a successful match: the
next matching operation will clobber their values. The values of these phrases
after an unsuccessful match is undefined behavior. The documentation will make
clear that they're only meaningful after a successful match and must be used
promptly.
