# (IE-0028) Extension documentation revisited

* Proposal: [IE-0028](0028-extension-documentation-revisited.md)
* Discussion PR link: [#28](https://github.com/ganelson/inform-evolution/pull/28)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0001](0001-extensions-with-resources.md), [IE-0017](0017-apps-and-extensions.md), [IE-0030](0030-extension-examples-and-testing.md)
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

A backticked code snippet is coloured as `inform` unless it uses two or more
backticks to mark it, in which case `plain`. Thus:

	Here we see `a sample of Inform source text`, and here some more
	computery gibberish: ``malloc(1024)``.

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
