# (IE-0003) Dividing source text into multiple files

* Proposal: [IE-0003](0003-multiple-source-files.md)
* Discussion PR link: [#3](https://github.com/ganelson/inform-evolution/pull/3)
* Authors: Graham Nelson
* Language feature name: None
* Status: Draft
* Related proposals: None
* Implementation: Partial in v10.1

## Summary

The ability to split a large source text across multiple text files.

## Motivation

Very large projects have source text which becomes unwieldy to store in a
single file, and source control is certainly easier if it can be broken into
multiple files. On really large projects, the Inform app may not be the ideal
editor for such a file or files, either.

Some people get around this by creating bogus extensions (see for example
[the Counterfeit Monkey repository](https://github.com/i7/counterfeit-monkey)),
but that isn't really what extensions are for.

## Components affected

- [x] Minor (additive) change to the natural-language syntax.
- [x] Minor change to inbuild.
- [ ] No change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Minor change to documentation.
- [x] Potential changes to the GUI apps.

## Impact on existing projects

None.

### Implementation

The main source file can declare that the contents of a heading are to be
found in an external source file, using a new form of heading annotation.
For example:

	Chapter 7 - Into the Woods (see "woods.i7")

says that the file `Source/woods.i7` in the Materials folder contains the
source text belonging to this heading. A problem message is issued if any
content appears underneath such a heading; so e.g.:

	Chapter 7 - Into the Woods (see "woods.i7")

	The Sylvan Glade is a room.

throws a problem, as does:

	Chapter 7 - Into the Woods (see "woods.i7")

	Section 7A - Acorns

because `Section 7A` is a part of `Chapter 7`, and should therefore be in
the external file with the rest of it.

This however is fine:

	Chapter 7 - Into the Woods (see "woods.i7")

	Chapter 8 - The Spiders
	
	The Great Web is a room.

because `Chapter 8` is not subordinate to `Chapter 7`.

The file `woods.i7` must also comply with the rules:

* It must open with a heading which exactly duplicates the one referring to it,
but without the bracketed annotation(s).
* It may not contain any subheadings of equal or higher rank than the one which
referred to it.

So for example:

	Chapter 7 - Into the Woods
	
	Section 7.1 - Some Trees
	
	The Sylvan Glade is a room.
	
	Section 7.2 - Some Acorns
	
	An acorn is a kind of thing. Three acorns are in the Glade.

...would be fine: the file opens with `Chapter 7 - Into the Woods`, which
duplicate the heading referring to it, and it goes on to contain only
headings inferior to Chapter - the two Section headings.

Note: this dovetails nicely with [IE-0009](0009-dialogue-sections.md), since
dialogue sections can now be isolated as script files in quite a tidy way.

## Alternatives considered

### Contents file option

This was in fact speculatively implemented, though not documented, in v10.1,
but it didn't really seem to mesh well with natural language, and felt too
much like a C-style `#include`.

The user can create a subfolder of the `Materials` folder called `Source`. This
must contain a file called `Contents.txt` which lists the source files to read,
one per line. For example, it might read:
```
	# Source is split into several files as follows
	Athens.i7
	Asia Minor.i7
	Pergamon.i7
```
(The `#` means a comment line and is, of course, optional.) inform7 then makes
the main source text be the concatenation of the text in these files, which must
all similarly be in the `Source` subfolder of the materials folder.

### Explicit include sentences option

In this model, projects continue to have a main source text file, as now.
But this can now make what amount to C-style `#include`s:

	Include source text in "Filename".

The filename is relative to the `Source` subdirectory of the materials directory
for a project. It uses a forward slash `/`, if necessary, to indicate a descent
into further subdirectories; this will be translated as appropriate for the
host operating system. (Thus, on Windows it will be read as `\`.)

Advantages: This is close to what projects like Counterfeit Monkey already
do: it simply changes extension inclusions into these new text inclusions.

Disadvantages: `#include` leads to a potential haystack of incomprehensible
relationships in any language, and for Inform, it cuts across the headings
system in awkward ways.

## GUI app integration

On the MacOS app, it's already the case that if the user clicks on a
clickback link to a source text reference in an external file (either from
a problem message or from the index) then the app opens the file in an
editable second window.

Such clickback links look like so:

	<a href="source:/Users/gnelson/experimental/Test.materials/Source/woods.i7#line7">
