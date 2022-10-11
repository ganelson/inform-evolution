# (IE-0013) Annotations for kit linking

* Proposal: [IE-0013](0013-annotations-for-kit-linking.md)
* Discussion PR link: [#13](https://github.com/ganelson/inform-evolution/pull/13) 
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: [IE-0006](0006-i6-syntax-annotations.md)
* Implementation: Partly implemented but unreleased

## Summary

A simple set of annotations, in the sense of [IE-0006](0006-i6-syntax-annotations.md),
so that directives in the source code for kits or in `Include (- ... -)` code
can be marked up in ways which affect how they are linked.

This is an incomplete draft, but what is described below is implemented on
the `master` branch of the repository. These features are not in any release
branch as yet.

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

More generally, kits have up to now "exported" really all of their identifiers,
and since we currently have what's almost a single global namespace, there's a
lot of potential for rival kits to have overlapping names.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No change to inform7.
- [x] Changes to inter, enabling it to read annotations.
- [x] Changes to the Inter specification, enabling it to store annotations.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, except that kits will need to be recompiled because of the change of
Inter bytecode version number, which advances from 2 to 3.

## +private

Most directives in Inform 6 syntax create something and give it an identifier,
that is, a name. For example, a function declaration such as:

	[ SquareRoot num;
		...
	];

is a single directive, making a function with the identifier `SquareRoot`.
Similarly, for example:

	Constant BEAST_ID = 666;
	Global current_horsemen_count = 4;

The Inform 6 language does not have namespaces in the modern programming
language sense, so all three of these identifiers become valid everywhere
in the program containing them.

That was fine for Inform 6, where entire programs were written by one
person at the same time, but now consider the way kits (though also written
in Inform 6 syntax) are written. Suppose two different kits each want to have
a function called, say, `Startup`? They cannot both claim the same name.
These kits may be written by different people at different times, who cannot
come to some private arrangement.

One way round this is to use `+private`. This marks an identifier so that
it can be used only in the current compilation unit (either the main source
text with its `(- ... -)` fragments, or a kit). For example:

	+private [ Startup;
		"Only I can use this.";
	];

This `Startup` function can be used freely inside the kit, but not from any
other kit or the main source text. (It cannot even be replaced.) To the
linker, it is invisible, and is part of the hidden workings. If another
compilation unit also defines something called `Startup` (public or private)
then there is no clash: each definition applies independently where it is
valid. A clash occurs only where an identifier is public in two different
compilation units.

`+private` can thus be used either for encapsulation (i.e., marking code
as being implementation which users should not have access to) or to
minimise the usage of the global namespace claimed by a kit. In other words,
a kit author could use `+private` on every definition not intended to be
seen by users.

(An alternative would be to make this the default, and have `+public` or
`+export` or similar to mark those names which the kit author does want to
expose. But in practice the big standard kits such as `BasicInformKit`
do want to expose a great many names, so that would mean a huge amount of
change.)

## +replacing

When the linker joins compilation units together, it looks for name clashes.
Note that use of `+private` (see above) may avoid such clashes, but suppose
one occurs. The obvious thing for the linker to do is to print an error
message, and indeed that's the default.

However, as noted above, the linker makes a "replacement" in response to
something like:

	Include (-
	[ SquareRoot num;
		"Nobody cares about square roots, son.";
	];
	-) replacing "SquareRoot".

This is a situation where both the source text and `BasicInformKit` are
defining functions called `SquareRoot`, but the "replacing" instruction tells
Inform to tell the linker to resolve the clash by using the source text
definition and throwing away the `BasicInformKit` one.

This is a clumsy solution, entered Inform only in 10.1.0, and should probably
already be deprecated in favour of:

	Include (-
	+replacing [ SquareRoot num;
		"Nobody cares about square roots, son.";
	];
	-).

The annotation `+replacing` is better because it applies to a single specific
declaration, and also because it can be used in kits as well as in the main
source text. Thus `MathKit` might declare

	+replacing [ SquareRoot num;
		...
	];

to provide some different algorithm for computing square roots, for example.

`+replacing` means "let this definition triumph over any other". If the linker
finds two definitions both marked as `+replacing` in this way, there's once
again an error message, since they can't both win.

Optionally, the annotation can be more specific:

	+replacing(from BasicInformKit) [ SquareRoot num;
		...
	];

allows this definition to replace the one in `BasicInformKit`, but continue
to throw a linking error if an unexpected rival appears from elsewhere.

Replacement is a more slippery idea than it first seems. Some things to bear
in mind:

* The replaced declaration does not exist anywhere. It hasn't simply been
renamed: it's gone.
* If no clash ever occurs, no error is produced. The `+replacing` definition
may not in fact have replaced anything, but it's still the valid one. So
it's possible to mark a definition as `+replacing` which would overlap a
definition in some other kit which might or might not be being used.
* It is not possible to use `+replacing` to override a `+private` definition
in another compilation unit.
