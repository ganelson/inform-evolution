# (IE-0001) Directory format for extensions with resources

## Motive

At present all Inform extensions are single files, with names ending ".i7x".
Historically these have been stored in "nests" like so:

	Extensions
		Emily Short
			Locksmith.i7x
			Red Fire Hydrants.i7x

This simple system had advantages, but causes problems as soon as the extension
needs to contain anything other than Inform source text. It is already awkward
to store documentation and examples inside this single file; there's certainly
no nice way to put anything else in. This limitation paints us into a corner.

That is particularly true for an extension which needs to be augmented by
other resources: an extension to be used only with a certain website template,
or to translate the language of play into French, or to provide low-level
features which need support from a kit of Inter code. We want end users to
be able to obtain everything necessary to use an extension in a single download.

In this proposal, then, extensions can contain kits, language bundles or
templates, but not other extensions or Inter compilation pipelines.

Finally, another motivation here is to leave room for future developments.

### A note on kits

Kits are libraries of Inter code, stored as binary Inter files but compiled
by inbuild from Inform 6-syntax source. They are new in Inform 10.1, so the
following will make little sense without having read the inter manual.

Most extensions do not need to use a kit, but a language extension like Norwegian
Language by Hypothetical Person would need to have NorwegianLanguageKit around.
Users have to successfully install both, and may need to do something clumsy to
ensure that the kit is read in, since although a kit can require an extension,
the reverse is currently not true. All very well for hackers experimenting with
the system, but not good for end users.

One implication of the proposal below is that authors of kits who want to make
them available for other Inform users should always distribute those kits
wrapped up inside extensions. As a result of that, end users need not know what
kits even are, and need not customise their projects in strange ways in order
to use them. If "Heapsort by Dannii Willis" is really powered by a kit of Inter
code, users will not need to care about that: it will be an extension like
any other, so far as the user is concerned.

Distributing kits wrapped up in extensions is no loss of functionality since:

(i) kits almost certainly need at least some source-text "To..." phrases or
rules to make their powers available to users, and a surrounding extension
is a natural way to do that, and

(ii) even if that is not the case, the surrounding extension could just be
a couple of lines of source text.

This also solves two other problems:

(i) Storing kits in a way which avoids namespace collisions if the user has
two extensions installed which each unwittingly have a kit called, say, SignalKit.

(ii) Making it easy for kit authors to provide documentation to users, since
this can just be the documentation on the wrapper extension.

## Proposed changes

### 1. Changing from one file to a directory of files

1.1. At any storage position where an extension file such as Locksmith.i7x is
expected, an extension directory can be given instead.

1.2. Either can be read, for backwards-compatibility, but the directory format
will be preferred. If we move to a public registry of Git repositories which
hold extensions, then directory-format will be required for those. In general,
we will want to phase out the use of single-file extensions across the board.

1.3. inbuild will have an option, say -convert-extension, which takes a single-file
.i7x extension (i.e. what exists now), perhaps incorrectly identified or with
an otherwise dubious filename, and makes an equivalent and correctly-named
directory. (Inbuild is already able to identify extensions properly regardless
of their filenaming.)

The Inform apps will be able to use this when they install extensions which the
user has downloaded from somewhere. At present, the UI apps all have their own
essentially duplicated code for working out how and where to install a new
extension: we really want to move that into inbuild, saving the UI apps from
having to know about any of this.

1.4. The naming rules for this directory will be almost the same as those for
single extension files at present, except that:

(a) no ".i7x" file extension will be used;

(b) a version number like "-v4", which can optionally be appended to single
extension file names, is mandatory in the case of directory names.

For (b), note that v10.1.0 of Inform introduced the possibility of an extension
filename containing its version number, so that multiple versions of the same
extension can be held alongside each other: thus where v9 and earlier would
only recognise "Locksmith.i7x", v10 also recognises "Locksmith-v3_2_1.i7x", say,
for version 3.2.1. (Provided that this is in fact the version number inside; if
it is not, Inform complains.)

1.5. Here is an example of what a directory might look like, for version 1.3 of
the hypothetical extension "Red Fire Hydrants by Emily Short":

	Red Fire Hydrants-v1_3
		Source
			Red Fire Hydrants-v1_3.i7x
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
				gushing.mp3

"Red Fire Hydrants-v1_3.i7x" is the extension file as at present: no change
to its syntax is proposed. The "Source" directory may contain further files in
line with (IE-3); to be decided.

Everything else (i.e. other than "Source") is optional.

### 2. Documentation

2.1. The "Documentation" subdirectory is optional, and provides an alternative to
writing documentation in the source text of an extension. It is an error for an
extension to provide documentation in both forms. This optional form is much richer,
allowing for larger manuals with multiple chapters and sections, examples, and
indexing.

