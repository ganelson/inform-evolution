# (IE-0041) Very first and very last rules

* Proposal: [IE-0041](0041-very-first-rules.md)
* Discussion PR link: [#41](https://github.com/ganelson/inform-evolution/pull/41)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: None
* Status: Implemented but unreleased

## Summary

To provide for rulebooks to have a designed `very first` rule, guaranteed
always to be present, which can initialise rulebook variables before any other
rules can access them. For symmetry, `very last` rules are also provided.

## Motivation

At present it's difficult for extension authors to set up a rulebook which has
variables needing to be initialised to (non-default) values. The trouble with this:

	The example rules are a rulebook.
	
	The example rules have a number called the magic number.
	
	First example rule:
		now the magic number is 77.

...is that it depends on the `first example rule` executing first. What if the
user of the extension also creates a `first` rule in this rulebook? What if
another extension tries to unlist the initialisation rule, or substitute
another rule for it? No guarantees can be made.

## Details

Optionally, a rulebook can have a rule defined which is guaranteed to be
followed first:

	Very first example rule:
		now the magic number is 77.

Or `The very first example rule` would have been equivalent. This rule is
always placed first in the rulebook, and the following restrictions are
placed on it:

- A rulebook can have at most one `very first` rule.
- The `very first` rule must be declared in the same compilation unit as
the rulebook, i.e., they must both be declared in the same extension, or both
in the main source text.
- A `very first` rule cannot be unlisted with a `not listed in` sentence.
- A `very first` rule cannot be substituted with a `listed instead of` sentence.

Symmetrically, there can optionally be a `very last` rule, which is subject
to the same restrictions. Note that this is not always guaranteed to run,
however, because a rulebook might have been halted by an earlier rule.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None: this is an addition to the language which changes no existing behaviour.
