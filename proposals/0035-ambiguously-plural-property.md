# (IE-0035) Ambiguously plural property

* Proposal: [IE-0035](0035-ambiguously-plural-property.md)
* Discussion PR link: [#35](https://github.com/ganelson/inform-evolution/pull/35)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: --
* Implementation: Implemented but unreleased

## Summary

To make the `ambiguously plural` property cause an object to set both of the
command parser's pronouns `IT` and `THEM`, regardless of whether the object
is `singular-named` or `plural-named`. By contrast, an object which is not
`ambiguously plural` sets `IT` if it is `singular-named` and `THEM` if it is
`plural-named`, but not both. The idea is to give this property to things
with names such as "string of pearls", so that `TAKE STRING. EXAMINE IT` and
`TAKE PEARLS. EXAMINE THEM` would both work.

## Motivation

This very minor change has its own IE proposal only because it both surfaces a
previously undocumented feature, which a few people have used over the years,
and also slightly changes its functioning.

Since we are bringing it out into the open, this seems the time to make it more
consistent and symmetrical. See the discussion here for the sort of confusion
previously caused:

https://intfiction.org/t/i-dont-know-how-to-describe-this-in-one-brief-sentence-solved/58709/4

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor change to CommandParserKit.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

This proposal will change the meaning of the `IT` pronoun after a command which
refers to an item which is both `ambiguously plural` and also `plural-named`.
This is very unlikely to cause problems.
