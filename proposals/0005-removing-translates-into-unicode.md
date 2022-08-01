# (IE-0005) Removing translates into Unicode

* Proposal: [IE-0005](0005-removing-translates-into-unicode.md)
* Discussion PR link: [#5](https://github.com/ganelson/inform-evolution/pull/5)
* Authors: Graham Nelson
* Language feature name: None
* Status: Draft
* Related proposals: None
* Implementation: None

## Summary

Providing Unicode character names without slowly loading large cumbersome
extensions, as has to be done at present.

## Motivation

Inform currently contains an obscure language feature enabling Unicode code
points (specified in decimal) to be given names. For example:

	latin small letter s with acute translates into Unicode as 347.
	dentistry symbol light down and horizontal with triangle translates into Unicode as 9156.

In practice this feature is used only by the extensions:

	Unicode Full Character Names by Graham Nelson
	Unicode Character Names by Graham Nelson

which make a huge number of such translations. Those extensions have to be
explicitly included by the user to have any effect, and they are slow to read
(because huge) and just saturate the compiler's symbols table. Indeed, the only
reason there are two such extensions is because the full one is so large. We can
do better.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Low. Projects including these two extensions will have to remove the `Include`
sentences doing so, but will then work as before. Projects making their own
`translates into Unicode` sentences will need to remove those sentences,
but very few people will be affected by that.

## Proposal

1. These two extensions will be removed.

2. The Inform installation will contain a stand-alone file of Unicode character
names. This might even be a straight copy of [the Unicode standard name list](https://www.unicode.org/Public/14.0.0/ucd/NamesList.txt),
though it may be sensible to continue to restrict Inform to the Basic Multilingual Plane characters from that.
The file will be read in to the compiler only on demand, that is, when the
compiler finds that it has to make sense of `unicode ...` because of text
such as:
```
	"[unicode Latin capital letter L with stroke]odz Churchyard"
```

3. A problem message will reject any usage of `translates into Unicode`.
