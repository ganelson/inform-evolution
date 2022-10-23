# (IE-0014) Inter names for rulebooks

* Proposal: [IE-0014](0014-inter-names-for-rulebooks.md)
* Discussion PR link: [#14](https://github.com/ganelson/inform-evolution/pull/14) 
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: [IE-0013](0013-annotations-for-kit-linking.md)
* Implementation: Implemented but unreleased

## Summary

A purely additive change to the `translates into Inter as...` sentence syntax,
allowing explicit Inter identifier names to be assigned to be rulebooks and
activities created in source text. In particular, this allows kits to access
rulebooks and activities created in extensions.

This is an incomplete draft, but what is described below is implemented on
the `master` branch of the repository. These features are not in any release
branch as yet.

## Motivation

Suppose that an extension creates:

	The psychic visibility rules is a rulebook.
	
	Psionically probing something is an activity.

And suppose that this extension uses a kit, which wants to run these. How is
the kit to refer to them, since it cannot access their natural language names,
and doesn't have any way to know their runtime ID numbers?

This exact bind occurs already with `BasicInformKit` and `WorldModelKit`, and
an extremely inelegant solution is used which involves carefully counting,
i.e., always relying on a given rulebook being the 18th to be declared. This
just about works for the built-in kits and the rulebooks guaranteed to be
declared early in Inform's run, but is too fragile for anyone else to use.

Attempts to write `GlkKit` and `DialogueKit` both ran into this issue, so
it is not entirely theoretical.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None: this change is purely additive to existing natural language syntax.

## Rulebook and activity naming

The existing sentence `X translates into Inter/I6 as Y` (the old syntax `I6`
is still allowed here, though `Inter` is now preferred) allows a variety
of language constructs to be named in the subject position `X`. This change
adds two more:

	The NAME rulebook/rules translates into Inter as "DESIRED_IDENTIFIER".

	The NAME activity translates into Inter as "DESIRED_IDENTIFIER".

For example:

	The banana rules is a rulebook.

	The banana rules translates into Inter as "BANANA_RULES".

	A banana rule:
		say "I am yellow!"

	To exhibit the behaviour:
		(- FollowRulebook(BANANA_RULES); -).

Of course, in natural language source text there are better ways, but the
point is that a kit linked into the story could also `FollowRulebook(BANANA_RULES);`.

Similarly:

	Grimly testing something is an activity.

	The grimly testing activity translates into Inter as "GRIM_ACT".

	For grimly testing:
		say "I am grimly testing."

	To exhibit the behaviour:
		(- CarryOutActivity(GRIM_ACT); -).

## Remark

The wording `...translates into Inter as...` is slippery. Different language
constructs deal with it differently. In these cases, what happens is that
a (public, thus linkable) constant with the given identifier name is defined
and equated to the internal ID. (This avoids annoying race conditions in
the compiler to do with shared variable set ID numbers.)

However, this is an implementation detail not visible to users, who get what
they want: an explicit Inter identifier evaluating to the right constant value.
