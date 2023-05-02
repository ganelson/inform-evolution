# (IE-0001) Directory format for extensions with resources

* Proposal: [IE-0001](0001-extensions-with-resources.md)
* Discussion PR link: [#1](https://github.com/ganelson/inform-evolution/pull/1)
* Authors: Graham Nelson
* Language feature name: None
* Status: Draft
* Related proposals: [IE-0016](0016-language-extensions-reform.md), [IE-0017](0017-apps-and-extensions.md)
* Implementation: In progress

## Summary

Substantially expanding the range of resources an extension can provide.

## Motivation

At present all Inform extensions are single files, with names ending `.i7x`.
Historically these have been stored in the `Extensions` directory of a
so-called "nest" (for example, the materials for a project are a nest).
For example:
```
	Extensions
		Emily Short
			Locksmith.i7x
			Red Fire Hydrants.i7x
```
This simple system had advantages, but causes problems as soon as the extension
needs to contain anything other than Inform source text. It is already awkward
to store documentation and examples inside this single file; there's certainly
no nice way to put anything else in. This limitation paints us into a corner.

That is particularly true for an extension which needs to be augmented by
other resources: an extension to be used only with a certain website template,
or to translate the language of play into French, or to provide low-level
features which need support from a kit of Inter code. We want end users to
be able to obtain everything necessary to use an extension in a single download.

In this proposal, then, extensions can contain kits, language bundles,
website and interpreter templates, sound effects, images, and data files, but
not other extensions or Inter compilation pipelines.

In particular, the way language bundles and translation extensions work - for making
Inform stories which play in languages other than English - is significantly
changed by all of this. The details are broken out in the associated proposal
[IE-0016](0016-language-extensions-reform.md).

## Future implications

Firstly, we expect that extensions will be the most practical way to
distribute kits and language translations, and because of that we won't try to
have separate public library-like repositories for those: we'll instead take
the view that users of Inform will always install what they need in the form
of extensions, even if those extensions are thin wrappers around something else.

Secondly, there's likely to be a shift in what is best practice when writing
or maintaining extensions. Currently, complicated extensions tend to be full
of inclusions of low-level code `(-` ... `-)`, for example: that's better when
broken out into an accompanying kit.

Distributing kits wrapped up in extensions is no loss of functionality since:

(i) kits almost certainly need at least some source-text `To...` phrases or
rules to make their powers available to users, and a surrounding extension
is a natural way to do that, and

(ii) even if that is not the case, the surrounding extension could just be
a couple of lines of source text.

This also solves two other problems:

(i) Storing kits in a way which avoids namespace collisions if the user has
two extensions installed which each unwittingly have a kit called, say, `SignalKit`.

(ii) Making it easy for kit authors to provide documentation to users, since
this can just be the documentation on the wrapper extension.

Traditional single-file extensions can still be read, for backwards-compatibility,
but the directory format will be preferred in future.

## Components affected

- [ ] No change to the natural-language syntax.
- [x] Major changes to inbuild.
- [ ] No change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Minor changes to documentation.
- [x] Eventual change to the GUI apps, when downloading or installing extensions.

## Impact on existing projects

Because of the way extensions can now provide figures, sounds and so on,
the Inform compiler has to be more proactive in looking for the relevant
files. As a result, and for the first time, Inform checks for the existence
of sound and picture files which the source text claims to exist.
Previously, if they did not exist then this would come to light only when
indexing or releasing. It's now a compilation failure to refer to a
non-existent resource.

This can be deactivated using the new command-line switch `-no-resource-checking`,
which is a convenience for the Inform test suite.

## Example

Here is an example of what a directory might look like, for version 1.3 of
the hypothetical extension `Red Fire Hydrants by Emily Short`:
```
	Red Fire Hydrants-v1_3
		extension_metadata.json
		Source
			Red Fire Hydrants.i7x
		Documentation
			Documentation.txt
			Examples
				Alpha.txt
				Zulu.txt
			Images
				diagram.jpg
		Materials
			Inter
				RedFireHydrantsKit
				DefaultFireFightersKit
			Figures
				dalmatian.jpg
			Sounds
				gushing.ogg
```
`Red Fire Hydrants.i7x` is the extension file as at present: no change
to its syntax is proposed.

`extension_metadata.json` is a JSON-format metadata file analogous to the
`kit_metadata.json` metadata for kits, and following the same schema, though
with an `extension-details` member replacing `kit-details`.

Everything other than `Source` and the JSON file is optional.

## JSON metadata file

A minimum viable metadata file looks like this:

	{
		"is": {
			"type": "extension",
			"title": "Red Fire Hydrants",
			"author": "Emily Short",
			"version": "1.3"
		}
	}

However, the file may also specify any or all of the following:

`"compatibility"` is a string saying which architectures the extension is
compatible with; by default it's assumed to be universally compatible. For
example:

	{
		"is": {
			...
		},
		"compatibility": "Glulx only"
	}

For the full specification of compatibility texts, see `Compatibility.w` in
the Inform source code module `arch`. Other examples would be `"Z-machine version 8"`,
`"Glulx without debugging"`, `"16d or 32d"`. Note that compatibility describes
architectures, not compilation targets: you can't specify "don't compile this via C".

`"activates"` is a list of strings, each of which is the name of some compiler
feature which the extension - simply by virtue of being used in a project - turns
on for that project.

`"deactivates"` similarly turns off compiler features.

`"needs"` specifies what other resources the extension needs in order to be used.
For example, here's a valid JSON metadata file for our example extension:

	{
		"is": {
			"type": "extension",
			"title": "Red Fire Hydrants",
			"author": "Emily Short",
			"version": "1.3"
		},
		"needs": [ {
			"need": {
				"type": "kit",
				"title": "RedFireHydrantsKit"
			}
		}, {
			"need": {
				"type": "kit",
				"title": "DefaultFireFightersKit"
			}
		}, {
			"need": {
				"type": "extension",
				"title": "Big Yellow Tubing",
				"author": "Aaron Reed"
			}
		} ],
	}

As can be seen, `"needs"` is a list of objects - here, three items long. Two of
the extension's requirements are kits. Those will be easy to find, since they
are private to the extension and stored within it (see below). But because they
are listed here as needs, Inform automatically merges them into any build of a
project which includes our extension. (It also incrementally rebuilds them
from source as necessary.)

It might seem that every kit private to the extension will be one of its kit needs,
and vice versa. In fact, neither of those has to be true:

* An extension can "need" a kit which is not private to it. Inform will then
look for this just as it looks for other resources - for example, from the project's
materials folder.
* An extension does not have to "need" every kit private to it. Kits can
in principle have quite complicated further requirements when used. We could
imagine a situation where the extension needs private kit A, which then sometimes -
but only sometimes - needs private kit B as well. The dependency of the extension
on kit A would be in the extension's JSON metadata, whereas the dependency of kit B
on kit A would be in kit A's JSON metadata.

The third need is for another extension, and this of course is not housed
inside the `Red Fire Hydrants-v1_3` directory - extensions cannot contain other
extensions. So it will be looked for in the usual manner.

Needs can optionally give a version number for what they want. For example:

	{
		"need": {
			"type": "extension",
			"title": "Big Yellow Tubing",
			"author": "Aaron Reed",
			"version": "7.2"
		}
	}

Such versions are then interpreted according to semver rules: so this would
work with any 7.x, where x is at least 2, but not with 7.1 or 8.3.

### Redundancy of the metadata file

Some of the information in the metadata file effectively duplicates information
which could be deduced from the extension's source text.

For example, Inform requires that the metadata must exactly match the title,
author and version given in the titling line of the extension source text. We would
not allow the extension to begin:

	Version 3.2 of Chrome Couplings by Andrew Plotkin begins here.

but the metadata to read:

	{
		"is": {
			"type": "extension",
			"title": "Shiny Chrome Couplings",
			"author": "Andrew Zarf Plotkin",
			"version": "3.2.8"
		}
	}

Similarly, the extension needs can be deduced from the source text, because
if the extension source includes:

	Include version 7.2 of Big Yellow Tubing by Aaron Reed.

...then it clearly needs this extension.

Not all of the data in the JSON file is redundant: the details about kits cannot
be expressed in any other way and are essential to the correct working of the
extension.

So why do we have any redundant data in this file? The answer is that we will want
Internet registries software to be able to know, in an efficient way, what
an extension is and what other resources it might need in order to operate. The
idea is that looking at the JSON file alone - which is straightforward to parse -
tools for filing and dealing with Inform resources can learn all that they need
to know. Moreover, the JSON schema we use can also describe other Inform resources,
such as kits. We want to handle all of these in a more or less uniform way.

## Extension materials

The optional `Materials` subdirectory can provide a variety of resources, just
like the materials folder for a project, and it has essentially the same layout.

* Figures can be provided in `Materials/Figures`.
* Sound effects can be provided in `Materials/Sounds`.
* Data files can be provided in `Materials/Data`.
* Templates for websites or interpreters can be provided in `Materials/Templates`.
This means extensions can be wrappers for these, so that users do not even need
to write their own release instructions - because those can be put in the
extension's source text.
* Kits can be provided in `Materials/Inter`. They are private to this extension,
and are linked into a project including the extension _if the extension metadata says so_. See above.
* Language bundles can be provided in `Materials/Languages`. See [IE-0016](0016-language-extensions-reform.md).
* But extensions may not be provided in `Materials/Extensions`. Extensions
containing other extensions... is a regress too far.

Inform already supports figure and sound effect declarations (and see [IE-0004](0004-using-data-files-in-blorbs.md) for data files),
with sentences like this:

	Figure of dalmatian mascot is the file "dalmatian.jpg".
	Sound of gushing water is the file "gushing.ogg".

If such sentences are found in the main source text, nothing changes about their
meaning: the files, `dalmatian.jpg` and `gushing.ogg`, are looked for in the
project's materials folder, as before.

But if such sentences are found in the source text for an extension directory,
then Inform looks for the files first in the materials for the extension.
If it doesn't find them there, it then turns to the project's materials folder.
(In particular, this means that any existing extension declaring figures or
sounds like this will continue to work as before.)

If Inform does find that the extension directory has provided the resource, it
then just checks to see if the author has _overridden_ this. For example, suppose
the author using `Red Fire Hydrants by Emily Short` doesn't like the dog picture,
and wants to substitute a better one. That author can then supply this:

	Hypothetical Project.materials/Figures/Red Fire Hydrants/dalmatian.jpg

Note that this is in a subdirectory of `Figures`, with the same name as that
of the extension. This means the author could replace `dalmatian.jpg` from
multiple different extensions, while still having a quite unrelated `dalmatian.jpg`
used by the project's main source text.

## Exactly how inbuild reads and repairs extensions

As noted above, directory extensions are fiddly to get right, because in some
cases the same information has to be present in multiple places.

Directory extensions can be scanned in two modes: repairing, and not. In
repairing mode, we not only forgive minor errors, but put them right.

* `inform` scans extensions in a project's Materials folder in repairing mode,
but all other extensions in non-repairing mode.
* `inbuild` scans in repairing mode if and only if the command line switch `-repair`
is used when running it.

The idea is that in repairing mode, we silently fix mistakes whenever it seems
prudent to do so, which means altering the extension in the file system. In
non-repairing mode, or if repairs are tried but fail because of, say, a file-system
error, we report all errors.

The main use case for repairing mode is this: When an author changes the title or
version number of an extension by editing the source text, Inform then automatically
takes care of correcting the discrepancies which would otherwise arise from the fact
that it now sits in a wrongly-named directory structure, and has non-matching JSON
metadata.

### What do we consider a directory extension?

`inbuild` (and hence `inform`) regards a directory as holding an extension if either:

* It contains a file called `extension_metadata.json`, or
* It has a directory name ending in `.i7xd`.

As we shall see, both of these are required to be true: so if only one is true, the
directory is regarded as a damaged extension, but still an extension.

### What is considered as damage, and what can be repaired?

#### Source file

(1) A directory extension must contain at least one source text file whose name ends
`.i7x`, and which is in the `Source` subdirectory of the directory.

If there are multiple files meeting this description, Inbuild first looks for one
called `Title.i7x`, where `Title` is the title of the extension. Failing that, it
looks for the alphabetically earliest.

If no file with a name ending `.i7x` is present, this is irreparable, and a problem
is thrown.

(2) The source text file must have a valid opening line in Inform extension syntax:

	Version 5.4 of Marble Staircases by Andrea Palladio begins here.

If not, this is irreparable, and a problem is thrown.

(3) While it is optional for single-file extensions to provide a version number,
it is mandatory for directory extensions to do so.

If not, this is irreparable, and a problem is thrown.

(4) The source text file must have the filename `<Title>.i7x`, where the Title
is as given in its own opening line. For example:

	Marble Staircases.i7x

In repairing mode, the file is renamed if this is not true.

#### Directory name for the extension

(1) The directory name for the extension must take the form `<Title>-v<Number>.i7xd`,
where the Title and Version number exactly match those in the source file. The
dots occurring in the Number, if any, are flattened to underscores. For example:

	Marble Staircases-v5_4.i7xd

In repairing mode, the directory is renamed if any part of this is wrong.

#### JSON metadata file

(1) The directory must contain a file called `extension_metadata.json`.

In repairing mode, such a file is created if absent, using the metadata extracted
from the first line of the extension source. For example:

	{
		"is": {
			"type": "extension",
			"title": "Marble Staircases",
			"author": "Andrea Palladio",
			"version": "5.4"
		}
	}

(2) The `extension_metadata.json` must be valid JSON, which matches the requirements in:

	inform7/Internal/Miscellany/resource.jsonr

If not, this is irreparable, and a problem is thrown.

(3) In this metadata, the `is.type` must be `extension`.

If not, this is irreparable, and a problem is thrown. (It seems best not to
attempt a "repair" if what has happened is that the user has accidentally copied
in a kit metadata file, say.)

(4) In this metadata, the `is.title`, `is.author` and `is.version` must all match those
declared by the extension source.

In repairing mode, the metadata file is rewritten to correct any discrepancies, taking
the extension source as the correct one. Note that any other material in the existing
JSON file, if there is one, will be preserved by this: only the `is` object changes.

#### Contents of the extension

(1) The extension directory may contain no files except:

* "Hidden files" beginning with a `.`, which Inform ignores.
* `extension_metadata.json`.

If it does contain other files, this is irreparable, and a problem is thrown.

(2) The extension directory may contain no subdirectories except:

* "Hidden subdirectories" beginning with a `.`, which Inform ignores.
* `Source`, which it must contain (see above).
* `Materials`, which is optional.
* `Documentation`, which is optional.

If it does contain other subdirectories, this is irreparable, and a problem is thrown.

(3) The Source subdirectory must contain exactly one (unhidden) file, which must
be the source text, and no (unhidden) subdirectories.

If it does contain other matter, this is irreparable, and a problem is thrown.

(4) The Materials subdirectory must contain exactly no (unhidden) files, and can
only contain the following (unhidden) subdirectories:

* `Data`, which is optional.
* `Figures`, which is optional.
* `Inter`, which is optional.
* `Sounds`, which is optional.
* `Templates`, which is optional.

If it does contain other matter, this is irreparable, and a problem is thrown.

(5) The Documentation subdirectory must contain exactly one (unhidden) file, called
`Documentation.txt`, and can only contain the following (unhidden) subdirectories:

* `Examples`, which is optional.
* `Images`, which is optional.

If it does contain other matter, this is irreparable, and a problem is thrown.

## Not yet implemented

### Documentation

The `Documentation` subdirectory is optional, and provides an alternative to
writing documentation in the source text of an extension. It is an error for an
extension to provide documentation in both forms. This optional form is much richer,
allowing for larger manuals with multiple chapters and sections, examples, and
indexing.

If provided, it must contain `Documentation.txt`, an indoc-format manual for
the extension. This is optionally allowed to use Examples and Images, in their own
subdirectories of `Documentation`. (There is no indoc configuration file; inbuild
will manage the finicky business of configuring indoc to compile this documentation.)

In v10.1.0, the documentation-compiler `indoc` is not included in the Inform apps,
but it now will be.

When inbuild wants to generate documentation from an extension providing this
directory, it will use `indoc` instead. (If running inside the apps, where the tools
may not be able to run each other directly for sandboxing reasons on the local
operating system, it will print out the necessary commands in some way so that the
app can then run `indoc` on its behalf.)

At present, inbuild generates documentation on an extension every time it is
used. For large `Documentation`-style manuals that could prove slow, so inbuild may
cease to generate documentation every time, and instead do so only if the source or
the full Inform build version has changed since the last time.

### Multiple source files

The `Source` directory may contain further files in line with [IE-0003](0003-multiple-source-files.md); to be decided.

### Conversion

`inbuild` will have an option, say `-convert-extension`, which takes a single-file
`.i7x` extension (i.e. what exists now), perhaps incorrectly identified or with
an otherwise dubious filename, and makes an equivalent and correctly-named
directory. (Inbuild is already able to identify extensions properly regardless
of their filenaming.)

The Inform apps will be able to use this when they install extensions which the
user has downloaded from somewhere. At present, the UI apps all have their own
essentially duplicated code for working out how and where to install a new
extension: we really want to move that into inbuild, saving the UI apps from
having to know about any of this.
