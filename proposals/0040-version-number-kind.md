# (IE-0040) Version number kind

* Proposal: [IE-0040](0040-version-number-kind.md)
* Discussion PR link: [#40](https://github.com/ganelson/inform-evolution/pull/40)
* Authors: Graham Nelson
* Status: Draft
* Related proposals: --
* Implementation: None as yet

## Summary

Adds a `version number` kind, whose constant values can be written `v16.0`, `v2.4`
or `v2.3.12`.

## Motivation

This arose from work to make internals of Glk and Glulx more accessible to
Inform source text. A Glulx interpreter and a Glk implementation can both
report their version number, but Inform had no good kind to hold the result.
Looking ahead, we may also want to be able to refer to the version numbers
of extensions used in a project.

The main issue with this kind is what the range of representable version numbers
should be:

- To represent possible extension versions, it must be able to hold ```MAJOR.MINOR.PATCH```
versions, but does not need the full range of semver ```+...``` or ```-...```
annotations.
- ```MAJOR```, ```MINOR``` and ```PATCH``` are all non-negative integers.
- The kind does not distinguish between ```2.3.0``` and ```2.3```, nor between
```7``` and ```7.0``` and ```7.0.0```. Elided final terms are always taken as ```0```.
- Because some interpreter/website templates use the year as major version, 
```2024.2``` is also plausible, so ```MAJOR``` needs to range at least to ```2100```.
- ```MINOR``` versions aren't currently used much in the Inform ecosystem, but
it's not unknown for these to reach ```100``` in the wider software world.
- Because legacy extensions were given serial numbers as well as release numbers,
e.g., ```3/130423```, and we interpret this as the semver ```3.0.130423```, the
```PATCH``` value needs to range at least to ```250000``` and ideally ```999999```.

It follows that:

- This range is too large for a 32-bit word: 2100 times 100 times 250000 is 52500000000,
about twelve times too big to fit.
- The simplest solution is therefore to store version numbers as what amounts
to a C struct of three words.
- ```PATCH``` values cannot fit in a 16-bit word, so even that wouldn't work
on a 16-bit architecture.

It's not impossible to imagine ways to manage this on a 16-bit architecture
such as the Z-machine, but the simpler solution is to restrict `version number`
to 32-bit architectures, where it's more likely to be useful anyway. For now,
then, this kind is defined in ```Architecture32Kit``` and not ```BasicInformKit```.

## Components affected

- [x] Additive change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor additions to runtime kits.
- [x] Minor additions to the Standard Rules and Basic Inform.
- [x] Minor additions to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

A project which defines `version number` or which defines a notation which
overlaps with `vN`, `vN.M` or `vN.M.L`, may have namespace clashes.

## Details

On 32-bit architectures (i.e., Glulx, C, C#) only, a new kind `version number`
exists. Constants are written as in these examples:

	v10.0
	v4.3
	v6.71.2129

Note that literal version numbers must be written with at least one dot, so
that `v2` is not a legal literal: `v2.0` must be written instead. (This is to
reduce confusion with variables innocently named `v2` or similar.)

The values must be literal decimal numbers in the range `0` to `999999999` (nine nines).
The following are all true:

	v0.0 is v0.0.0
	v10.0 is v10.0.0
	v4.3 is v4.3.0

Three phrases can be used to extract the numbers:

	major version of (V - version number)
	minor version of (V - version number)
	patch version of (V - version number)

So for example `minor version of 4.56.2` evaluates to `56` (a `number`).

Version numbers are sorted lexicographically: major first, then minor, then patch. Thus:

	v4.0 is greater than v3.0
	v3.0 is greater than v2.99
	v2.3 is greater than v2.2.17

For example, if we declare:

	Compatibility relates a version number (called V1) to a version number (called V2)
	when the major version of V1 is the major version of V2 and V1 is at least V2.
	
	The verb to be compatible with means the compatibility relation.

then we have a relation which tests semver compatibility:

	"whether or not v0.7.5 is compatible with v0.2" = truth state: true
	"whether or not v0.7.5 is compatible with v0.7.63" = truth state: false
	"whether or not v0.7.5 is compatible with v1.7" = truth state: false

The three phrases:

	glk version number
	glulx version number
	interpreter version number

(which also exist only on 32-bit architectures) have all been rewritten to
return `version number` values, not `number` values.
