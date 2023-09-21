# (IE-0028) Extension documentation revisited

* Proposal: [IE-0028](0028-extension-documentation-revisited.md)
* Discussion PR link: [#28](https://github.com/ganelson/inform-evolution/pull/28)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0001](0001-extensions-with-resources.md), [IE-0017](0017-apps-and-extensions.md), [IE-0030](0030-extension-examples-and-testing.md)
* Implementation: In progress

## Summary

To make extension documentation richer and more visually comprehensible, by
incorporating a Markdown parser into `inbuild`. To remove the `indoc` tool
and use the same documentation-generator to build the two big language manuals
inside the Inform apps, and on the Inform website.

## Motivation

This is a spin-off from work done for [IE-0001](0001-extensions-with-resources.md)
and [IE-0017](0017-apps-and-extensions.md), but really improvements were long overdue.

Extensions have always been able to provide documentation, by placing it below
a sort of tear-off line of syntax at the bottom of the extension file. Thus:

	...
	
	Hypothetical Extension ends here.
	
	---- DOCUMENTATION ----
	
	This is a neat extension. It...

Documentation can now instead be provided as a stand-alone file, `Documentation.md`,
for directory-stored extensions: see [IE-0001](0001-extensions-with-resources.md).
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

Extension documentation is marked up with the widely-used Markdown markup notation.
Numerous slightly varying dialects of Markdown exist, and Inform extensions use
"Inform-flavoured Markdown". This is as standard as we can make it, and includes:
* All CommonMark features, i.e.: emphasis, strong emphasis, backticked code snippets,
links, images, web autolinks, email autolinks, block quotes, ordered lists,
unordered lists, thematic breaks, indented code samples, fenced code samples
with optional info strings, ATX headings (i.e. using `#` characters), setext
underlined headings (i.e. using a second line of `----` characters), backslashed
escapes, and HTML entities; *except that*
* For security reasons, raw HTML blocks are not passed through as HTML,
and similarly raw HTML tags are not allowed. So `this is <b>bold</b>`
does not place the word "bold" in boldface, and instead passes the angle-bracket
notations through into the resulting text.
* All GitHub-flavored Markdown's extensions to CommonMark: strikethrough, tables,
task list items and extended autolinks.
* A further extension so that Example, Chapter and Section headings, as used in
past versions of Inform extension documentation, continue to be understood.
* A further extension to allow indexing: see below.

There are many guides to Markdown in existence already, but these examples
give the general idea:

	# Introduction

	## Purpose

	The "Philately" extension provides ways to manage stamp collections.
	Contact <stampbuff@gmail.com> with any bug reports.

	## Collecting

	A [rare stamp](chapter3.html) marked with a curly H, resembling the
	mathematical symbol &HilbertSpace;, recently ~~went on sale~~ was sold for **$712.30**.
	(_All prices here are estimates._ See https://www.stanleygibbons.com.)
	We would define this in Inform with `The Harding Memorial 1923 Issue is a stamp.`
	Or, for example:

		The Harding Memorial 1923 Issue is a stamp. The price of the Harding
		Memorial is $712.30.

	Roger S. Brody notes:

	> Some Harding rotary press-printed stamps were perforated gauge 11 x 11
	> on the flat-plate equipment instead of the normal 10 x 10 rotary
	> perforating machine, producing an important twentieth century rarity.

	In British news, recent sales include:

	| Stamp                                                 | Price   |
	| :---------------------------------------------------- | ------: |
	| GB 1883 SG183 10s Ultramarine                         | €614.72 |
	| GB 1862 SG77 3d Pale carmine-rose (Wmk. Emblems) Pl.2 | €234.18 |

	Note however that:

	1) Prices depend on condition and circumstances.
	2) Victorian issues vary widely in circulation. My collection includes:
	   - [ ] none of the 500 orange-red 1d Mauritius Post Office stamps (1847)
	   - [x] one of the roughly 21 billion Penny Red stamps (1841-79)

	Moreover:

	~~~ Inform6
		[ DisplayStamp st;
			...
		];
	~~~

### The headings extension

As noted above, Chapter and Section headings can be achieved using the standard
Markdown notation `#` and `##`, or by "setext underlining", or in the traditional
Inform extension notation. The following are all equivalent:

	# Perforations
	
	Perforations
	============
	
	Chapter: Perforations

And similarly:

	## Embossing
	
	Embossing
	---------
	
	Section: Embossing

