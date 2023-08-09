# (IE-0031) BANISH debugging command

* Proposal: [IE-0031](0031-banish.md)
* Discussion PR link: [#31](https://github.com/ganelson/inform-evolution/pull/31)
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: --
* Implementation: Implemented but unreleased

## Summary

A new debugging command, BANISH, which removes things from play.

## Motivation

The ABSTRACT command can move things from one place to another, but is unable
to remove something from the object tree altogether, so that it is out of play.

See Jira bug report [I7-2140](https://inform7.atlassian.net/issues/I7-2410),
where Andrew Schultz proposes this.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, since in contemporary Inform any command clash between the user's source
text and the debugging commands in `WorldModelKit` is resolved amicably (i.e.,
if the author of a project also creates a command BANISH, it won't throw
problems because of the name-clash with this one).
