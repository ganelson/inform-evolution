# (IE-0005) Removing translates into Unicode

* Proposal: [IE-0005](0005-removing-translates-into-unicode.md)
* Discussion PR link: [#5](https://github.com/ganelson/inform-evolution/pull/5)
* Authors: Graham Nelson
* Language feature name: None
* Status: Draft
* Related proposals: None
* Implementation: Implemented but unreleased

## Summary

Providing Unicode character names without slowly loading large cumbersome
extensions, as has to be done at present, and allowing recently-standardised
emoji and other Supplementary Multilingual Plane characters for the first time.

## Motivation

Inform currently allows Unicode characters to be placed in quoted text by
giving their names: for example,

	"A sign on the door cryptically reads: [unicode black chess knight]."

This is equivalent to writing:

	"A sign on the door cryptically reads: [unicode 9822]."

since the black chess knight character occupies code point 9822 (decimal, that
is: in hexadecimal, 265E) in Unicode. (And if you can find a way to type this
character directly, you can even use it in quoted text just like any other letter.)

The ability to give names of characters has two limitations at present:

* A name can only be used if it has been declared with a sentence such as
"Black chess knight translates into Unicode as 9822".
* Only plane 0 of Unicode has been available up to now: that is, only values
from 0 to 65535.

In practice, of course, users never typed their own `translates into Unicode`
sentences, but instead included the extensions:

	Unicode Full Character Names by Graham Nelson
	Unicode Character Names by Graham Nelson

which made a huge number of such translations. Those extensions are slow to read
(because huge) and just saturate the compiler's symbols table. Indeed, the only
reason there are two such extensions is because the full one is so large. (And
even that only covered names from Unicode 4.1, which is now very out of date.)
We can do better.

## Components affected

- [x] Minor changes to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Low. Projects including these two extensions will have to remove the `Include`
sentences doing so, but will then work as before.

## Proposal

1. The extensions `Unicode Full Character Names by Graham Nelson` and
`Unicode Character Names by Graham Nelson` are removed from the Inform
distribution.

2. A problem message rejects any usage of `translates into Unicode`, telling
the user there is no need to write this sentence any longer.

3. Instead, when necessary, the Inform compiler reads the file:

	Internal/Miscellany/UnicodeData.txt

from its own installation. This file is an exact copy of the basic data file of
the same leafname included in version 15.0.0 (dated September 2022) of the Unicode
Consortium standard: being used unmodified, its distribution within Inform is
within the terms and conditions granted by the Consortium.

4. Inform recognises Unicode character names case-insensitively, but otherwise
these names must exactly match those in the Unicode standard. Thus `[unicode WHITE CHESS BISHOP]`
and `[unicode white chess bishop]` are equivalent.

5. If compiling to the Z-machine, which is a 16-bit virtual machine, Inform
continues only to allow code points in Unicode plane 0, that is, in the range
0 to 0xFFFF.

6. However, if (as is true by default) Inform is compiling to a 32-bit VM such
as Glulx or to C, then Inform allows plane 1 as well, i.e., up to 0x1FFFF.
Allowing plane 1 for the first time means that many emoji and other exotic
script characters are now legal:
	```
	[unicode Egyptian hieroglyph o030]
	[unicode tetragram for vastness or wasting]
	[unicode signwriting floorplane shoulder hip move]
	[unicode playing card queen of diamonds]
	[unicode robot face]
	[unicode fish cake with swirl design]
	```
Inevitably, you're at the mercy of what fonts on your computer can actually
display. On the author's current MacOS installation, "signwriting floorplane
shoulder hip move" turns out to be unavailable, for example, though the rest worked.

7. Inform recognises all names of characters in Unicode version 15.0.0 except
for those with Unicode category `Cc`: control characters left over from ASCII
and which have no name according to Unicode. Just two control characters have
names in Inform:  `TAB` (9) and `NEWLINE` (10). So `[unicode tab]` is allowed,
for example.
