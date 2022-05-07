# (IE-0003) Dividing source text into multiple files

## Motive

Very large projects have source text which becomes unwieldy to store in a
single file, and source control is certainly easier if it can be broken into
multiple files. On really large projects, the Inform app may not be the ideal
editor for such a file or files, either.

Some people get around this by creating bogus extensions, but that isn't really
what extensions are for.

## Existing changes which are implemented but not documented yet

1. inform7 new command-line setting, -source X, tells inform7 to read the file X
instead. This can be outside of the project folder entirely. However, it
is still of course a single file; and the apps do not use it, so the only way
to access this from the apps at present is to use a place an "inform7-settings.txt"
file in the Materials folder, and to put "-source X" into that file.

2. Alternatively, the user can create a subfolder of the Materials folder called
"Source". This must contain a file called "Contents.txt" which lists the source
files to read, one per line. For example, it might read:
```
	# Source is split into several files as follows
	Athens.i7
	Asia Minor.i7
	Pergamon.i7
```
(The "#" means a comment line and is, of course, optional.) inform7 then makes
the main source text be the concatenation of the text in these files, which must
all similarly be in the Source subfolder of the materials folder.

## Questions

Is the above sensible?

How should the apps handle this? Possible things the apps might do:

(a) Have a per-project setting for "use in an external editor". On MacOS, for
example, you could tell Inform.app you want to edit in BBEdit. If so, the
Source panel does not appear at all, and the app refers links to source over
to that other editor. (I think most text editors support something like the
shell command "bbedit whatever.ni:1242", meaning, open whatever.ni if it is
not now open, and go to line 1242.)

(b) Make the Source panel understand about multiple-source-file projects as
in (2) above, and use e.g. a drop-down tab to choose which one is currently
being edited. Source links then select the relevant file before jumping to
the relevant line.
