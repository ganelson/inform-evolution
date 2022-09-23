# (IE-0012) A new GlkKit

* Proposal: IE-0012
* Reference link: [#12](https://github.com/ganelson/inform-evolution/pull/12)
* Authors: Dannii Willis
* Language feature name: None
* Status: Draft
* Related proposals: None
* Implementation: None

## Summary

Move code from the Glulx Entry Points and other extensions into core as a new Glk Kit.

## Motivation

The extension [Glulx Entry Points by Emily Short](https://github.com/ganelson/inform-public-library/blob/main/docs/resources/Extensions/Emily%20Short/Glulx%20Entry%20Points.i7x) has become an almost essential part of the Inform ecosystem. Almost, but not fully, and there are some extensions which are not compatible with it, most notably [Unified Glulx Input by Andrew Plotkin](https://github.com/i7/extensions/blob/9.3/Andrew%20Plotkin/Unified%20Glulx%20Input.i7x). Many extensions depend on GEP, which means that you can't use them if you need to use UGI, even though nothing may directly conflict between those extensions and UGI itself.

It is proposed that most of GEP be incorporated into the core Inform project as a new Glk Kit, that is, the parts of GEP that are foundational and universally applicable. The parts of GEP that provide just one possible way to implement a feature will be split off into new extensions. This should allow more mixing and matching between formerly GEP-based extensions and UGI-based extensions.

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

It is proposed that GlkKit contain the following:

### From BasicInformKit's [Glulx.i6t](https://github.com/ganelson/inform/blob/master/inform7/Internal/Inter/BasicInformKit/Sections/Glulx.i6t)

A lot of this extension should be moved to GlkKit. Things like the Infglk and Keyboard Input sections definitely should be, but I'm less sure about some of the more Glulx focused sections (and in particular how they interact with Inform's C output mode.)

### [Glulx Definitions](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Glulx%20Definitions.i7x)

The Glulx Definitions extension extracted out the phrases that checked for Glk interpreter features, and added Glulx phrases too. It also contains the definition of the glk event kind, "g-event". Should we consider renaming this to "glk event"?

### [Glk Object Recovery](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Glk%20Object%20Recovery.i7x)

This extension replaces the convoluted multiple phase-based GGRecoverObjects/IdentifyGlkObject API that dates back to I6 with a simpler process that directly calls rulebooks for each phase of GGRecoverObjects. The logic is the same, it just does things in a more natural way for Inform 7.

The rulebooks defined here begin with "glulx", but really relate to "glk". Should they be renamed?

### [Glk Events](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Glk%20Events.i7x)

This extension is an alternative to the original I6 HandleGlkEvent API inherited by I7 and GEP. One of the main flaws with that API is that the HandleGlkEvent hook is only called by the three standard input functions: VM_KeyChar, VM_KeyDelay, and VM_ReadKeyboard. If something else (usually an extension) ever calls glk_select by itself then the hook won't be called. So what this extension does instead is intercept calls to glk_select. This means the author (or extension authors) has a single place to handle all events, including converting events from one type to another, cancelling the event (convert it to a null event), etc.

### [Alternative Startup Rules](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Alternative%20Startup%20Rules.i7x)

This probably shouldn't strictly be part of GlkKit, but it would be good to incorporate at the same time.

This extension is the result of my belief that the standard I7 startup code seems somewhat poorly organised. In particular, despite the existence of the starting the virtual machine activity, a bunch of actions are put into the startup rules. So this extension shifts them into the appropriate stages of the starting the virtual machine activity. In particular, the before rules should not use any IO functions (it doesn't do anything to catch errors printed at that stage, though as discussed in earlier emails that would probably be a good idea, and wouldn't actually be very hard to add.) The for rules are for setting up IO systems, and the after rules is a good place for low level miscellaneous startup stuff, including a good place for extensions to place their code. The startup rules can then be more for the author's own code. And splitting up the rules into the three activity rules also means that less needs to be done to order them manually, as they'll have a natural ordering.
It also breaks up VM_Initialise which does 5 separate tasks, some of which an extension might want to replace without touching the others.

The relevant parts of 10.1 have changed significantly since 6M62 so it can't simply be incorporated, but the principles of where to place rules should be followed for all the current rules.

These changes should probably also be made for Z-Code output too.

### GEP leftovers

6. Command replacement support

Command replacement should be made into a new extension.

7. Miscellaneous:
    1. Redraw the status line
    2. Print the prompt

These could be brought into the Standard Rules, or perhaps into Basic Screen Effects?

## Questions

1. Should non-Glk but Glulx related stuff be brought over? Perhaps even the Glulx gestalt phrases should be put in BasicInformKit instead of GlkKit?
2. Whether anything should be renamed
3. Some of the extensions make use of I6's `Replace` statement to intercept functions such as `glk_select`. These functions are currently defined in the Infglk section, which incorporates the infglk.h file, I think unchanged. Should we continue to use Replaces, or should we be more willing to modify things within the Infglk section?

    For example, command replacement currently sets the I6 array `buffer`. But an extension could in theory call `glk_request_line_event` with a different array (I don't know if any do). We could make command replacement more rigourous by intercepting calls to `glk_request_line_event` and saving the buffer it was used with, and then use that buffer in the command replacement extension.

5. Would this proposal actually allow for UGI to be used at the same time as some formerly GEP-based extensions? If not, what would need to change?
