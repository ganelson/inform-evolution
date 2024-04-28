# (IE-0037) Relative kinds

* Proposal: [IE-0037](0037-relative-kinds.md)
* Discussion PR link: [#37](https://github.com/ganelson/inform-evolution/pull/37)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0038](0038-division-of-time.md)
* Implementation: Implemented but unreleased

## Summary

Develops Inform's dimensional analysis for kinds to allow for cases where one
kind represents differences in another: for example, so that the difference
between two times of day can be a time period rather than another time of day.

## Motivation

In Inform's previous understanding, some numerical kinds are dimensionless,
in which case all arithmetic operations can be performed on them, and with
the result being of the same kind as the operands. `number` and `real number`
are both dimensionless in this sense. Other numerical kinds have dimension,
and certain operations on them are forbidden, or result in other kinds for
the answer. But even for dimensioned kinds, addition and scaling are always
considered possible.

For example, if we create:

	A memory address is a kind of value.
	$<hex> specifies a memory address with parts hex (four hexadecimal digits).

This is a dimensioned kind. If we try to multiply two memory addresses, we
get a problem message, unless Inform is given a suitable rule:

	A memory address times a memory address is ...

...though in this case it's hard to imagine what that could possibly mean,
and the restriction on multiplication seems reasonable.

On other other hand, we can perform `$2A00 minus $29C0`, but the answer is
the address `$0040`. Clearly that's not the address of something, so this
is conceptually all wrong: the answer shouldn't be a memory address. On
the other hand it's a perfectly sensible calculation, whose answer expresses
that the two addresses are 64 bytes apart.

We need a better way to express ideas like this, and quite a small change
to Inform's handling of dimensions is sufficient to achieve that.

The prime example of a kind which has been struggling to cope, conceptually,
is `time`, so see [IE-0038](0038-division-of-time.md) for this feature in use.

## Components affected

- [x] Small additive change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Small changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Small changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None.

## New syntax

Suppose we set up two kinds, one which will be _fiducial_, measuring some
sort of position according to some conventional scale, and the other
_relative_, in that it will hold differences between two values of the
first kind. For example:

	A memory address is a kind of value.
	$<hex> specifies a memory address with parts hex (four hexadecimal digits).

	A memory extent is a kind of value. 10 bytes specifies a memory extent.

The new ability is to write this sentence:

	A memory address minus a memory address specifies a memory extent.

Suppose we call the two kinds `F` and `R`, which in this example are `memory address`
and `memory extent`. Then:

- `F` is dimensionless. Only four arithmetic operations are possible on it:

  - An `F` minus an `F` is an `R`.

  - An `F` plus an `R` is an `F`.

  - An `F` minus an `R` is an `F`.
  
  - An `F` to the nearest `R` is an `F`.

- `R` has dimension, and is signed. All the usual arithmetic operations
  apply to it: an `R` plus an `R` is an `R`, an `R` divided by an `R` is
  a `number`, an `R` divided by a `number` is an `R` and so on.

This also affects the kind-checking used on the phrases:

	increase X by Y
	decrease X by Y

These previously required `Y` to have the same kind as `X` in order to work:
now they require it to have whatever kind can validly be added to `X`. Thus,
if `M` is a `memory address that varies`, we pass `increase M by 10 bytes`
but reject `increase M by $1200`.

For example:

	"$2A00 - $29C0" = memory extent: 64 bytes
	"$2A00 + 20 bytes" = memory address: $2A14
	"2 times 20 bytes" = memory extent: 40 bytes
	"20 bytes divided by 5 bytes" = number: 4
	"$2A12 to the nearest 16 bytes" = memory address: $2A10

Whereas all of these produce problem messages:

	20 bytes + $2A00
	$2A00 + $29C0
	2 times $2A00
	$2A00 divided by $0200
	$2A00 divided by 10
