# (IE-0010) Concepts

* Proposal: [IE-0010](0010-concepts.md)
* Discussion PR link: [#10](https://github.com/ganelson/inform-evolution/pull/10)
* Authors: Emily Short and Graham Nelson
* Language feature name: `concepts`
* Status: Draft
* Related proposals: [IE-0009](0009-dialogue-sections.md)
* Implementation: Implemented but unreleased

## Summary

A new top-level kind of object, `concept`, useful for representing abstract
concepts like "forgiveness" or "Italian cookery" which have no physical
existence in the world model.

This proposal was redrafted in early October 2022 in the light of
experience. All functionality here is implemented in draft on the `master`
branch of the repository, but not in any release branch as yet.

## Motivation

In some sense most data is abstract: the number `17` is not located at any
specific point in the Universe in the way that, say, Spain is. On the face
of it, the natural way to represent an abstract idea like `forgiveness`
is to create a kind of `value`:

	An virtue is a kind of value. Forgiveness, patience and a preference
	for sauce over gravy are virtues.

This is often just what's needed, but can be awkward in situations where
we want a single value to hold either a physical concept or an abstract one.
No kind can represent "either a `thing` or a `virtue`" if `virtue` is a kind of
`value` rather than a kind of `object`. Also, Inform has slightly better
facilities for parsing names of objects than values.

As result, authors have often used things to represent abstractions:

	An virtue is a kind of thing. Forgiveness, patience and a tolerance
	of 1970s progressive rock double-albums are virtues.

But `thing` is conceptually wrong here, and means that these objects, even
though they remain out of play, would be picked up by something like
`now every thing is in the Nursery`.

This simple proposal therefore creates a new top-level kind of `object`
called `concept`, and a new action `talking to it about` which models an
attempt to talk about the concept with somebody.

## Components affected

- [x] Minor change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor change to `BasicInformKit`.
- [x] Minor change to the Standard Rules, no change to Basic Inform.
- [x] Minor change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Any project using the keyword `concept` or an action `talking to...` is
potentially affected, since these new names will then clash.

In case this causes problems, the existence of concepts can be removed by
switching off the language feature `concepts` in a project's JSON metadata.

## Proposed changes

`concept` is a new top-level kind of object. (That is, its superkind
is `object` itself.) Concepts are created with assertions in the usual way, and can
have names recognised by the command parser as usual, too. So, for example:

	The paranormal is a concept. Understand "supernatural" as the paranormal.

Concepts have no physical existence, of course, so it is meaningless (and
an error) to write something like

	The paranormal is a concept in the Dining Room.

By default there are no concepts, and none are created automatically.

### Concepts as knowledge

Note that concepts are free to be used by Inform projects (or extensions) as
a convenient way to represent abstract ideas. This proposal intentionally
keeps "concept" as open-ended as possible, and simple knowledge-representation
systems are very easy to set up with existing Inform features. For example,
the following makes a set of concepts into things that characters might or
might not know about:

	An idea is a kind of concept. Knowing relates various people to various
	ideas. The verb to know about means the knowing relation.
	Bohr knows about quantum physics.

produces:

	> RELATIONS
	Knowing relates various people to various ideas:
	  Bohr  >=>  quantum physics

And our dialogue system in this proposal could then make good use of such
concepts:

	(about an idea)

	Bohr (now the player knows about quantum physics): "You see, it's all
	a matter of complementarity."

### Concepts as choices

For conversation, concepts are useful because people often talk about or act
upon ideas rather than physical things or places. You can contemplate "revenge"
as an option, or discuss "quantum physics", but you can't pick either of them
up or put them on a table.

In choice-based interactive fiction, in particular, concepts are useful to
represent things the player might want to talk about or act upon. We might
imagine them arising in a variety of ways:

(a) In a command parser-based game, commands like **TALK ABOUT CONCEPT**,
could be parsed as asking for conversation about a given concept object.
See the `talking about` action provided in [IE-0009](0009-dialogue-sections.md).

(b) In a parser game augmented with embedded menus, we might see options 1-3
of things to say next, but also be able to freely type **ASK ABOUT CONCEPT**
in order to get a fresh menu on the new subject matter. (Cf. "Pytho's Mask".)

(c) In a purely choice-based game, a "topic inventory" - represented in Inform
as a list of concepts, perhaps, or a knowledge relation like the one above -
could provide a dynamically changing range of extra options at any turn.
(Cf. "Subsurface Circular".)

(d) In a game relying on external NLP tools or services, selecting topics based
on intent and entity recognition (and similar features) provided by the external
processing tool could be mapped onto concept objects in Inform.

### Concepts in the command parser

Being objects, concepts have names which can be parsed by a command parser, so
all of the usual `Understand` machinery works as expected:

	Wetness is a concept. Understand "dampness" as wetness.

And we can set up actions on concepts:

	Pondering is an action applying to one concept.

	Carry out pondering a concept (called X):
		say "You think deeply about [X]."

	Understand "ponder [concept]" as pondering.

## Minor supporting changes in the Inform language

The above example action required two small changes to the Inform language:

1. Allow `applying to` in action definitions to name kinds of object which are
not kinds of thing. Thus, `applying to one region`, `applying to one concept`.
These previously threw problem messages.

2. Make the token `[D]`, where `D` is a description of a kind of object which is
not a kind of thing, automatically infer `any`. In particular, `[concept]` would
always mean `[any concept]`, i.e., there would never be any attempt to apply
touchability or visibility to concepts.

Neither change affects the meaning of existing source code.
