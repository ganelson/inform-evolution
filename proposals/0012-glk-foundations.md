# (IE-0012) Glk foundations

* Proposal: IE-0012
* Reference link: [#12](https://github.com/ganelson/inform-evolution/pull/12)
* Authors: Dannii Willis
* Language feature name: None
* Status: In progress
* Related proposals: None
* Implementation: None

## Summary

Incorporate code from the extension Glulx Entry Points.

## Motivation

The extension [Glulx Entry Points by Emily Short](https://github.com/ganelson/inform-public-library/blob/main/docs/resources/Extensions/Emily%20Short/Glulx%20Entry%20Points.i7x) has become an almost essential part of the Inform ecosystem. Almost, but not fully, and there are some extensions which are not compatible with it, most notably [Unified Glulx Input by Andrew Plotkin](https://github.com/i7/extensions/blob/9.3/Andrew%20Plotkin/Unified%20Glulx%20Input.i7x). Many extensions depend on GEP, which means that you can't use them if you need to use UGI, even though nothing may directly conflict between those extensions and UGI itself.

It is proposed that most of GEP be incorporated into the core Inform project, that is, the parts of GEP that are foundational and universally applicable. The parts of GEP that provide just one possible way to implement a feature will be split off into new extensions. This should allow more mixing and matching between formerly GEP-based extensions and UGI-based extensions.

### Contents of Glulx Entry Points

GEP includes the following features:

1. A kind for Glk events
2. Phrases for checking interpreter features (gestalts)
3. Glk object recovery: to fix memory references after restarting/restoring
4. Exposing a few I6 variables
5. Event handling rulebook
6. Command replacement support
7. Miscellaneous:
    1. Redraw the status line
    2. Print the prompt

## Components affected

- [ ] Minor changes to the natural-language syntax (see point 4 above.)
- [ ] Major changes to inbuild.
- [x] Change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Change to runtime kits.
- [x] Changes to the Standard Rules and Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps, when downloading or installing extensions.

## Impact on existing projects

Glulx Entry Points was removed from Inform 10.1 to minimise disruption, but anyone who has installed it manually or any extensions that depend on it may have issues until those extensions are updated.

## Proposal

It is proposed that Inform incorporate the following extensions (which were previously split out of Glulx Entry Points):

### [Glulx Definitions](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Glulx%20Definitions.i7x)

The Glulx Definitions extension extracted out the phrases that checked for Glk interpreter features, and added Glulx phrases too. It also contains the definition of the glk event kind.

### [Glk Object Recovery](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Glk%20Object%20Recovery.i7x)

This extension replaces the convoluted multiple phase-based GGRecoverObjects/IdentifyGlkObject API that dates back to I6 with a simpler process that directly calls rulebooks for each phase of GGRecoverObjects. The logic is the same, it just does things in a more natural way for Inform 7.

### [Glk Events](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Glk%20Events.i7x)

This extension is an alternative to the original I6 HandleGlkEvent API inherited by I7 and GEP. One of the main flaws with that API is that the HandleGlkEvent hook is only called by the three standard input functions: VM_KeyChar, VM_KeyDelay, and VM_ReadKeyboard. If something else (usually an extension) ever calls glk_select by itself then the hook won't be called. So what this extension does instead is intercept calls to `glk_select`. This means the author (or extension authors) has a single place to handle all events, including converting events from one type to another, cancelling the event (convert it to a null event), etc.

### GEP leftovers

6. Command replacement support

Command replacement should be made into a new extension.

7. Miscellaneous:
    1. Redraw the status line
    2. Print the prompt

These could be brought into the Standard Rules, or perhaps into Basic Screen Effects?