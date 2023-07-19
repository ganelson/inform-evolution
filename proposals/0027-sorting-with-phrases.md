# (IE-0027) Sorting with custom comparison phrases

* Proposal: [IE-0027](0027-sorting-with-phrases.md)
* Discussion PR link: [#27](https://github.com/ganelson/inform-evolution/pull/27)
* Author: Dannii Willis
* Language feature name: None
* Status: In progress
* Related proposals: None

## Summary

Add phrases that allow lists and tables to be sorted with custom comparison
phrases.

## Motivation

Most other languages allow arrays to be sorted with a specified comparison
function. This functionality has been missing in Inform 7.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor change to BasicInformKit.
- [x] Minor changes Basic Inform.
- [x] Minor addition to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, since this feature is entirely additive.

## Specifics

The new phrases are:

- sort (list of values of kind K) with (phrase (K, K) -> number)
- sort (table name) with (phrase (table name, number, number) -> number)

The list phrase functions just as other common languages do (for example, see
[C#](https://learn.microsoft.com/en-us/dotnet/api/system.collections.icomparer.compare?view=net-7.0#system-collections-icomparer-compare(system-object-system-object)),
[Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)).
The phrase takes two list elements and returns a number indicating the relative
order of the elements: negative if the left value should come before the right,
zero if they are equal, and positive if the left value should come after the right.

Since there is no kind representing an entire table row, to sort a table we pass
the table name and two row numbers. The comparison phrase can then access any
columns it wishes in order to determine the relative order of the two rows. Any
completely blank rows will be put at the bottom, just as with the existing sorting
phrases.