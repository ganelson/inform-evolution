# (IE-0003) Dividing source text into multiple files

* Proposal: [IE-0003](0003-multiple-source-files.md)
* Reference link: [#3](https://github.com/ganelson/inform-evolution/pull/3)
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

- [ ] No change to the natural-language syntax.
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

## Options

Three ways to do this have been proposed. At present, we lean towards option 3.

### Contents file option 1

This is in fact implemented, though not documented, in v10.1. It will be removed
if we go for the other options.

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

Advantages: This restricts the splitting-up straightforwardly to a concatenation,
and doesn't get into an elaborate tree of files including each other.

Disadvantages: This looks very odd in the GUI apps, since the main source text
is not being used, but is still presented by the apps just as if it were.

### Inclusion option 2

In this model, projects continue to have a main source text file, as now.
But this can now make what amount to C-style `#include`s.

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

### Externalised headings option 3

External source files must be tied to headings in the source text. Thus:

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
the external file with the rest of it. This however is fine:

	Chapter 7 - Into the Woods (see "woods.i7")

	Chapter 8 - The Spiders
	
	The Great Web is a room.

because `Chapter 8` is not subordinate to `Chapter 7`.

Advantages: This makes text file inclusion coherent with the headings
structure of a large source text automatically, and dovetails nicely with
[IE-0009](0009-dialogue-sections.md), since dialogue sections could then
be isolated as script files in quite a tidy way.

## GUI app integration

For the GUI apps, this proposal presents two challenges, either or both of
which they may want to ignore.

1. Should the user be allowed to edit files stored in `Source` from the
Materials folder? If so, what affordances should the apps offer for browsing
and choosing those?

2. How should we handle clickback links used in problem messages and the
Index? (The familiar little orange discs with arrows in.) At present, these
use special URL protocols, as in:
```
	<a href="source:story.ni#line16">(button)</a>
	<a href="source:/Users/gnelson/Inform.app/Contents/Resources/Internal/Extensions/Graham Nelson/Standard Rules.i7x#line544">(button)</a>
```
That is, the file path is relative to the `Source` subdirectory of the project
directory (_not_ the materials directory), unless a full file system path is
given. How should a clickback link to an external source file be done:
for example, would it be better to use something like this to make a clickback
to line 16 of `woods.i7`?
```
	<a href="source:external/woods.i7#line16">(button)</a>
```

