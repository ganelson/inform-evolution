# (IE-0007) Double-precision real numbers

* Proposal: [IE-0007](0007-double-precision-reals.md)
* Authors: Andrew Plotkin and Graham Nelson
* Language feature name: None
* Status: Draft
* Related proposals: None
* Implementation: None

## Summary

Quietly double the precision of real-number arithmetic used in Inform projects,
without the user needing to do anything to make this happen.

## Motivation

Inform's existing `real number` kind is, in C terminology, `float` rather than
`double` — that is, it stores real numbers as single-precision 32-bit values.

While these are fast to read and write, the lack of precision in them is a
little painful. Single-precision provides (a little more than) 6 significant
figures of accuracy, whereas double-precision provides at least 15. While
Inform is not a language used for scientific work, it does have rather nice
features for handling dimensions, SI units, and so on, and it seems a pity
for rounding errors and so on to make it unable to solve physics homework
problems accurately enough to hand in.

More significantly for Inform users, probably, is that single-precision reals
can only represent integers exactly in the range −16,777,216 to 16,777,216: thus,
if an Inform `number` is cast to an Inform `real number` and then back again,
it will only be approximately equal to the original when outside that range.
By contrast, a double-precision real can hold any Inform `number` exactly, and
indeed can hold any integer up to 9,007,199,254,740,992.

For example, with the present implementation of real numbers:

	"N" = number: 1000000000
	"1.333 times N to the nearest whole number" = number: 1332999936
	"2 times the arcsine of 1" = real number: 3.14159

Better accuracy can only be bought by giving up some speed. Trigonometry is slower
for doubles, for example, and for Inform, double values not fitting into a single
VM word incurs an overhead. However, Inform is not a language where speed in handling
numerical data is likely to be important. Following the general rule nowadays
that double should always be used over float whenever speed is unimportant,
Inform should take this trade-off, and use double if it can.

We can only do this if double-precision arithmetic can actually be used on the
platforms we are compiling to. For the Z-machine it cannot, but then that was
already true for single-precision. For Glulx, the necessary opcodes were added
as version 3.1.3 of the Glulx specification (2022), with the corresponding
support added to Inform 6 in version 6.40 (2022). For C, of course, `double`
has always been available as an alternative to `float`.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Major changes to BasicInformKit; no change to other runtime kits.
- [x] Major changes to Basic Inform; no change to the Standard Rules.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Except that more accurate calculations may slightly change the output of
existing projects, we do not expect any adverse impact.

## Options considered

1. Make the existing `real number` kind double-precision instead of single.

2. Add a new `double real number` kind, leaving the current `real number`
as single-precision.

3. Make the existing `real number` kind double-precision, but add a new
`single real number` kind for single-precision too.

Option 3 has no real advantage over the others. Option 1 is simpler for users
and much easier to understand, but just might skew the behaviour of existing
projects using real numbers. Option 2 preserves existing behaviour for all
projects, but at the cost of confusion for users (why are there two kinds for
the same thing?) and extra compiler complexity (for promotions between the two
real kinds, not to mention the ambiguous kind of a constant like `3.1415`).

The proposal is therefore for Option 1.

## Proposal

`real number` will become a block value. The short block will always have
extent 2, providing 64 bits of data. This will be exactly the Glulx VM 3.1.3
format of two-word array. In particular, there will be no long block, and so
real number calculations will never use or need the heap.

The `Mathematics.i6t` section of `BasicInformKit` will be rewritten to use
the new double opcodes. For example,

	[ REAL_NUMBER_TY_Sin in out; @sin in out; return out; ];

will become:

	[ REAL_NUMBER_TY_Sin in out;
		@dload in sp sp;
		@dsin sp sp sp sp; 
		@dstore out sp sp;
		return out;
	];
