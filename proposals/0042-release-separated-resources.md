# (IE-0042) Release along with separated resources 

* Proposal: [IE-0042](0042-release-separated-resources.md)
* Discussion PR link: [#42](https://github.com/ganelson/inform-evolution/pull/42)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: None
* Status: Implemented but unreleased

## Summary

A new `Release along with...` option which places image and sound files outside
of a blorbed story file, rather than inside, as is the default; and writes a
corresponding resource map file, in order to correlate them to their resource
numbers.

## Motivation

At present all image and sound resources are embedded within a blorbed story
file when a Release takes place. If that story file is to be used by a website
interpreter, this can make for an extremely large amount of base64-encoded
material to be processed: this makes fetching the page, and starting up the
interpreter, slower, and consumes more browser memory. For smaller projects
with only light illustrations, there is no real issue, but Inform projects
now exist with nearly 1 GB of audio-visual content.

This proposal therefore creates a new `separated resources` release option:
it is off by default, and therefore doesn't change the standard behaviour.

Interpreters will need work in order to be able to use this new feature, and
so there are also affordances for them to signal their limited compliance.
Because of this, the new feature may not benefit users much as yet, but it's
usually a good idea for Inform to provide a clear framework first in order
to move the ecosystem along, and that's what this proposal does.

One motivation for this feature is that people have already been using
workarounds to get to the same end, partly because of an unfortunate bug
(Jira [I7-2657](https://inform7.atlassian.net/browse/I7-2657)), partly
in order to use `.mp3` files, an audio format not supported by Glulx.

Inform itself has no objection to `.mp3` files:

	Sound of thunder is the file "Thunder.mp3".

In past builds, though, any attempt to release this story would produce an error
on the report page, saying that `.mp3` files were not supported. To get around this,
some people were renaming their files to have `.aiff` suffixes, then renaming them
back again after the release.

We want to make all this more straightforward. This proposal therefore includes
_minimal_ support for `.mp3` files, but only in `separated resources` mode,
where no Glulx support is needed.

## Details

The new behaviour is selected by:

	Release along with separated resources.

and of course affects only what happens on a Release run. This option replaces
the two previous options:

	Release along with separate figures.
	Release along with separate sounds.

Those are both now withdrawn, and produce a problem message suggesting a switch
to the new option.

- What always happens:

	- Copies of all picture files used (including the cover art) are made
		in the directories `Release/Figures`, which is created if necessary,
		i.e., if it does not already exist and if there is at least one picture.
	- And similarly for `Release/Sounds`.
	- A resource map file is created as `Release/X.resourcemap.json`, where `X`
		is the leafname of the blorb: e.g., `story.gblorb.resourcemap.json`.
	- The placeholder `[RESOURCEMAP]` is set to the relative URL of this of
		this file from the `Release` folder, i.e., at present just to its leafname.
		Note that if `separated resources` is not used, then this placeholder exists
		but is blank: an interpreter can use that to detect whether or not it
		should be using these separated resources.
	- The placeholder `[ENCODEDRESOURCEMAP]` is set to the contents of this JSON
		file, i.e., to the JSON-encoded resource map.

- What also happens if releasing with an interpreter:

	- The base64-encoded file bundled into the `Release/interpreter` directory
		is the Glulx story file, _not_ the Blorb.
	- A JavaScript-friendly form of the JSON encoding of the resource map is
		written to `Release/interpreter/jsresourcemap.js`. This consists of the
		values of the placeholders `[JSRESOURCEMAPTOP]` and `[JSRESOURCEMAPTAIL]`
		placed either side of the JSON content. The point of this is to avoid
		the need to load a JSON file, which might violate local `file:` browser
		security measures.
	- A warning is thrown about potential lack of support for `separated resources`,
		unless the interpreter has declared a new placeholder `[SUPPORTSRESOURCEMAP]`.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, except for the note about `separate figures` and `separate sounds` above.

### The format of the resource map

The resource map is a file written in JSON format. Here is a minimal example:

	[ {
		"id": 1,
		"url": "Figures/DefaultCover.jpg",
		"alttext": "",
		"format": "JPEG",
		"width": 680,
		"height": 680
	}, {
		"id": 3,
		"url": "Figures/mmatisse.jpg",
		"alttext": "Matisse's painting of Madame Matisse",
		"format": "JPEG",
		"width": 961,
		"height": 1200
	} ]

Formally, this is a JSON array (hence the outer `[` and `]`) which lists JSON
objects (each in `{` and `}`), each of which describes a single resource. Those
objects have the following compulsory fields:

- `id` (a JSON integer): the resource ID. Resource IDs match those used in the
	compiled story file, and are at least 1, with resource ID 1 used for cover
	art. At present, Inform never compiles resource maps which do not include
	resource 1, because it supplies default cover art even if the user does not,
	but interpreters should probably not absolutely rely on this. Resource IDs
	are unique, i.e., no two resources in the map can ever have the same ID,
	but not necessarily contiguous, as in the above example, where there is
	no resource 2.

- `url` (a JSON string): the relative URL to the resource, from the `Release`
	directory. Note that `/` is used as the directory separator, regardless
	of the operating system `inblorb` is running on (i.e., even if the resource
	map is generated from Windows, it will still use `/` not `\`). 

- `alttext` (a JSON string): text which can be shown by screen-readers or other
	accessibility tools for users unable to view images or hear sounds.

- `format` (a JSON string): one of the following:

	- `JPEG`, one of the picture formats
	- `PNG`, one of the picture formats
	- `AIFF`, one of the sound formats
	- `Ogg Vorbis`, one of the sound formats
	- `MIDI`, one of the sound formats
	- `MP3`, one of the sound formats
	- `unknown`, meaning that `inblorb` did not recognise the file format

In addition, there are optional fields which are present only for some formats:

Field        | `JPEG` | `PNG` | `AIFF` | `Ogg Vorbis` | `MIDI` | `MP3` | `unknown`
------------ | ------ | ----- | ------ | ------------ | ------ | ----- | ---------
`width`      | yes    | yes   | no     | no           | no     | no    | no
`height`     | yes    | yes   | no     | no           | no     | no    | no
`duration`   | no     | no    | yes    | yes          | no     | no    | no
`bps`        | no     | no    | yes    | yes          | no     | no    | no
`channels`   | no     | no    | yes    | yes          | no     | no    | no
`samplerate` | no     | no    | yes    | yes          | no     | no    | no
`version`    | no     | no    | no     | no           | yes    | no    | no
`tracks`     | no     | no    | no     | no           | yes    | no    | no

These are as follows:

- `width` (a JSON integer): image width in pixels — that is, the actual width
	of the image, not the desired width when the image is displayed.

- `height` (a JSON integer): image height in pixels, similarly.

- `duration` (a JSON integer): duration of the sound in centiseconds.

- `bps` (a JSON integer): bits of data needed per second, for a sound.
	Basically measures how much storage is needed: smaller is better.

- `channels` (a JSON integer): number of channels for a sounds, which is
	always at least 1.

- `samplerate` (a JSON integer): sample rate, measured per second, for a sound.
	Basically measures the quality of the encoding: higher is better.

- `version` (a JSON integer): normally 0, 1, 2, for the type of MIDI file used
	to make up the sound:
	
	- 0 means one track containing up to 16 channels;
	- 1 means one or more tracks, commonly each with a single channel, making up a single tune;
	- 2 means one or more tracks, where each is a separate tune in its own right.

- `tracks` (a JSON integer): number of tracks in the MIDI file, which is at least 1.

### How `.mp3` files are handled

Support here is limited. Essentially, until and unless the Glulx format adopts MP3,
all that Inform is doing is to provide a facilitated workaround. Because of that,
any Release involving `.mp3` files produces a warning notice: but it does not, as
it previously would have done, halt with an error. Note that:

- `.mp3` files appear in a resource map with the format `MP3`, and are copied
	into the `Release/Sounds` directory if the release is along with `separated resources`,
	just like other sounds. 

- `.mp3` files are encoded in a blorb as data chunks of type `BINA`, that is,
	as if they were binary data files. Obviously, they will not be playable,
	but at least this masquerade ensures that the Blorb is legal.

### The `[SUPPORTSRESOURCEMAP]` placeholder

This placeholder should be set only in the `(manifest).txt` file for an interpreter.
If it is not set, then on any Release run of Inform with `separated resources` which
uses the interpreter, `inblorb` will produce a warning:

	The release did succeed, but with warnings:

	- this interpreter does not support 'separated resources'

If the placeholder is set to some general text, then this is used to change the
warning. For example, if `(manifest).txt` contains:

	[SUPPORTSRESOURCEMAP]
	sounds are not supported
	[]

then users will see:

	The release did succeed, but with warnings:

	- this interpreter has limited support for 'separated resources': sounds are not supported

If the placeholder is set to:

	[SUPPORTSRESOURCEMAP]
	Yes
	[]

then no warning will be issued, and the assumption is that the interpreter fully
supports this feature.

### Additions to the Blurb language

`Blurb` is a configuration language used by `inform7` to pass instructions to
`inblorb`. The `separated resources` feature is supported by four new Blurb
instructions:

- `copy pictures to "PATH"`. Places copies of all the picture files to the
	given path.

- `copy sounds to "PATH"`. Similarly for sounds.

- `resource map to "FILE"`. Filename to write the JSON resource map file to.
	If this command is not given, no file is written.

- `js resource map to "FILE"`. Similarly, but for the JavaScript-wrapped version:
	see above.

