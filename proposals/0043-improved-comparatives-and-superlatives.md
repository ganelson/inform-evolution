# (IE-0043) Improved comparatives and superlatives

* Proposal: [IE-0043](0043-improved-comparatives-and-superlatives.md)
* Discussion PR link: [#43](https://github.com/ganelson/inform-evolution/pull/43)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: None
* Status: Implemented but unreleased

## Summary

Comparative and superlative forms of adjectives defined by measuring properties
have been part of Inform for many years, but worked only for properties of objects.
This proposal extends them to properties of other values (such as scenes), and
improves their typechecking.

## Motivation

Inform has long been able to read this:

	A thing has a number called the cost. The cost of a thing is usually 5.
	Definition: a thing is costly if its cost is 10 or more.

and then to form not only `costly`, but also the comparative `costlier` and
the superlative `costliest`:

	say "You are proud of [the list of costly things in the trophy case].";
	if the diamond is costlier than the Fabergé egg:
	let the treasure be the costliest thing on the table;

However, none of this worked for property-owning kinds which were not kinds
of object. This was inconsistent, and authors trying it were met with an
enigmatic mixture of compiler and runtime errors: see Jira bug report
[I7-2595 Comparatives/superlatives on a scene can lead to RTEs](https://inform7.atlassian.net/browse/I7-2595).
There seems no reason why the language should not treat all property-owning
kinds equally here.

Similarly, attempts to find the `total` of a numeric property with constructions
similar to `total cost of things on the table` would fail if, instead of being
things, the domain set contained property-owners which were not objects.

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

(a) The comparative form of an adjective becomes a relation, and one which is not
entirely simple to typecheck. Consider the condition:

	if X is costlier than Y

For this to be valid, `X` must be of a kind which can have the `cost` property(*),
but what should `Y` be? This was somewhat muddily handled in past builds, where
the typechecker allowed some slightly incorrect choices. The new answer is that
`Y` must _either_ be of a kind conforming to `X`, in this case another object,
_or_ of the kind which the property values take, in this case `number`. This
produces two different meanings:

	if the diamond is costlier than the Fabergé egg:
	if the diamond is costlier than 75:

where `X` is of kind `thing`, and `Y` is of kind `thing` in the first example,
and `number` in the second.

In past builds, in particular, `Y` was not fully checked. This, for example,
would compile:

	if the diamond is costlier than "chopped liver":

with unfortunate consequences at runtime. It is just possible that some
oddball uses of comparatives was being let through but did not lead to disaster,
and the new version of Inform may now report a problem.

(*) Technically, matters are easied slightly if `X` is an object: it's enough
for `X` to be any object if `costly` can be defined over some subset of objects,
because this test can then safely be performed at runtime.

(b) Checking of superlatives also changes, and this exposed a minor error in
the Examples as previously supplied. Consider this excerpt from the example
"Curare":

	A thing has a number called the last use.
	The last use of a thing is usually 0.
	
	Definition: a thing is old if its last use is 12 or less.

	To decide which thing is cyclically random (collection - a description of objects):
		let choice be the oldest member of the collection;
		now the last use of the choice is the turn count;
		decide on choice.

In previous builds, `oldest D` would work for any `D` of the kind `description of objects`.
The new build wants `D` to be a `description of things`, since `thing` is the
domain of the definition of `old`. So it was necessary to rewrite `Curare` as:

	To decide which thing is cyclically random (collection - a description of things):

## Details

This seems best explained by demonstration. This now compiles:

	A scene has a number called priority.
	Definition: a scene is questy if its priority is 1 or more.

	Cursed Casket Theft is a scene with priority 4.
	Temple Defilement is a scene with priority 2.
	
	When play begins:
		showme the questiest scene;
		showme the priority of the questiest scene;
		showme whether or not the Cursed Casket Theft is questier than the Temple Defilement;
		showme whether or not the Cursed Casket Theft is questier than 3;
		showme whether or not the Temple Defilement is questier than 3;
		showme the total priority of the scenes.

Previously it would not have compiled, since `scene` is not a kind of object.
The output is now:

	"questiest scene" = scene: Cursed Casket Theft
	"priority of the questiest scene" = number: 4
	"whether or not the Cursed Casket Theft is questier than the Temple Defilement" = truth state: true
	"whether or not the Cursed Casket Theft is questier than 3" = truth state: true
	"whether or not the Temple Defilement is questier than 3" = truth state: false
	"total priority of the scenes" = number: 6

Similarly:

	Length is a kind of value. 10 angstroms specifies a length.
	
	Color is a kind of value.
	The colors are red, orange, yellow, green, blue, violet.
	A color has a length called wavelength.
	The wavelength of red is 7000 angstroms.
	The wavelength of orange is 6200 angstroms.
	The wavelength of yellow is 5600 angstroms.
	The wavelength of green is 5150 angstroms.
	The wavelength of blue is 4700 angstroms.
	The wavelength of violet is 4100 angstroms.
	
	Definition: a color is warm if its wavelength is 5600 angstroms or more.
	
	When play begins:
		showme the warmest color;
		showme the wavelength of the warmest color;
		showme whether or not yellow is warmer than blue;
		showme whether or not yellow is warmer than 5000 angstroms;
		showme the list of warm colors;

which previously did not work because `color` is not a kind of object. This
now produces:

	"warmest color" = color: red
	"wavelength of the warmest color" = length: 7000 angstroms
	"whether or not yellow is warmer than blue" = truth state: true
	"whether or not yellow is warmer than 5000 angstroms" = truth state: true
	"list of warm colors" = list of colors: {red, orange, yellow}
