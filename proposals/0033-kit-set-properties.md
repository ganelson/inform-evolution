# (IE-0033) Kit-set properties

* Proposal: [IE-0033](0033-kit-set-properties.md)
* Discussion PR link: [#33](https://github.com/ganelson/inform-evolution/pull/33)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0025](0025-kit-enumerated-kinds.md)
* Implementation: Implemented but unreleased

## Summary

Extends the Neptune mini-language to allow kits to assign properties to kinds
of object which are invisible from source text, and can thus be used as part
of the private low-level implementation for features.

## Motivation

Inform already has a way to force all instances of a given kind of object to
have certain property values, in a sneak-under-the-radar way, but it is a
clumsy feature and one which currently works so crudely that it can only
work if stories are compiled via Inform 6, not via C. This is to write, say:

	Include (-
		has enterable supporter,
		with max_capacity 10,
	-) when defining a rideable animal.

This works very simplistically: when Inform code-generates an Inform 6 class
definition for the kind in question, it simply transcribes this text verbatim.
If Inform is trying to code-generate a C structure representing the kind,
the user is out of luck. In any case, this crude syntax is not really consistent
with other uses of `Include (- ... -)`, and the whole point is to express
something which is invisible from the point of view of source text: so why
is it being done in source text?

The following small feature is intended to replace the `Include ... when defining...`
syntax. Because it is only available from Neptune files, it means that only
kits can perform this dubious magic trick, though of course directory-format
extensions can now include kits. If this seems a little restrictive, that's
intentional, really: it's a feature which should only be used by kits anyway.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] Minor change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, though on subsequent releases of Inform the `Include ... when defining...`
syntax will be withdrawn, so that extension authors are advised to use the
new functionality instead.

## Details

The above example use of `Include ... when defining...` can now be replicated
with the following stanza from a Neptune file:

	properties of rideable animal {
		attribute: enterable
		attribute: supporter
		property: max_capacity = 10
	}

The stanza `properties of K { ... }` must give the name for `K`, which must
be a valid kind of object (though it can be one with no instances), and can
give the name either in the singular or the plural. Inside this stanza,
only the following are allowed:

		attribute: ATTRIBUTENAME
		attribute: ~ATTRIBUTENAME
		property: PROPERTYNAME
		property: PROPERTYNAME = NUMBER

The syntax `~ATTRIBUTENAME` means the attribute is set to `false`, not `true`;
this is traditional Inform 6 notation. The short form `property: PROPERTYNAME`
sets the property value to 0.