2.2. If provided, it must contain "Documentation.txt", an indoc-format manual for
the extension. This is optionally allowed to use Examples and Images, in their own
subdirectories of "Documentation". (There is no indoc configuration file; inbuild
will manage the finicky business of configuring indoc to compile this documentation.)

2.3. In v10.1.0, the documentation-compiler "indoc" is not included in the Inform apps,
but it now will be.

2.4. When inbuild wants to generate documentation from an extension providing this
directory, it will use "indoc" instead. (If running inside the apps, where the tools
may not be able to run each other directly for sandboxing reasons on the local
operating system, it will print out the necessary commands in some way so that the
app can then run "indoc" on its behalf.)

2.5. At present, inbuild generates documentation on an extension every time it is
used. For large "Documentation"-style manuals that could prove slow, so inbuild may
cease to generate documentation every time, and instead do so only if the source or
the full Inform build version has changed since the last time.

### 3. Kits

3.1. An extension directory can optionally contain one or more kits, inside
a "Materials/Inter" subdirectory. Like all kits, these will have names ending in
"Kit" (and no file extension, since a kit is itself a directory holding a variety
of resources).

3.2. Kits contain metadata which, among other things, can say that if the
kit is to be used then so must a given extension: unsurprisingly, a kit
which entirely belongs to an extension must mandate the use of that extension.

3.3. (i) If just one kit is supplied, it is automatically linked into any project
including the extension.

(ii) If multiple kits are supplied, the one whose name is that of the extension
but with spaces removed and "Kit" put at the end is linked in. (So, for example,
"RedFireHydrantsKit".) Any other kits also supplied will only be linked in if
the If-This-Then-That rules for the primary kit's metadata call for them. For
example, "RedFireHydrantsKit" could see if the user is also intending to have
"FireFightersKit" present, and if not then to link "DefaultFireFightersKit"
instead.

Clearly (ii) is an edge case, but it's something which BasicInformKit
does in order to be able to work in both basic and non-basic Inform projects.

### 4. Language bundles

"Language bundles" are used internally to represent natural languages (English,
French, German and so on), but they are in practice no use without an extension
to match.

4.1. An extension directory can optionally contain one or more language bundles,
inside a "Materials/Languages" subdirectory. In practice, it is likely to contain
either none or exactly one, of course.

4.2. At present inform/inform7/Internal/Languages holds a selection of
language bundles written some years ago, but it really shouldn't: the core
Inform distribution should ship only with the default English language bundle.
All others will be deleted from the core distribution.

Instead, what might formerly have been stored as "Internal/Languages/French" is
now at "Extensions/Marcel Proust/French Language/Materials/Languages/French" (i.e.,
it has moved into the extension itself).

4.3. Note that by making language bundles belong to extensions, we make it possible
for a user to have alternative versions of the same language to choose from.
If there is also an "Extensions/Victor Hugo/French Language" then these extensions
can provide different French language bundles, and the one applying to a given
project will be the one belonging to the extension the user chose to include.

It will however be an error for a project to include two extensions which both
include a language bundle with the same name. (This is no loss to the user: that
was never going to end well.)

4.4. A curiosity of Inform at present is that every language bundle found by
inbuild becomes an enumerated value for the "natural language" kind when Inform
is working. Cool in a way, but in practice this means that the same source text
compiles differently on different users' machines according to which language
bundles they have installed, which is a little odd. This will change so that
the only enumerated values are those from (i) English (the default), and (ii)
any actually Included extension which contains one.

4.5. Language bundles will be able to include Preform files: details to be worked
out.

### 5. Media resources

5.1. Materials/Figures and Materials/Sounds allow extensions to provide these,
exactly as the Materials folder for a project can.

5.2. Inside an extension, filenames in figure and sound declaration sentences are
taken as referring to these, but see (7) below.

### 6. Website/interpreter templates

6.1. An extension directory can optionally contain one or more website templates,
inside a "Templates" subdirectory. In practice, it is likely to contain
either none or exactly one, of course. The effect of this is that if the
extension is included by a project, then "Release with..." will be able to
use this template, and if not, not.

### 7. Overriding materials

7.1. Suppose a project called, say, "Boston Fire Dept" wants to use an extension,
but to override its choice of a given image (or kit, or sound, or template, or
language bundle) and provide a customised alternative. It can do so by providing
those alternatives in its own materials directory.

For example, "Boston Fire Dept.materials/Figures/Red Fire Hydrants/dalmatian.jpg"
would override "Red Fire Hydrants-v1_3/Materials/Figures/dalmatian.jpg".

7.2. Note the presence of the intermediate directory "Red Fire Hydrants". The project
could conceivably also have its own unrelated image of a dalmation, with the same
leafname. (To be considered: would this cause problems in the blorb? Do we need to
mangle these filenames to ensure there are no name clashes in the final release?)
