# (IE-0029) Basic IO

* Proposal: [IE-0029](0029-basic-io.md)
* Discussion PR link: [#29](https://github.com/ganelson/inform-evolution/pull/29)
* Author: Dannii Willis
* Language feature name: None
* Status: Accepted
* Related proposals: [IE-0017](0017-apps-and-extensions.md)

## Summary

Extend the basic IO functionality of Basic Inform, including incorporating
Basic Screen Effects.

## Motivation

As part of [IE-0017](0017-apps-and-extensions.md) the former built-in extensions
have been removed. Most of them will be available in the Public Library, but
[Basic Screen Effects](https://github.com/ganelson/inform/blob/r10.1/inform7/Internal/Extensions/Emily%20Short/Basic%20Screen%20Effects.i7x)
is so close to essential that it was decided it would be better to just incorporate
its contents into Basic Inform. A few extra phrases will also be added.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor change to BasicInformKit.
- [x] Minor changes Basic Inform.
- [x] Minor addition to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Projects currently relying on Basic Screen Effects may need to slightly change
some phrases, but where possible the new Basic Inform implementation will be made
compatible.

## Specifics

Basic Screen Effects has the following functionality:

- measuring the screen/game window
- clearing the screen/specific windows
- customising the status line
- key press phrases
- pausing the game activity
- showing boxed quotations
- centering text
- coloured text (Z-Machine only)

These are generally very useful features, and are also quite basic. Incorporating
them into Basic Inform shouldn't be an issue as it is unlikely anyone would want
a very different implementation of these features. We can also now make coloured
text supported in Glulx as well as Z-Machine. However we don't need to incorporate
absolutely all of BSE; in particular having a whole activity for pausing the game
seems like an overcomplication.

While BSE lets you read one keypress event, it has no phrases for reading text.
Neither does Basic Inform at present! Accepting a line of text from the user is
a pretty fundamental operation that is supported by almost every other language.
So it is proposed that we also add phrases to read text and to read and parse a
number from the user.
