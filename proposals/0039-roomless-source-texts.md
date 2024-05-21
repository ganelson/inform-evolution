# (IE-0039) Roomless source texts

* Proposal: [IE-0039](0039-roomless-source-texts.md)
* Discussion PR link: [#39](https://github.com/ganelson/inform-evolution/pull/39)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0009](0009-dialogue-sections.md)
* Implementation: Implemented but unreleased

## Summary

To enable source texts which do not define a room to be compiled without
problem messages, with Inform creating an implicit room in which the action
will entirely take place.

## Motivation

This idea came from work on dialogue, where some dialogue-only stories
may have no concept of room or of spatial modelling at all. For years,
though, small Inform test cases have been creating bogus rooms, often
called `Laboratory`, for the test to take place in: small demonstrations
of Inform will no longer need to do that.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [x] Minor changes to the Standard Rules, though not Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None. This proposal affects only source texts which would previously have
been rejected with a problem message.

## Details

- A new use option `Use nameless room descriptions` modifies the looking action
  so that the boldface location part is not printed. (This affects only the
  `room description heading rule`.)  This option is available whether or not
  the source text defines rooms.

- If a source text does not define a room, the traditional problem message for
  not defining a room is not issued.

  Instead, Inform creates a room and starts the player in it; and the
  `use nameless room descriptions` option is automatically set.
  
  We'll call this the _implied room_.

- Any object located in the source text only as `here` — for example, in an
  assertion like `The old oak table is here.` — is placed in the implied room.

- The implied room has an empty description text, and the printed name ``Stage``.
  It can similarly be referred to in phrases or rules as `the Stage`. This name is
  chosen for consistency with the adjectives `on-stage` and `off-stage`: in
  a story with an implied room, an object will be `on-stage` if and only if
  it is enclosed by the `Stage`.
  
  Assertions can also refer to the Stage:
  
      The old oak table is in the Stage.

  However, if such assertions are impossible to reconcile with Stage being a
  room (`Jane is carrying the Stage`) then a problem message is issued. In
  particular, this catches the easily made mistake:
  
      The old oak table is on the Stage.
  
- Should the source text already define something called `stage`, then that
  definition is not overridden: the implied room will then be impossible to
  refer to, but that is unlikely to matter.
  
  If it does matter, the author can simply add this to the source text:
  
      My Super Stage Room is a room.
      Use nameless room descriptions.

  `My Super Stage Room` will then do just what an implied room would have done,
  but have a different name.

## Voidology theory

A consequence of the above change is that, for the first time, the empty
source text is now a valid Inform program (though it is not a valid Basic
Inform program, which needs a `To begin: ...` phrase). The empty source
text produces an empty sort of experience.