However, note that (unlike CommonMark) Inform-flavoured Markdown does not allow
level 1 (Chapter) or level 2 (Section) headings to be used inside block quotes
or list items.

Single-file extensions sometimes contain examples in their documentation. The
notation for these is also recognised:

	Example: ** Definitely Unhinged

(Example headings remain for the sake of backwards compatibility, but examples
should now really be broken out into stand-alone text files.
See [IE-0030](0030-extension-examples-and-testing.md).)

### Phrase details boxes

A blockquote in which the opening line reads `phrase: ...` is set out as a
"specification-of-a-phrase" box, matching the same convention used in the
main Writing with Inform documentation for Inform. For example:

	>	phrase: {ph_setpronouns} set pronouns from (object)
	>	This phrase adjusts the meaning of pronouns like IT, HIM, HER and THEM in
	>   the command parser as if the object mentioned has become the subject of
	>   conversation. Example: `set pronouns from` here -
	>
	>		set pronouns from the key;
	>		set pronouns from Bunny;
	>
	>	might change IT to mean the silver key and HIM to mean Harry "Bunny" Manders,
	>   while leaving HER and THEM unaltered.

Note that block quotes can contain multiple paragraphs, code samples (as in this
example), and even lists.

### Syntax colouring

Code samples are syntax-coloured, and Inform uses four different colouring schemes:

* `inform` or `inform7` produces Inform source text formatting.
* `inform6` (one word) produces Inform 6 formatting.
* `transcript` treats the material as a transcript of story file play.
* `plain` applied no colouring or special formatting.

A fenced code block can optionally choose a colouring with the first word of
its info string (which is read case insensitively):

	~~~ Inform6
		[ DisplayStamp st;
			...
		];
	~~~

If it makes no such choice, or is an indented code block with no info string,
then (a) if it contains a line beginning `>` then it is coloured as `transcript`,
and otherwise (b) it is coloured as `inform`.

A backticked code snippet is coloured as `inform` if it uses a single backtick
each side; as `transcript` if it uses two; and as `plain` otherwise. Thus:

	Here we see `a sample of Inform source text`, whereas ``EXAMINE SPEC`` is
	a command, and this is computery gibberish: ```malloc(1024)```.

It's important to use the double-backtick convention for transcript commands
for accessibility reasons: screen-reading software for partially sighted users
makes all caps material difficult to follow.

### Paste buttons

As in past versions of Inform, a code sample marked `{*}` has a "paste me into
the app" button attached in place of these symbols. For example:

	To set the scene:
	
		{*}	"Definitely Unhinged"
	
		Include Philately by Stanley Gibbons.

		The British Museum Stamps Room is a room. The cabinet is a closed openable
		container in the Stamps Room.

	And so on:
	
		{**}The Saxony 1856 is a stamp in the cabinet. "A forgery by Jean de Spirati!
		Priceless."

Here the `{**}` notation means that the sample continues the previous one, and
that a single paste button in the `{*}` position collects all of this text
together into a single run.

### Links and cross-references

The standard Markdown syntax `[Label]` or `[this is a link][Label]` makes a
link where the destination is referred to by a label, rather than given explicitly.
In traditional Markdown, that label can then be defined:

	See [CommonMark] for more on this.
	
	[CommonMark]: https://commonmark.org "The CommonMark specification"

And this would work fine in Inform's Markdown too, but it's not very helpful
when what we want is to make an internal cross-reference. Suppose we want a
link to Chapter 2 of the current document: we can't say what the URL for that
is because we don't know exactly how Inbuild will choose to store the HTML.

Because of that, the dictionary of labels for links is pre-populated by Inform
so that it automatically contains the names of all Chapters and Sections in
the current document, and also all Examples. So for example:

	See [Perforations] and also [Embossing]. The example [The Red Mercury]
	shows how to implement Austrian rarities.

...would all work automatically provided that the extension documentation
contains a chapter called "Perforations", a section called "Embossing" and
an example "The Red Mercury".

### Images

Extension documentation should not use externally hosted images, i.e., images
on some server which requires an Internet access to fetch. Instead, they
should use only their own private images, and in the following way:

(i) Image files should be stored in the `Images` subdirectory of the
documentation directory.

