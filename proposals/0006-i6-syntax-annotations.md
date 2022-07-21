# (IE-0006) New annotations for I6 syntax

* Proposal: [IE-0006](0006-i6-syntax-annotations.md)
* Authors: Graham Nelson, David Kinder, and Andrew Plotkin
* Language feature name: None
* Status: Draft
* Related proposals: [Corresponding I6 proposal](https://github.com/DavidKinder/Inform6/issues/189).
* Implementation: None

## Summary

A standard way to mark up Inform 6 syntax to provide annotations to help tools
with linking or compilation, but which do not change its basic meaning.

## Motivation

At present, many extensions need a little Inter source (written in Inform 6
syntax) to power them: either to provide low-level functionality — say, to
make obscure glk calls — or to replace some existing function in `BasicInformKit`
or `WorldModelKit` with a hacked version.

This means we see a lot of hybrid code. In a better world, I6-syntax business
could be done in a kit accompanying an extension, rather than in an extension
as such. But to fully support that, we would want kits to be able to specify
linking rules for their definitions.

For example, an extension can say:

	Include (-
	[ SquareRoot num;
		"Nobody cares about square roots, son.";
	];
	-) replacing "SquareRoot".

But a kit has no way to indicate that its definition of a `SquareRoot` function
should replace one from another kit, in the same sort of way.

Similarly, we might want some functions or variables in a kit to be private to
that kit, and not to be callable from anywhere else.

But kits are written in Inform 6 syntax, and I6 has no way to express these
ideas. So if we ever do want to develop them, we need to introduce new syntax
into the language recognised by the I6-to-Inter compiler. But if we do, we
effectively fork the I6 language.

This is therefore a joint proposal for a mutually agreed syntax for annotating
I6 directives which can be used either:

(a) By the I6-to-Inter compiler inside inform7 which processes `Include (- ... -)`
and compiles kit source, or

(b) By the real I6 compiler itself, to allow development of I6 towards some
mooted ideas such as type hints for variables.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None.

## Proposal

One possible syntax is to open a directive with "+annotation", possible with
bracketed details. For example:

	+replacing(from BasicInformKit) [ SquareRoot num;
		"Nobody cares about square roots, son.";
	];
	
	+private [ NewtonRaphsonMadness x y;
	...
	];
