# (IE-0036) Extension licencing

* Proposal: [IE-0036](0036-extension-licencing.md)
* Discussion PR link: [#36](https://github.com/ganelson/inform-evolution/pull/36)
* Authors: Graham Nelson
* Status: Draft
* Related proposals: --
* Implementation: None as yet

## Summary

Provides for extensions, and also projects and other Inform resources, to say explicitly under what legal terms they can be used.

## Motivation

The existing Inform documentation is both vague and cavalier about presuming that extensions are shared under some form of Creative Commons licence. Whether or not that is a good choice is open to debate, but it's clear that in the more rigorous copyright climate of 2024, we need something more explicit. In particular, authors need to know they can use Public Library extensions safely.

## Components affected

- [x] Minor changes to the natural-language syntax.
- [x] Minor changes to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None, unless they have for some reason defined a `COPYRIGHT` command already.

## Source text changes

The following new special meaning is provided for assertion sentences:

	The licence/license/copyright/origin URL/rights history for this extension/story is <quoted-text>.

`licence` = `license`, so this can make four textual settings.

1) "for this extension" can be used only in an extension, and the same setting can be made at most once in each extension.

2) "for this story" can be used only in the main source text, and the same setting can be made at most once. The values are written into four new bibliographic variables:

       story licence
       story copyright
       story origin URL
       story rights history

   These then appear on the Library Card in the Index. (To think about: should they appear in the iFiction record?)

3) Either for extensions or for stories, the defaults are:

       licence = "Unspecified"
       copyright = "<author name> <year of compilation>"
       origin = ""
       rights history = ""

4) The licence text must be _either_ `Unspecified` _or_ one of the machine-readable licence codes listed at: [https://spdx.org/licenses/](https://spdx.org/licenses/)

5) The copyright text must have the form `<name> DDDD` or `<name> DDDD-EEEE`, where DDDD >= 1980 and EEEE > DDDD, and must not contain a `©` symbol or the substring `(c)` or `(C)`.

6) The origin URL text, if given, must be a URL beginning `http://` or `https://`.

7) The rights history can be any text.

For example:

	The licence for this extension is "CC-BY-4.0".
	The copyright for this extension is "Emily Short 2006-2024".
	The origin for this extension is "https://www.emshort.com/inform".
	The rights history for this extension is "Adapted by permission from sample code by Adam Cadre in 2006."

## JSON metadata changes

In the JSON specification used to describe all Inform resources (kits, extensions, projects, etc.), `<resource-metadata>` gains a new optional field:

	?"rights": <legal-metadata>

where `<legal-metadata>` is specified as follows:

	<legal-metadata> ::= {
		"licence": string,
		"rights-owner": string,
		"date": number,
		?"revision-date": number,
		?"origin-url": string,
		?"rights-history": string
	}

For example:

	"rights" {
		"licence": "CC-BY 4.0",
		"rights-owner": "Emily Short",
		"date": 2006,
		"revision-date": 2024,
		"origin-url": "https://www.emshort.com/inform",
		"rights-history": "Adapted by permission from sample code by Adam Cadre in 2006."
	}

For extension and story JSON, Inform will automatically set this on the basis of the settings made in source text, in the obvious way. For kits, authors are free to write their own, but if a kit is included as part of an extension then it is not allowed to have a `"rights"` object is in its own metadata since it falls under the umbrella `"rights"` object of the extension.

## Documentation

The Inform documentation will give guidance on what is good practice - in particular, suggesting possible licences. But this will be guidance, not binding on users.

The Public Library will only accept extensions whose licences are on a white-listed set, though, and we should probably say what that set is in the documentation. I propose the following whitelist as a starting point:

- `CC0-1.0`, i.e., CC0
- `CC-BY-4.0`
- `Unlicense`, i.e., The Unlicense
- `MIT-0`, i.e., MIT No Attribution
- `Artistic-2.0`, i.e., Artistic License 2.0

I suggest that documentation encourage people to use the least restrictive licence possible, but make the point that the Public Library will, as a matter of policy, not accept an extension where the curators feel that something unethical has been done. Although a licence might allow the author's name to be removed, for example, or for an extension to be hijacked to make some political point (e.g. about gender), we would choose not to host the result on the PL, even though it was legal as a derivative work by the terms of the licence.

## The `VERSION` command

As now, the text output by the `VERSION` command will be the place where the contribution made by extensions is credited.

For example:

	Hypothetical Extension by Emily Short v2.3.12 is included here under the terms of CC-BY 4.0. Adapted by permission from sample code by Adam Cadre in 2006. See: https://www.emshort.com/inform
	...
	For information about and links to full text of licenses, see: https://spdx.org/licenses/

The command verb `COPYRIGHT` will now be aliased to `VERSION`.

## Releasing changes

All releases will now generate a file LICENSES.txt which essentially consists of the `VERSION` output.

If the release is along with a website, this will be linked in the website.
