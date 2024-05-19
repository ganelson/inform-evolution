# (IE-0016) Language extensions reform

* Proposal: [IE-0016](0016-language-extensions-reform.md)
* Discussion PR link: [#16](https://github.com/ganelson/inform-evolution/pull/16)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0001](0001-extensions-with-resources.md)
* Implementation: Implemented but unreleased

## Summary

This proposal is a spin-off from work done for [IE-0001](0001-extensions-with-resources.md),
"Directory format for extensions with resources". With that implemented, a "language
extension" - one which enables Inform-made stories to play in natural languages
other than English - can now contain its own language bundle and kit. In the
process, language bundles have also been reformed, and this proposal is intended
to document the new arrangement.

## Motivation

The aim here is to make language extensions simpler for authors to deal with,
to remove out-of-date restrictions in the compiler, and to lay some ground-work
for future development. We clearly still have a long way to go on localising
Inform in any complete way.

## Components affected

- [x] Minor extension to the natural-language syntax.
- [x] Significant changes to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Projects playing in languages other than English might need their top lines
modified, but will probably not. (This depends on how the translation extension
they use has been set up.)

"Translation extensions" - those which enable play in languages other than
English - will need to be reconstructed.

"Language bundles" will similarly need to be changed, and folded into the
associated translation extensions.

It is also just possible that source text dealing with the obscure built-in
kind `natural_language` might need to change, but only if it was doing something
quite strange to begin with.

## Evolution

To see the point of these changes, it's worth first summarising how Inform has
previously handled languages other than English.

### Inform v9 and earlier

To make Inform produce stories playing in, say, French, the user needed both
a language bundle (a directory containing some metadata, called `French`), and
also an extension (say, `French Language by Victor Hugo`). The bundle would
refer to the extension, but they were not stored together. Bundles could be
stored in the `Languages` subdirectory of a project's materials folder, or
in the `Languages` subdirectory of the user's installed resources directory,
alongside the user's `Extensions`. Failing that, Inform could fall back on
a few language bundles ready-installed in its internal resources area. This
was a fairly random selection and hard-wired references to the partly or fully
made translations which existed in the early 2010s: 

	French
	German
	Italian
	Spanish
	Swedish

Meanwhile, the extension (say, `French Language by Victor Hugo`) would need
to define some Inform 6 material, which it would have to write with length
inclusions using `(-` ... `-)`, and which would replace out various English-specific
definitions in a template file called `Language.i6t`: for example, the extension
would replace the existing array `LanguageNumbers` with one of its own, with
suitable data for French rather than English.

The author of a story wanting to use all of this would write an opening title line like so:

	"Madame Bovary" by Gustave Flaubert (in French)

The Inform compiler would read the `(in French)`, look for a language bundle
called `French`, and see from that which language extension it should automatically
include.

### Inform v10

Largely the same setup for language bundles. However, with the abolition of the
"Inform 6 template", and its replacement by kits, the way which the translator
had to redefine arrays like `LanguageNumbers` changed. These arrays and functions
were now in a kit called `EnglishLanguageKit`. When the Inform compiler saw from
the opening title line:

	"Madame Bovary" by Gustave Flaubert (in French)

...that the story was to be in French, it would load `FrenchLanguageKit` instead
of `EnglishLanguageKit`. That meant that many `(-` ... `-)` inclusions could go
from the extension itself, but on the other hand, the translator now had to
create a new kit called `FrenchLanguageKit`, making a French version of
`EnglishLanguageKit`. While this was conceptually better (or arguably so), it
meant that users now had to juggle three associated resources: the bundle,
the extension, and the kit.

Another change with Inform v10 was the arrival of the build manager, `inbuild`.
This supported the idea that a project might have three different natural languages
associated with it:

* the language of play, which is what the reader/player sees at runtime;
* the language of syntax, which is what the author writes as source text;
* the language of indexing, which is the language used in the Index for a project.

However, `inbuild` did not really allow these to be set independently in any
very helpful way. It was only really possible to set the language of play, and
the language of indexing would be set to that automatically; while the language
of syntax remained `English`, so that Inform's "Preform" features could not very
reliably be used to change the syntax of Inform to make it more French-like.

### As changed by this proposal

The new scheme is that a translator supplies just one resource to authors: the
extension. This is an extension in directory form (see IE-0001), which means it
can contain various resources inside itself. It uses this ability to store its
own language bundle (in effect, identifying itself as being not just any extension,
but a language extension), and its own kit. The directory structure might look
like this:

	French Language-v3_1
		extension_metadata.json
		Materials
			Inter
				FrenchLanguageKit
					...
			Languages
				French
					Index.txt
					language_metadata.json
					Syntax.preform
		Source
			French Language.i7x

Of course, authors using this extension would never need to look inside it,
and would probably download it in zipped form as a single download.

In addition, the syntax for opening lines of Inform projects has been extended.
These, for example, are all possible:

	"Madame Bovary" by Gustave Flaubert (in French)
	"Madame Bovary" by Gustave Flaubert (played in French)
	"Madame Bovary" by Gustave Flaubert (written in English, played in French)
	"Madame Bovary" by Gustave Flaubert (played and indexed in French)
	"Madame Bovary" by Gustave Flaubert (written, played and indexed in French)
	
In this way, the languages of syntax, play and indexing can all be set
independently. Writing just `in French` is equivalent to `played in French`;
similarly, Inform allows `in French language` as equivalent to `in French`.

The defaults, for those not set, are:

* The language of play defaults to English.
* The language of syntax defaults to English.
* The language of index defaults to the language of play.

When Inform (properly speaking, `inbuild`) looks at this opening line of a
project, it now searches for language bundles differently. If it reads

	"Madame Bovary" by Gustave Flaubert (played in French)

then it knows it needs to find a language bundle called `French`. In previous
releases, Inform would search for this by looking in the `Languages` subdirectories
of the "nests" of resources available: in the project's materials folder, then
in the user's installed area, then in the Inform app's built-in resources.

Now, however, Inform looks _first_ for an extension called `French Language`,
stored in directory form. (It looks for this by putting the name of the language,
here `French`, in front of the word `Language`. In practical terms, that means
that from now on, all translation extensions need to have titles in this form:
`Spanish Language`, `Russian Language` and so on.) Inform will use this extension
only if it is in directory form, and contains within it a language bundle with
the corresponding name, `French` - as in the example directory layout above.
If Inform does find that, then:

* Inform uses the included language bundle, e.g., `French`;
* Inform automatically includes the translation extension, e.g., `French Language by Victor Hugo`;
* Inform automatically includes the kit `FrenchLanguageKit` but does not include `EnglishLanguageKit`.

Of course, if we use only the language name to identify which translation
extension we want to use, then we make it hard to have multiple versions of the
same language by different translators. To get around that, authors can be
more explicit about who they want the translator to be:

	"Madame Bovary" by Gustave Flaubert (in French by Victor Hugo)
	"Madame Bovary" by Gustave Flaubert (in French by Marcel Proust)
	
This causes Inform to search for a translation extension called `French Language`
by a specific translator, not just for anybody's `French Language`.

The above is what happens if French is going to be the language of play. If it
is going to be the language of syntax, then Inform also loads the file `Syntax.preform`
from the language bundle. (This file is otherwise optional, and is not needed if
the translation extension is not intended to be able to change the language of syntax.)
`Syntax.preform` for English is read first, and then `Syntax.preform` for the
language of syntax, if it is something different from English. (Both need to be
read because the Standard Rules, etc., are all in English syntax source text.)
It is beyond the scope of this note to explain how `Syntax.preform` works, but
see the compiler's documentation on Preform, and the English version.

The built-in set of language bundles other than `English`, which in 10.1 was
`French`, `German`, `Italian`, `Spanish`, and `Swedish`, has been removed from
the core Inform distribution and will thus disappear from the apps. This will
prevent out-of-date references there from causing problems, and will make
clearer that anyone can create a translation extension, without needing changes
to be made to the core Inform distribution.

Finally, an oddity of Inform up to now has been that the built-in kind of value
called `natural_language` was set up on each compilation with one instance for
each language whose bundle the compiler could find - so, typically, a game would
be compiled with six enumerated values of `natural_language`, even though most
would come from language bundles not involved in the compilation at all. This is
unsatisfactory, because it means that the same source text compiles differently
according to what resources are installed on the user's computer. This Basic Inform
source text:

	To begin:
		showme the list of natural languages.

produces different output for different users. This practice has been abolished.
Instead, `natural_language` is an enumeration of a very simple kind indeed - it
is compiled with either one or two instances. `English language` is always present,
and is the default value for the kind; if the language of play is different from
English, then that provides a second instance. So for example:

	"list of natural languages" = list of natural languages: {English language, French language}

## New metadata for language bundles

Like (directory) extensions and kits, language bundles now provide metadata about
themselves in a JSON file, in this case called `language_metadata.json`. This replaces
a simpler-looking file called `about.txt` which looked something like this:

	1	French
	2	Français
	3	en français
	4	fr
	5	Nathanaël Marion

The `about.txt` file must be removed. For the JSON replacement, follow this model:

	{
		"is": {
			"type": "language",
			"title": "French"
		},
		"language-details": {
    		"supports": [ "played", "written", "indexed" ],
			"translated-name": "Français",
			"iso-639-1-code": "fr",
			"translated-syntax-cue": "en français",
			"flag": "🇫🇷"
		}
	}

A few notes:

(1) The language bundle does not specify an author in its `is` object:
but of course it's very likely to be part of an extension which does have a
named author, so it's not really anonymous.

(2) A language bundle, unlike a kit or an extension, cannot have `needs`, that
is, cannot require other resources to be present. It is the lowest of the low:
it is needed by others, but needs nothing for itself.

(3) The `"supports"` list says what this bundle can do. Inform will reject an
attempt to compile a project `written in French`, for example, if `"written"`
is not in the `"supports"` list for that language. To support `"played"`, a kit
must be provided as part of the surrounding extension; to support `"written"`, a
Preform file must be provided in the language bundle; to support `"indexed"`,
an `Index.txt` file must be provided in the language bundle.

(4) `"translated-syntax-cue"` is now optional. At one time it was intended that
projects could perhaps have opening lines like so:

	"Madame Bovary" by Gustave Flaubert (en français)

and that the "cue" `en français` would make Inform look for a suitable language
bundle, and then set the language of syntax and play both to that language.
That now seems too confusing, and is difficult for `inbuild` to deal with
efficiently. So for the moment, at least, `"translated-syntax-cue"` is not used.

(5) `"flag"` is optional. If given, it should be a Unicode representation of an
appropriate national flag (i.e., an Emoji Flag Sequence made up of two adjacent
Regional Indicators). Languages of course are not the same thing as countries,
but it's just for the sake of a visual shorthand.