(ii) Images should be displayed using the usual Markdown `![...]` notation,
but using the _leafname alone_ as the image label. Thus, if `InvertedJenny.jpg`
is one of the image files in the `Images` subdirectory, it can be displayed thus:

	Only 100 of the following have ever existed:
	
	![1918 24c US stamp of a Curtiss JN-4 plane but printed upside-down][InvertedJenny.jpg]

In fact, to display just the image, it would be enough to write:

	Only 100 of the following have ever existed:
	
	![InvertedJenny.jpg]

but the full form is highly recommended since it provides alt-text for display
by screen-readers for partially sighted users.

### Indexing

An extension to Markdown provides for traditional TeX-style indexing markers:
indexing in the sense of drawing up an index for a book. For example:

	The ^{catalogue} of ^{@Roger S. Brody} lists three variants.^^{misprints}

The outward appearance of this line is exactly as if it had read:

	The catalogue of Roger S. Brody lists three variants.

but it accumulates three entries in the alphabetical index for the documentation:

	Brody, Roger S.
	catalogue
	misprints

Material in braces normally comes through into the visible documentation, but if
the caret `^` is doubled then it does not. The `@` marker is for names, and
inverts them when indexing, so that `^{@Alice Zephyr}` reads as "Alice Zephyr"
but indexes as "Zephyr, Alice".

Note that indexing marks are not read in backticked code or code examples. So:

	The `^{indexing}` notation is...

would not index anything, because the index markers occur inside a backticked
piece of code. (This is a point of difference with the old `indoc` tool, which
_did_ read index markers in example Inform source text.)

### Subentries

If an entry's text contains a colon (with substantive material either side),
that's taken as a marker that something is a subentry. Thus:

	^{reptiles: snakes}

indexes under the "snakes" subentry of the entry "reptiles", while coming
through into the visible documentation only as "snakes". Thus:

	"Why did it have to be ^{reptiles: snakes}?" mused Indy.

comes out as:

	"Why did it have to be snakes?" mused Indy.

Sub-entries can be arbitrarily deep. There can be, but need not be, index
entries for the super-entry (in this case "reptiles") elsewhere.

### See...

Indexes often provide multiple ways to look up the same thing: for example,
they might contain an index entry reading "Superman, see Kent, Clark".

	^^{Kent, Clark <-- Superman}
	^^{reptiles <-- crocodiles <-- alligators}

The second example here creates two such crossreferences: "crocodiles, see
reptiles" and "alligators, see reptiles".

### Alphabetization

The index consists of "headwords", the terms being indexed (which may or may not
be just one word), which are presented in alphabetical order in the index.

These are alphabetized in a way which excludes initial "a", "an" or "the";
if the first word is a number from 1 to 12, it's replaced by the spelled
version (thus "3 A.M." appears as if "three A.M."); other numbers are sorted
numerically - thus "Zone 10" appears after "Zone 9", not after "Zone 1"; and
any bracketed text is ignored for alphabetisation purposes - so "(leaf) tea"
is alphabetised as if it were "tea".

Alphabetization can be altered using the `-->` notation. For example:

	The Anglo-Saxon measure of land was the ^{hundred --> 100}.

This creates the index entry "hundred", but also tells `inbuild` that it
should be alphabetised as if it were the number 100: it thus appears in a
different place in the index, but still as the text "hundred". This only
needs to be done once. Subsequent indexing markers `^{hundred}` will
go into the same place.

Note that writing either `^{hundred --> 100}` or `^^{hundred --> 100}` will
create an index reference to "hundred", as well as changing where this
headword appears in the A-Z. If you want to set the alphabetization without
creating an index reference at all, you can use three carets:

	^^^{hundred --> 100}

## The optional contents and site-map files

While individual extensions or kits are unlikely to need them, `inbuild`
allows any directory of documentation to contain one or both of the following
files:

- `contents.txt` specifies the Markdown source files which make up one or
more "volumes", and defines the notations used to index them.
- `sitemap.txt` specifies the URLs for the HTML pages of the miniature
website which the volumes are turned into.

The point of having two different configuration files is that one defines
the content while the other says how to generate HTML from it: thus the
same content could generate different output for different purposes just
by swapping out `sitemap.txt`.

Any syntax or other errors in these files will result in the extension
documentation consisting of a list of those errors.

### contents.txt

If no `contents.txt` file is provided then there is a single volume only,
whose source must be entirely contained in the file `Documentation.md`, and
the only indexing notations used are the basic ones described above. For
almost all extensions, that will be completely fine, of course.

