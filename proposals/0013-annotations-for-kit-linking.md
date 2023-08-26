# (IE-0013) Annotations for kit linking

* Proposal: [IE-0013](0013-annotations-for-kit-linking.md)
* Discussion PR link: [#13](https://github.com/ganelson/inform-evolution/pull/13) 
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0006](0006-i6-syntax-annotations.md), [IE-0018](0018-use-options-and-kit-configuration.md), [IE-0024](0024-reorganisation-of-standard-kits.md)
* Implementation: Partly implemented but unreleased

## Summary

A simple set of annotations, in the sense of [IE-0006](0006-i6-syntax-annotations.md),
so that directives in the source code for kits or in `Include (- ... -)` code
can be marked up in ways which affect how they are linked.

This is an incomplete draft, but what is described below is implemented on
the `master` branch of the repository. These features are not in any release
branch as yet.

The new test cases `Namespaces` and `Replacement` exercise the following.

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

## +private and +public

Most directives in Inform 6 syntax create something and give it an identifier,
that is, a name. For example, a function declaration such as:

	[ SquareRoot num;
		...
	];

is a single directive, making a function with the identifier `SquareRoot`.
Similarly, for example:

	Constant BEAST_ID = 666;
	Global current_horsemen_count = 4;

Identifiers can be either "private" or "public" from the linker's point of
view. This only affects whether code in one compilation unit can refer to an
identifier defined in another one. (A compilation unit is either the source
text with its `(- ... -)` fragments, or a kit). Public means it can, private
means it cannot. So for example, if:

	+private [ Startup;
		"Only I can use this.";
	];

is defined in `ExampleKit` then `Startup` can be accessed only from that kit.
This allows a kit to "hide" its implementation details so that other kits, and
the source text, cannot get access to them.

And similarly for `+public`. Note that Inform throws a problem message if
these markers are used redundantly - for example, if a function is marked
`+public` in a context where everything is public by default anyway. (So
`+public` is only useful if `+namespace` has been used with `access private`
specified - see below.)

## +namespace

All material compiled in kits or in `Include (- ... -)` is considered as
belonging to some "namespace". Where none is declared, this will be `main`,
the global namespace.

The annotation:

	+namespace(MyKit);

which must occur on its own (note the semicolon `;` rather than any directive
following on from it) declares that the directives which follow all belong to
the namespace `MyKit`. This continues to the end of the inclusion or kit,
or to the next `+namespace` marker, whichever comes first.

Within a namespace, all identifiers are implicitly prefixed with the namespace
name and a backtick (except for `main`, where identifiers are as written). Thus:

	+namespace(Secret);
	[ Function x;
		...
	];
	Constant z = 2;
	+namespace(main);
	Constant z = 1;

creates the identifiers ``Secret`Function``, ``Secret`z``, and `z`. The two
definitions of what looks like `z` are therefore not contradictory, because
they define two different things.

With a namespace other than `main`, writing an unqualified identifier will
implicitly mean "the definition in this namespace if there is one, and the
definition in `main` if not". For example:

	+namespace(Secret);
	[ Example;
		Hello(z);           ! Calls Hello in the namespace Secret with argument 2 
		main`Example();     ! Calls Example in the namespace main
	];
	Constant z = 2;

	[ Hello x;
		print "Hello to ", x, ".^";
	];

	+namespace(main);
	Constant z = 7;
	[ Example;
		Secret`Hello(z);    ! Calls Hello in the namespace Secret with argument 7
		Hello(13);          ! An error: there is no Hello in the namespace main
	];

Namespaces cannot be nested and can be reopened at any point. If two different
kits both use a namespace called `SoundEffects`, then it's the same namespace.

Optionally, `+namespace` can specify a default access. For example:

	+namespace(Secret, access private);

puts us in namespace `Secret`, and says that until the next namespace marker,
all definitions are by default private, as if they were all marked `+private`.
Similarly for `access public`, though that is the default.

Note that it's fine to write something like:

	+namespace(main, access public);
	
	[ MyAPIFunction;
		MyImplementation(1745);
	];

	+namespace(main, access private);
	
	[ MyImplementation undocumented_number;
		...
	];

thus dividing a kit into a public half and a private half.

The namespace `replaced` is reserved for the linker's own use (see below) and
attempting to create anything in it will throw an error.

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

* If no clash ever occurs, no error is produced. The `+replacing` definition
may not in fact have replaced anything, but it's still the valid one. So
it's possible to mark a definition as `+replacing` which would overlap a
definition in some other kit which might or might not be being used.
* It is not possible to use `+replacing` to override a `+private` definition
in another compilation unit.
* By default, the replaced definition is destroyed, but see below.

### +replacing(keeping)

By default, when `+replacing` is used, the replaced declaration does not exist
anywhere any more. It has been struck out of the Inter tree, and is gone.

However, the optional parameter `keeping` can be used to ensure that the
original is retained after all. It clearly cannot have the same name as before,
and so is moved into the namespace `replaced`.

For example, this effectively says "double the value of `REQUISITION_STACK_SIZE`,
whatever that is":

	+replacing(keeping) Constant REQUISITION_STACK_SIZE = (2 * replaced`REQUISITION_STACK_SIZE);

Whereas this would produce an error:

	+replacing Constant REQUISITION_STACK_SIZE = (2 * replaced`REQUISITION_STACK_SIZE);

since then ``replaced`REQUISITION_STACK_SIZE`` would not exist: the old definition
would have been destroyed, and so could not have been used.

Or for another example, the following "corrects" the output of the existing
definition of `SquareRoot` from `BasicInformKit` by rounding up to the nearest
natural number, rather than down:

	+replacing(keeping) [ SquareRoot num x;
		x = replaced`SquareRoot(num);
		if (x*x < num) x++;
		return x;
	];

so that `the square root of 17` will then evaluate to 5, not 4.

`+replacing(keeping)` can only be applied to global variables (`Global ...`),
constants, arrays and routines: an error will be thrown for attempts to use
it with other constructs.
