# (IE-0014) Inter names for rulebooks

* Proposal: [IE-0014](0014-inter-names-for-rulebooks.md)
* Discussion PR link: [#14](https://github.com/ganelson/inform-evolution/pull/14)  
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0013](0013-annotations-for-kit-linking.md)
* Implementation: Implemented but unreleased

## Summary

This change deprecates the `translates into Inter as...` sentence syntax,
in favour of two different syntaxes which offer a wider range of functionality.
In particular, this allows kits to access rulebooks and activities created in
extensions.

This is an incomplete draft, but what is described below is implemented on
the `master` branch of the repository. These features are not in any release
branch as yet.

## Motivation

The existing sentence syntax `X translates into Inter as Y` (or equivalently
`X translates into I6 as Y`) allows users to specify how natural language
names for language constructs should match up with identifier names in the
compiled output. Or at least, that's how the sentence was originally envisaged.
In practice, it had two subtly different meanings:

(1) Sometimes it meant "X is created by the source text, but I want to refer
to it in Inter code as well, calling it Y."

(2) But sometimes it meant "Y is created by Inter code, for example in a kit,
but I want to refer to it in source text as well, calling it X."

These go in opposite directions, which makes it slippery to understand
exactly what the sentence does. Prior to this proposal there were six possible
uses of the sentence - two with meaning (1), four with meaning (2):

* The construct is an object or kind of object. Meaning (1).
* The construct is an action. Meaning (1).
* The construct is a global variable. Meaning (2).
* The construct is a property. Meaning (2).
* The construct is a rule. Meaning (2).
* The construct is an Understand token. Meaning (2).

It seems better to use clearer wording which distinguishes these meanings
better, not least because we could then make it possible for the user to
choose which is meant in individual cases.

Also, the list does not currently include rulebooks or activities, and that
causes some nuisance already. Suppose that an extension creates:

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

This proposal therefore also extends the range of language constructs to
which these sentences can apply.

## Components affected

- [x] Minor change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [x] Minor changes to the Standard Rules and Basic Inform.
- [x] Minor change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

The syntax `translates into Inter as` is being deprecated. All existing uses
of it will continue to work for now, but the next major version of Inform (i.e.,
v11) will throw a problem message if this deprecated sentence form is used.
So this is a good time for extensions to migrate to the new syntax.

Note that `translates into Inter as` will already throw a problem message if used
with the new use-cases below (such as rulebooks and activities). But since these
were not previously possible, that breaks no existing syntax.

## New sentence syntaxes

Instead of using `translates into Inter as` for both meanings (1) and (2) above,
under this proposal `is accessible to Inter as` is used for meaning (1), and
`is defined by Inter as` is used for meaning (2). For example:

	The outside object is accessible to Inter as "out_obj".
	The taking action is accessible to Inter as "Take".

	The time of day variable is defined by Inter as "the_time".
	The printed name property is defined by Inter as "short_name".
	The initialise memory rule is defined by Inter as "INITIALISE_MEMORY_R".
	The understand token a time period is defined by Inter as "RELATIVE_TIME_TOKEN".
	The Table of Fizzy Drinks is accessible to Inter as "TABLE_FIZZ".
	The niceness column table column is accessible to Inter as "NICENESS_COLUMN".

The Inform examples, built-in extensions and test cases have all had their
usages of `translates into Inter as` replaced with the appropriate syntax in
each case.

## Rulebook and activity naming

Rulebooks and activities are not Inter concepts, and can only be defined by
natural language source text. So it's clear which syntax must be used:

	The NAME rulebook/rules is accessible to Inter as "DESIRED_IDENTIFIER".

	The NAME activity is accessible to Inter as "DESIRED_IDENTIFIER".

For example:

	The banana rules is a rulebook.

	The banana rules is accessible to Inter as "BANANA_RULES".

	A banana rule:
		say "I am yellow!"

	To exhibit the behaviour:
		(- FollowRulebook(BANANA_RULES); -).

Of course, in natural language source text there are better ways, but the
point is that a kit linked into the story could also `FollowRulebook(BANANA_RULES);`.

Similarly:

	Grimly testing something is an activity.

	The grimly testing activity is accessible to Inter as "GRIM_ACT".

	For grimly testing:
		say "I am grimly testing."

	To exhibit the behaviour:
		(- CarryOutActivity(GRIM_ACT); -).

## Accessing properties by name

The existing sentence:

	The printed name property is defined by Inter as "short_name".

can only be used for properties of objects, and tells the compiler that a
kit will define the property with a `Property` or `Attribute` directive.

This is not helpful if, for example, we want to set up a property for a kind
of value which is not an object, and then refer to that from a kit. For
example, suppose the source text says:

	A dialogue beat can be performed or unperformed. A dialogue beat is
	usually unperformed.

where `dialogue beat` is a kind, but not a kind of object. Before this
proposal, there was no way for Inter code to read or write the `performed`
property. But it now can:

	The performed property is accessible to Inter as "performed".

Note that this is "accessible to", not "defined by" Inter, since the
property is entirely created by the compiler from the natural language
source text. Code in some kit can now write, say:

	WriteGProperty(DIALOGUE_BEAT_TY, db, performed, true);

to set the "performed" property for the beat `db`.

## A note about kinds of object

Work done for this proposal also fixed Jira bug I7-2225, "Translating kinds
into I6 doesn't work". This now works as expected:

	A fruit is a kind of thing.
	The fruit kind is accessible to Inter as "K_fruit".

The Inter class for fruit is called `K_fruit`. Moreover, Inter identifiers
derived from this one make parallel translations: thus `K_fruit_Count` is
the constant for the number of instances, `K_fruit_First` is the first
instance, and `K_fruit_Next` the property for moving forward in the linked
list of fruit instances. Without the above `...is accessible to...` sentence,
these would have been called something unpredictable like `K13_fruit`,
`K13_fruit_Count` and so on -- the unpredictable part being the number 13.
