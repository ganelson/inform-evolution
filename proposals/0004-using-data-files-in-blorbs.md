# (IE-0004) Access to data files embedded in blorbs

## Motive

The Blorb format and the glk interface layer already implement the ability for
blorb files to include read-only data files (which can contain text, or
IFF-style forms, or simply binary data). Existing Glulx interpreters, including
Quixe running on Javascript, can already read from such resources.

However, Inform users cannot at present use this functionality. Inform 7 has
a concept for read/write external text files, but not for data resources on a
par with the "figures" and "sounds" it does recognise. So Inform authors cannot
include data resources in their projects.

How useful this facility is (given that it is read-only, and that the files
have to be embedded in a blorb) remains to be seen, but it seems a pity for this
functionality to be unavailable when it would be relatively simple to provide.

## Inform changes

1.1. There will be a new kind of value, "internal file" - compare the existing kind
"external file". Neither will conform to the other and there is no broader kind
called "file" to which they both conform. Like "external file", "internal file"
will be an enumeration whose values are all determined at compile-time.

1.2. External files are now created with sentences like:

	The File of Glaciers is called "ice".

This will not change. But we will now also read something like:

	The internal data file of glaciation areas is called "ice extents.usgs".

Note "internal data". We will also allow "internal text" and "internal form";
these three variants correspond to the three data file formats used by Blorb,
that is, BINA, TEXT and FORM.

1.3. Just as users currently store figures and sounds in the Figures and Sounds
subdirectories of the materials directory for a project, they will store their
internal files in the Data subdirectory.

1.4. Directory-format extensions (see IE-1) can similarly store internal files
in the corresponding location, Materials/Data.

1.5. Inform does not police the filename extension or format for this data, and
will simply trust the user.

1.6. The phrase "contents of (F - internal file)" returns a pointer to a list
whose entries are the words of the file. This list is read-only (it cannot be
sorted, for example - a run-time problem would be generated), but otherwise
a perfectly normal list.

1.7. And similarly for "contents of (F - internal file) as bytes".

## BasicInformKit changes

2.1. A new section, "Internal Files.i6t", will be added to BasicInformKit.
Its content will be within #ifdef TARGET_GLULX, that is, will not be available
on the 16-bit architecture (i.e., the Z-machine).

The code in this section manages data access for all internal files, opening
them and seeking the correct position on demand.

2.2. "internal file" is an enumeration, but as with "figure" and "sound", the
runtime representation of an Inform constant such as "file of glaciation areas"
is a resource ID number.

2.3. InternalFileReadWord(id, n) will return word n from that file, counting
from 0. Similarly for InternalFileReadByte(id, n). If n lies outside the extent,
a run-time problem is thrown.

2.4. The function InternalFileLength(id) returns the extent in bytes.

2.5. The internal representation of a list will change to include (i) a read-only
flag, and (ii) a way to say "the contents of this list are in the data file
with the following resource ID".

## Inblorb changes

3.1. The following command will be added to the Blurb language:

	data [ID or NAME] "FILENAME" type TYPE

This is analogous to the existing picture command, in that either a numerical
ID can be given for the resource ID, or if not inblorb will choose one; and
if a NAME is given, will print out an I6 constant declaration of that. (This
is no longer very useful, but since we have this for figures and sounds, we
will have it for these too.) "type" must be one of BINA, TEXT or FORM, in
full caps.