If given, `contents.txt` is a list of commands, each on its own line. Blank
lines are ignored, as are lines beginning with the comment character `#`.
Three commands are legal:

(1) `volume: "Full Title of Volume" or "LABEL"` creates a volume, giving both
a full title for it and also a convenient abbreviation, which should not
contain white space and should be different from the label of any other
volume in the same set. For example:

	volume: "Writing with Inform" or "WWI"

(2) `text: "Leafname.md"` specifies that the current volume, that is, the one
most recently declared, has this file as its next piece of content. A volume
can have any number of content files; for example:

	volume: "The Lord of the Rings" or "LOTR"
	text: "Fellowship.md"
	text: "Towers.md"
	text: "Return.md"

would establish that the Markdown source for the LOTR volume would consist
of these three files concatenated in the order given.

The wildcard character `*` may be used up to once in each text name. So:

	text: "chapter*.md"

may match multiple files, and if so, will concatenate them in alphabetical
order.

Note that each `text: ...` command is required to match at least one file,
and also that the `contents.txt` file as a whole is required to exactly account
for all of the files in the documentation folder whose names end in `.md`.
It is thus impossible for loose Markdown files in this folder not to appear in at
least one volume of the documentation.

(3) `index notation: NOTATION = MEANING` allows the author to extend the
set of indexing notations used in the volumes of this documentation set.

Indexing notations are used to show what "category" a headword belongs in.
By default, there are just two categories: `standard` and `name`. It's as if
the following commands are assumed:

	index notation: ^{headword} = standard
	index notation: ^{@headword} = name (invert)

To reiterate, these commands need not be given, since they are in effect
already, but they make simple examples of the syntax. The notation part on
the left shows how to mark up the `headword`, using special characters either
to the left or the right or both, to indicate the category. In the case
of `standard`, nothing special appears on either side; in the case of `name`,
an `@` character has to appear on the left. More elaborately:

	index notation: ^{XYZZY-headword-PLUGH} = magic

would set things up so that:

	The ^{XYZZY-magic word-PLUGH} cannot always be used.

would be printed as:

	The magic word cannot always be used.

but would make an index reference to the headword "magic word", of category `magic`.
Of course, that's not a very good notation, but _chacun à son goût_.

Multiple notations for the same category are perfectly legal. So:

	index notation: ^{name... headword} = name (invert)

would make typing `^{name... Emily Short}` equivalent to typing `^{@Emily Short}`.

Categories are shown in the index, and they can also affect the HTML styling, as
defined by some CSS, of the headword in question.

Note the option `(invert)` placed after the category name. Several options
are available:

* `(invert)` means "invert forenames and surname" and is what turns `Emily Short`
into the headword "Short, Emily".
* `(bracketed)` means "use different CSS styling on any pieces of the
headword which are in round brackets", the CSS style in question being
`indexCATEGORYbracketed`, where `CATEGORY` is the category name.
* `(unbracketed)` means the same, except that any such round brackets are
also removed from the headword.
* `(under X)` means: make every headword in this category a subentry of the
headword `X`.
* `(also under X)` means the same, except that headwords in this category
are in the index twice, once in their own right and once as a subentry of `X`.
* `(prefix "TEXT")` means to add the text `TEXT` in front of all headwords
of this category.
* `(suffix "TEXT")` means to add the text `TEXT` after all headwords
of this category.
* `("TEXT")` adds the gloss text `TEXT` to all headwords of this category.

So for example:

	index notation: ^{!headword} = monarch (suffix " (of Scotland)") (under monarchs)

would mean that `^{!James VI}` would lead the headword "James VI (of Scotland)"
being filed as a subentry of the headword "monarchs".

### `sitemap.txt`

If given, `sitemap.txt` is a list of commands, each on its own line. Blank
lines are ignored, as are lines beginning with the comment character `#`.
The following commands are legal:

`contents: STYLE to "PATH"`. The default is `contents: standard to "index.html"`.
There are only two possible styles: `standard` and `duplex`, which is a special
case forcing all the documentation into the customised look of the manuals
inside the Inform app, where there are two volumes presented side by side on
a special contents page. (An extension will never want to use `duplex`.)

`volume contents: VOLUME to "PATH"`. This is only needed to give individual
volumes their own contents pages. `VOLUME` should be the quoted title or label
of the volume in question, which must be one of those created by the `contents.txt`
file. For example:

	volume contents: "Writing with Inform" to "WI_index.html"

