# (IE-0024) Reorganisation of standard kits

* Proposal: [IE-0024](0024-reorganisation-of-standard-kits.md)
* Discussion PR link: [#24](https://github.com/ganelson/inform-evolution/pull/24)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0013](0013-annotations-for-kit-linking.md)
* Implementation: Implemented but unreleased

## Summary

Inform projects used to link to `BasicInformKit`, `EnglishLanguageKit`, and then
either `WorldModelKit` and `CommandParserKit` or `BasicInformExtrasKit`. In this
change, `BasicInformExtrasKit` is abolished, while builds always include one of
the two new kits `Architecture16Kit` or `Architecture32Kit` according to whether
the platform is 16-bit or 32-bit.

## Motivation

The aim here is simply to make the considerable mass of kit code easier to
understand, with fewer uses of conditional compilation. `BasicInformExtrasKit`
as it stood contained quite a lot of code which almost duplicated functions in
`WorldModelKit`, while both `WorldModelKit` and `BasicInformKit` contained a great
deal of conditional compilation.

## Components affected

- [x] Very minor addition to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Major changes to runtime kits.
- [x] Very minor changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Probably none.

## Notes on the changes made

Itemising the changes would not be useful, since these are all implementation
details which should make no difference to users, but:

* `Architecture16Kit` is now linked if and only if the platform is 16-bit.
* `Architecture32Kit` is now linked if and only if the platform is 32-bit.
* `BasicInformExtrasKit` is never linked, and has been removed from the core Inform repository.
* `Architecture16Kit` and `Architecture32Kit` developed out of the old sections
called `ZMachine.i6t` and `Glulx.i6t` in `BasicInformKit`, which were entirely
wrapped inside `#ifdef TARGET_ZMACHINE` or `#ifdef TARGET_GLULX` respectively.
* `WorldModelKit` also contained similar sections, and these are also removed,
in favour of a single implementation of "state" actions such as restart,
restore, save and undo. See the new section `State.i6t`.
* This was done mostly by old-fashioned refactoring, but use of the new
replacement feature of the linker - see [IE-0013](0013-annotations-for-kit-linking.md) -
was also helpful. See in particular the new section `Startup.i6t` of `BasicInformKit`,
which gives functions used when the program starts up in a Basic Inform project:
in a non-Basic project, different functions found in `WorldModelKit` replace
those functions.
* A new architectural constant, `CHARSIZE`, is now defined whenever a kit is
being built. Analogous to `WORDSIZE`, it has the value 1 in the 16-bit architecture,
and 4 in 32-bit, where `CHARSIZE` equals `WORDSIZE`.

## New style convention for writing kit code

Conditional compilation in the standard kits used to be written as if there
were only ever two possible targets: the Z-machine VM, or the Glulx VM. This
is no longer quite true, and it is better to think of two different architectures.
Compiling to C has the same architecture as compiling to Glulx, but it is not
actually the same thing as Glulx. Because of that, conditionally compiling on
the symbols `TARGET_ZCODE` and `TARGET_GLULX` is not ideal. Instead we now
follow the following convention. It should be emphasized that these all have
the same *effect*, so the difference is just a signal to the reader of the
source code:
* Use `#Iftrue (WORDSIZE == 2);` for general economy measures needed when
we run in a 16-bit machine with limited memory - smaller arrays, less
stack space, and so on.
* Use `#Iftrue (CHARSIZE == 1);` to distinguish the version of the command
parser which writes characters into single bytes from the one which uses
words for characters, and accommodates much more of Unicode, where `CHARSIZE`
equals `WORDSIZE`.
* Use `#Ifdef TARGET_ZCODE;` as little as possible, and only when genuinely
using some feature of the Z-machine architecture not found in Glulx.

## Marking activities to omit them from RULES output

The RULES testing command traces all uses of rules, but would be swamped if
certain highly frequently called rules were logged in this way (for example,
the ones dealing with printing object names). The old implementation of
activities marked certain activity rulebooks as to be omitted from RULES:

	Grouping together something
	Issuing the response text of something
	Listing contents of something
	Printing inventory details of something
	Printing room description details of something
	Printing the name of something
	Printing the plural name of something

This set of "hidden" activities was hard-coded. This is no longer true, and
a new annotation allows an activity to be defined as being "hidden":

	Printing the plural name of something (hidden in RULES command) is an activity.

While this is probably useful only to the Basic Inform and Standard Rules
extensions, it's always better not to have hard-coded lists of exceptions.
