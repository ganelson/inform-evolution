# (IE-0028) Extension documentation revisited

* Proposal: [IE-0028](0028-extension-documentation-revisited.md)
* Discussion PR link: [#28](https://github.com/ganelson/inform-evolution/pull/28)
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: [IE-0001](0001-extensions-with-resources.md), [IE-0017](0017-apps-and-extensions.md)
* Implementation: In progress

## Summary

To make extension documentation richer and more visually comprehensible.

## Motivation

This is a spin-off from work done for [IE-0001](0001-extensions-with-resources.md)
and [IE-0017](0017-apps-and-extensions.md), but really improvements were long overdue.

Extensions have always been able to provide documentation, by placing it below
a sort of tear-off line of syntax at the bottom of the extension file. Thus:

	...
	
	Hypothetical Extension ends here.
	
	---- DOCUMENTATION ----
	
	This is a neat extension. It...

Documentation can now instead be provided as a stand-alone file for
directory-stored extensions: see [IE-0001](0001-extensions-with-resources.md).
But the not-very-elegant tear-off syntax remains legal for single-file extensions.

However, this proposal is not about where the marked-up source for the documentation
comes from, but about what `inbuild` then does with it, and that was also quite
deficient in a number of ways:
* Extension text was read through the Inform lexer, as if it were source text,
which occasionally produced bizarre bugs or peculiar layout choices.
* All the extension documentation was jammed together onto a single HTML page.
* There was no syntax-colouring on code samples.
* Display boxes for phrase definitions, a feature of the main manual, could
not be placed in extension documentation.
* Paste buttons in examples were allowed, but could not be broken up with
commentary, as they can in examples in the Inform manual. In practice this
led to examples being written with very little commentary, which isn't ideal.
* The HTML generated was frequently sloppy in mis-matching tags, and was
sometimes unreadable in dark mode.

All of this has been addressed.

## Components affected

- [ ] No change to the natural-language syntax.
- [x] Major changes to inbuild.
- [ ] No change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to BasicInformKit.
- [ ] No change to Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None.

## Markup syntax

The markup syntax is simple enough that it seems easiest to describe it in full,
and just note where something is new.

(1) Trailing space is ignored. Any leading space is converted to an indentation depth,
in units of one tab equals four spaces, rounding down. Thus if a line begins with
a tab, or begins with four spaces, it is considered indented by one tab stop.

(2) Unindented lines in certain formats are headings:

	Chapter - Title
	Chapter: Title
	Section - Title
	Section: Title
	Example: *** Title
	Example - *** Title

In the case of example headings, any number from 1 to 4 asterisks can appear.
There is no distinction between the use of a hyphen or colon.

A chapter contains all material beneath it up to the next chapter heading, or
to the end of the file. A section ceases if either another section or a new
chapter appears, and an example ceases at the next heading of whatever kind.
`inbuild` is very flexible: authors can have no headings at all, or can use
just sections, or just chapters, or just examples, for instance.

NEW: `inbuild` now generates one HTML page for each chapter, along
with a title page for the extension. (If the extension contains matter before
the first chapter heading, this is considered introductory and appears on the
title page: if there are no chapter headings, everything appears on the title
page, which is the only page.) In addition, each example generates its own
HTML page. An extension with 6 documentation chapters and 11 examples therefore
produces a miniature website of 18 HTML pages.

(3) Indented lines are code examples.

NEW: These are now syntax-coloured. Of course, different displayed pieces of
code can have different meanings, and will not always be Inform source text.
`inbuild` therefore has a concept of the current "language" being coloured;
by default, this is Inform. Once set, it remains until the end of the current
section, chapter, example or definition, whichever comes first.

Whereas a typical code example looks like this:

	This is a regular example.
	
		Example line 1 here, indented.
		And line 2 here.
	
	The running text resumes.

...it is now also legal to give a special first line of the example which
changes the colouring language:

	This example shows output rather than source code:
	
		(transcript)
		> GET PYRAMID
		The Great Pyramid of Giza is slightly oversized for your knapsack.
	
	And once again the running text resumes, but the next will continue:
	
		> CLIMB PYRAMID
		How easy that is to type.

`inbuild` currently recognises four "languages": `Inform`, the default;
`Inform 6`; `transcript`; and just plain `text`, where no syntax is coloured
at all.

(4) If the opening line of a code example is marked with a `*: `, then `inbuild`
places a paste button there, which if clicked in the Inform app will paste
the content of the code into the Source panel.

NEW: If the next code example is marked `**: `, then this is considered a continuation
of the code to be pasted. For example:

	Try this:
	
		*: "The Heat-Death of the Universe"
	
	An overly dramatic title, but never mind.
	
		**. The Wyndham Theatre is a room.

causes the paste button to appear just once, but to have content merging the
two code samples together. (There can be any number of such, but they have to
be consecutive.)

(5) NEW: An unindented line which begins `{defn}` or `{defn TAG}` begins a
phrase description, which continues until a line which reads just `{end}`.
These use the exact same syntax followed by the `Writing with Inform` source,
so e.g.:

	```
	{defn ph_nearestwholenumber}(real number) to the nearest whole number ... number
	This phrase performs signed addition on the given values, whose kinds must
	agree, and produces the result. Examples:
	
		1.4 to the nearest whole number = 1
		1.6 to the nearest whole number = 2
		-1.6 to the nearest whole number = -2
	
	We probably ought to bear in mind that the limited range of "number" means
	that the nearest whole number might not be all that near. For example:
	
		6 x 10^23 to the nearest whole number = 2147483647
	
	because 2147483647 is the highest value a "number" can have.
	{end}
	```

The `TAG` is in theory for use in the Index to correlate phrases with their
documentation, but for now this does nothing with extension documentation.

Phrase descriptions cannot be nested, and cannot contain headings.

## Generating documentation outside of Inform

`inbuild` can work alone to generate little documentation websites from
extensions:

	$ inbuild -document EXTENSION -document-to PATH

where `EXTENSION` is the filename or directory name for the extension, and
`PATH` the directory to write the mini-website into, which must exist.

Free-standing documentation in the above markup syntax can also be converted:

	$ inbuild -document-from FILE -document-to PATH

## Syntax errors in documentation markup

The documentation generator aims to fail safe, and not to force projects to
fail to compile just because they use an extension with broken documentation.
Producing error messages in the conventional way would be inappropriate.

Instead, when `inbuild` does find a clear error, it now renders this as a red
error message box at the relevant position.

In fact, there are relatively few markup errors. All sequences of chapter
headings, section headings and examples are legal, for example.