`pages: VOLUME to "PATH"` or `pages: VOLUME by BREAKING to "PATH"`. Specifies
how the content in the named volume is to be turned into individual HTML files.
Giving `all` in place of a volume name applies the command to everything in
every volume, and in particular is sensible if there's only one volume because
no `contents.txt` file was given. Optionally, material can be split up
`by sections` or `by chapters`, in which case the content will be divided
up following the level 1 and 2 headings in the Markdown documentation.
If it is not split up in this way, then there will be one HTML file output
for each Markdown file of source material in the volume.

Clearly this can all result in multiple HTML files being created, so the
`PATH` needs some flexibility in expressing names for those files. It supports
two special characters: `#` expands to the section and chapter number for
split files, so for example `5_16` for section 16 of chapter 5; and `*`
expands to the unexpanded leafname of the Markdown source file from
which material was drawn.

For example:

	pages: "Writing with Inform" by sections to "WI_#.html"

The default, if no `sitemap.txt` file is supplied, is:

	pages: all by chapters to "chapter#.html"

`example: LABELLING to "PATH"` specifies how any examples in the documentation
should be labelled — they can be `numbered`, i.e., labelled 1, 2, 3, ...,
or `lettered`, labelled A, B, C, ... — and what files they should be written to.
Again `#` is a special character meaning the label for the example. The
default setting here is:

	examples: lettered to "eg_#.html"

which produces the files `eg_A.html`, `eg_B.html` and so on. (If there are
more than 26 examples, it's clearly better to use numbers, but the lettering
runs `A`, ..., `Z`, `2A`, `2B`, ..., `2Z`, `3A`, ... and so on, so letters
do not actually run out.)

`FORM index: "TITLE" to "PATH"` specifies that a given index page should be
generated, what title to give it, and where to put it. (These are indexes
in the sense of book indexing, and not to do with the HTML sense of `index.html`
pages.) For example, the built-in Inform documentation uses all four possible
index pages:

	alphabetical index: "Alphabetical Index of Examples" to "alphabetical_index.html"
	numerical index: "Examples in Numerical Order" to "numerical_index.html"
	thematic index: "Examples in Thematic Order" to "thematic_index.html"
	general index: "General Index" to "general_index.html"

The general index is the A-Z listing of index entries produced by the `^{...}`
notations used throughout the volumes; the other three indexes are ways to
catalogue the examples in the volumes. "Thematic" order in this sense means the
order in which examples occur in the second rather than the first volume; it's
not likely to be useful for any other documentation set.

`cross-references: to "PATH"` writes a set of `indoc`-style cross-references
generated from phrase descriptions and heading markers in the volumes into
the named file, which is plain text rather than HTML. For example:

	cross-references: to "xrefs.txt"

This is only likely to be helpful for the main Inform documentation, not for
extension documentation.

## Generating documentation outside of Inform

`inbuild` can work alone to generate little documentation websites using the
above conventions, and can do so in three ways.

(1) Firstly, `inbuild` can rebuild documentation for individual extensions:

	$ inbuild -project P -document EXTENSION -document-to PATH

where `EXTENSION` is the filename or directory name for the extension, `P`
is the project within whose context the documentation is being compiled, and
`PATH` the directory to write the mini-website into, which must exist.

(2) Secondly, `inbuild` can build documentation from a free-standing directory
of content in the same format as the `Documentation` folder of an extension
or kit, but without needing it to be part of anything else, and without needing
any project as its context:

	$ inbuild -document-set PATH -document-to PATH

The optional `-document-sitemap FILE` tells `inbuild` to use the supplied
file as sitemap instructions, instead of any sitemap which came in the
documentation set (if one did). This allows documentation set up in one
context to be repurposed in another.

(3) Thirdly and least usefully, single one-off documentation files in
Markdown can also be converted:

	$ inbuild -document-from FILE -document-to PATH

Once again, an optional `-document-sitemap FILE` can be supplied. This is
essentially equivalent to generating from a documentation set directory
which contains only the single `FILE` in Markdown format.

## Syntax errors in documentation markup

The documentation generator aims to fail safe, and not to force projects to
fail to compile just because they use an extension with broken documentation.
Producing error messages in the conventional way would be inappropriate.

Instead, when `inbuild` does find a clear error, it now renders this as a red
error message box at the relevant position.

In fact, there are relatively few markup errors. All sequences of chapter
headings, section headings and examples are legal, for example.
