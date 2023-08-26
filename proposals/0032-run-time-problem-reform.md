# (IE-0032) Run-Time Problem Reform

* Proposal: [IE-0032](0032-run-time-problem-reform.md)
* Discussion PR link: [#32](https://github.com/ganelson/inform-evolution/pull/32)
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: [IE-0001](0001-extensions-with-resources.md)
* Implementation: Partially implemented

## Summary

Kits (other than the built-in ones), and directory-format extensions, gain the
ability to issue run-time problems of their own for the first time. Each RTP
now has its own explanation file, written in Markdown format, which dynamically
generates HTML for the apps to show when a user runs into an RTP.

## Motivation

The process for issuing RTPs is currently very inflexible. Only `BasicInformKit`
and `WorldModelKit` are able to issue them, and even that division into two
piles is awkwardly done at present with linker tricks. New kits, such as
`DialogueKit`, also need to issue RTPs, but currently can't. It's also long
overdue to be able to issue RTPs from Inform source text, rather than leaving
this an ability only of kit code.

## Components affected

- [ ] No change to the natural-language syntax.
- [x] Major changes to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor changes to runtime kits.
- [x] No change to the Standard Rules. Phrase added to Basic Inform.
- [ ] No change to documentation.
- [x] Minor changes to the GUI apps.

## Impact on existing projects

None.

## Review of the current situation

At present, when the app runs a story file it is continuously monitoring the output to look for lines in the shape:

	*** Run-time problem TOKEN: ...

For example,

	*** Run-time problem P17: You can't divide by zero

In Inform v10, the `TOKEN` always takes the form `P1`, `P2`, ..., i.e., is always a P followed by a decimal number, but that is not actually required.

The app responds to seeing such a line by opening the Results pane opposite the Story, and showing a pre-made web page:

	HTMLDIRECTORY/RTP_TOKEN.html

where HTMLDIRECTORY is its directory of web pages. The files - currently `RTP_P1.html`, `RTP_P2.html`, ... and so on - are stored inside the app as static HTML pages. These are created in the `make transferoutcomepages` target, which is part of `make integration`. The work is done by a tool called `inrtps` using a single big text file of idiosyncratically formatted chunks of writing about the various RTPs.

## Changes needed for the apps

The app should now detect the following pattern instead:

	*** Run-time problem TOKEN: PATH

For example, the first line here should trigger the app:

	*** Run-time problem DividedByZero: INTERNAL/Inter/BasicInformKit/RTPs
	*** You can't divide by zero.

(The second and any subsequent lines are purely informational, and the app should not require them to be present.)

In the current state of the Inform repository, this is the only RTP which is printed in the new form, for the sake of testing, but the rest will be converted shortly. The following source will trigger it, for testing purposes:

	Lab is a room.

	Instead of jumping:
		showme 2 / 0.
		
	Test me with "jump".

The `TOKEN` is a run of one or more whitespace characters: in practice it will always be alphanumeric and begin with a letter, and contain in particular no punctuation or spaces.

The `PATH` tells the app where to find the explanation of this message. It identifies a directory in which a whole pile of RTP explanation pages can be found: the one we want is then `PATH/TOKEN.html`. Note that `PATH` may contain internal white space (though it may not end with white space): it continues to the end of the line it is printed on. For example:

	*** Run-time problem BadKeyholder: MATERIALS/Extensions/Emily Short/Locksmith-v7.i7xd/RTPs
	*** This is not a key which makes sense

In interpreting the path, the prefixes `INTERNAL/` and `MATERIALS/` should be expanded to the paths to the internal Inform resources and to the Materials folder for the current project, respectively. If these prefixes are not present, the path is an absolute pathname.

The app now locates Markdown source explaining the problem as, say, the file `BadKeyholder.md` or `DivideByZero.md` in the relevant folder. It then makes a call to inbuild to have an HTML version of the page generated on the fly:

	inbuild -internal INTERNAL -markdown-from MARKDOWNFILE -markdown-to HTMLFILE

Here, `INTERNAL` is the usual internal directory inside the app, always supplied when it calls inbuild. `MARKDOWNFILE` is the filename of the file (e.g. `DivideByZero.md`) referred to above. `HTMLFILE` is some scratch file whose filename the app can choose: it can be anywhere convenient to the app.

`inbuild` then generates the `HTMLFILE`. This process cannot fail short of some major system disaster (running out of memory, disc full, etc.), because all Markdown source is legal, and takes no appreciable time to run. The app then displays `HTMLFILE` as the web page to show in the Results panel.

Story files will only be set to print the quoted path in debug builds, since they are meaningful only when debugging in the app. A released story file, running outside of the apps, will therefore print something like this if it divides by zero:

	*** Run-time problem DividedByZero:
	*** You can't divide by zero.

## New method for kits to issue RTPs

At present only `WorldModelKit` and `BasicInformKit` can issue RTPs, and they do so by calling a function `RunTimeProblem`, e.g.:

	RunTimeProblem(RTP_LISTRANGEERROR);
	RunTimeProblem(RTP_BADCONTAINMENT, F, T);

The first parameter is always one of an enumerated set of constants, currently listed in `Definitions.i6t` inside `BasicInformKit` (despite the fact that many of them related to `WorldModelKit`).

These enumerated constants will go. Instead, calls will look like:

	IssueRTP("ListRangeError", "Attempt to use list item which does not exist.", BasicInformKitRTPs);
	IssueRTP("BadContainment", BadContainmentR, WorldModelKitRTPs, F, T);
	
The first parameter is the `TOKEN`. The second is explanatory text, which can be either a string, or a routine, such as this one:

	[ BadContainmentR par1 par2 par3;
		print "Attempted to put ", (the) par1, " in ", (the) par2, ".^";
	];

Or, the second parameter can also be `-1` to mean "give no explanatory text on the second line".

The third parameter is the `PATH`. Note that for these example calls, the path has been given using the constants `BasicInformKitRTPs` and `WorldModelKitRTPs`. Inform automatically defines such constants for all kits included in a build. For example, it may define:

	Constant Architecture32KitRTPs = "INTERNAL/Inter/Architecture32Kit/RTPs";
	Constant BasicInformKitRTPs = "INTERNAL/Inter/BasicInformKit/RTPs";
	Constant CommandParserKitRTPs = "INTERNAL/Inter/CommandParserKit/RTPs";
	Constant EnglishLanguageKitRTPs = "INTERNAL/Inter/EnglishLanguageKit/RTPs";
	Constant WorldModelKitRTPs = "INTERNAL/Inter/WorldModelKit/RTPs";

The net result is that any kit can issue its own RTPs. A kit author need only use a function call like the one above, and put a suitable explanation in Markdown format into the `RTPs` subdirectory of the kit.

## New method for extensions to issue RTPs

Extensions have never previously been able to issue RTPs. Given that directory-format extensions can contain kits, and kits can now issue RTPs, it would now be possible to use that ability to trigger custom RTPs from an extension. But there's also a more direct way which need not involve any kit or use of I6-syntax code at all.

A new phrase is now defined in Basic Inform. For example, one could write:

	issue the run-time problem "DialogueNestedTooDeep";
	say "*** Dialogue scene nesting became too complicated";

This is legal only in the source text of a directory-format extension, and only if the parameter (here `"DialogueNestedTooDeep"`) is a literal double-quoted text which contains the leafname of a Markdown file stored in the extension's `RTPs` subdirectory. For example, suppose the above source were in the extension `Dialogue Trickery by Marvin Queeg`: then the compiler would require the existence of the file:

	Whatever.materials/Extensions/Marvin Queeg/Dialogue-v3.i7xd/RTPs/DialogueNestedTooDeep.md

Assuming this is indeed one of the RTPs provided by the extension, the phrase then compiles to a suitable function call to `IssueRTP`, giving the correct RTP name-token and path.

## Withdrawal of inrtps

As noted above, in v10 the core Inform build process used a small tool called `inrtps` to manufacture the RTP and certain other outcome pages, such as the one displayed by the app if inform7 should unexpectedly crash.

This has now gone, and is no longer part of the repository. Its functions, such as they were, have been absorbed into `inbuild`. The HTML generated by it has been modernised to remove all use of `<font>` tags and layout-by-tables rather than by CSS, and to make all the tags tidily paired. The same CSS is used for these pages as for other index/extension documentation/etc. pages displayed in the app, so issues with needing to render differently on different platforms, and handling dark mode properly, should all be resolved.

Removing `inrtps` affects the makefile, though only for the targets `make integration` and `make transferoutcomepages` (and their variants `make forceintegration` and `make forcetransferoutcomepages`): this now calls `inbuild` not `inrtps`.

If you are using these app-integration make targets, you may be setting a variable called `INRTPSOPTS` in your `make-integration-settings.mk` file to configure them. If so, you now needn't bother to. It is no longer used for anything and is ignored.
