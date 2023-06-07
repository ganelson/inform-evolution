# (IE-0021) No automatic plural synonyms use option

* Proposal: [IE-0021](0021-no-automatic-plural-synonyms.md)
* Discussion PR link: [#21](https://github.com/ganelson/inform-evolution/pull/21)
* Authors: Zed Lopez
* Language feature name: --
* Status: Accepted in principle
* Related proposals: --
* Implementation: In progress

## Summary

Multiple people on the IF forums have wished for the ability to suppress the
automatic understanding of things by their kind name plurals; this implements
a use option that does so.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [x] Minor changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, since this feature is entirely additive.

## Specifics

At present:

	A badger is a kind of animal. A badger is in the Night Garden.

causes the command parser to recognise BADGERS as a (plural) noun referring
to any and all objects of kind `badger`. This is often a good thing, but
does cause some authors problems: those authors would rather be explicit.
This new use option turns off the feature of Inform which provides this
service: authors can then choose their own plural names (or not, as they
please). For example:

	Use no automatic plural synonyms.

	A badger is a kind of animal.
	Understand "brocks" as a badger.
	
	A badger is in the Night Garden.
	
Here EXAMINE BADGERS would fail to parse, but EXAMINE BROCKS would work.
