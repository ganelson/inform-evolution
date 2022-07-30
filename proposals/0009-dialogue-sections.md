# (IE-0009) Dialogue Sections

* Proposal: [IE-0009](0009-dialogue-sections.md)
* Authors: Emily Short and Graham Nelson
* Language feature name: `dialogue`
* Status: Draft
* Related proposals: [IE-0010](0010-concepts.md)
* Implementation: None

## Summary

An extensible system for dialogue, combining natural-language playtext in
"dialogue sections" of the source text with an active runtime component,
the "director", implemented with a new kit, `DialogueKit`.

## Motivation

This is an ambitious proposal, marking the largest expansion of Inform in many
years. While Inform is good at modelling the physical world, it has only
sketchy support for conversation "out of the box". Many extensions have been
created to provide such features over the years, but inevitably lack the
elegance and ease of use of a built-in language feature.

The need to do something about this is now pressing. To many contemporary
IF authors, especially in the commercial sector, interactively branching
narratives build largely on dialogue have become steadily more important.
In the 2020s, Inform should make it as easy as possible to write sophisticated
interactive dialogues, in the same way that in the early 2000s it aimed to
offer a step change in convenience for authors of command-parser IF.

A guiding principle of Inform's early design as a language was to imitate
the conventions of printed books or magazines: hence, for example, the syntax
of the Table data structure, which copies the look of data tables in
non-fiction books or scientific papers. The obvious analogue for long runs
of dialogue is to imitate printed playtext, as found in performance scripts
of plays, or in printed books collecting such plays together. (This is nothing
new to IF authors, since systems such as Ink, Twine and Prompter all use
similar approaches.)

Playtext has different punctuation and layout from regular prose, so in this
proposal in can be used only under section headings of source text marked as
containing dialogue. And in general, wherever possible, the proposal aims to
build out dialogue support in ways which cohere with existing Inform language
features. In particular, authors who know some basics of Inform (rooms,
things, people, the use of `now` and `if`, and actions) can make good use
of this to add state to what might otherwise be a simple branching narrative.
Authors confident in creating relations and tables can build sophisticated
knowledge models into their dialogue.

Inform is often, though generally quietly, used in the commercial games
industry as a tool for rapid prototyping. This proposal aims to assist that
use case, making dialogue prototyping much easier, and also opening the
possibility that such a prototype dialogue system could go on to be used
in the resulting production game. Since Inform is now able to compile C code
and link into large game programs written in, say, C# or C++, previous
technical obstacles to this have now been removed.

## Components affected

- [x] Major change to the natural-language syntax.
- [x] Minor change to inbuild in order to manage dependency on `DialogueKit`.
- [x] Major change to inform7: new problem messages, new Preform syntax and so on.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Major change to runtime kits, with new `DialogueKit` added. Changes to other kits minor.
- [x] Minor changes to the Standard Rules and Basic Inform.
- [x] Major change to documentation, with new chapters and examples needed.
- [ ] No change to the GUI apps.

## Impact on existing projects

Low. If an Inform source text does not contain any dialogue sections, then
most of the features below will deactivate (and `DialogueKit` will not be
linked into the compiled Inter code).

Because this proposal builds new kinds and adds a few new activies and rules,
the names of those novelties could cause namespace collisions with existing
projects. For example, an existing project using the term "dialogue beat"
might no longer compile.

However, the changes made by this proposal are almost all gated by a named
language feature, `dialogue`. This will be on by default but will be possible
to turn off in a project's JSON metadata, so that existing source texts
which do not want the new features but do want to keep compiling can easily
be configured to do so.

Runtime support will be delivered by a new Inter kit, `DialogueKit`, in the
standard Inform distribution. This is included in a build only if the "dialogue"
language feature is active and only if the project actually contains dialogue
sections.

It follows that inbuild will need to look for dialogue sections when parsing
source text to work out whether a project depends on `DialogueKit` or not.
`DialogueKit` in turn requires `WorldModelKit`.

## Overview

Most Inform source text imitates prose writing: it's intended to look like
a novel or a short story, and is divided up by headings into chapters,
sections and so on. Under this proposal, sections will now be allowed to
consist of "playtext" instead of prose: that is, will look like the printed
script of a play. For example, this is a modest dialogue section:

	Section 2 - On the Battlements (dialogue)

	(About the paranormal.)

	Marcellus: "What, has this thing appear'd again to-night?"

	Bernardo: "I have seen naught but [list of things in the Battlements]."
	
	Marcellus: "Horatio says 'tis but our fantasy."

This dialogue is made up of "dialogue beats", or "beats" for short, which
are brief conversational exchanges on a given subject. Here, there's just
one beat. The individual speeches are called "dialogue lines", or "lines"
for short: in this example there are three.

The Inform compiler will parse playtext like the above, and convert it into
data structures which it will compile into the story file. At runtime, a
subsystem called the "director" will decide when somebody speaks, who that
person is, and what they say. (`DialogueKit` is essentially the implementation
of the director.)

The director is sometimes "active", sometimes "passive". By default it is
passive, which means that it only begins dialogue when the author specifies
this. In active mode, it can also fill conversational lulls by trying to
find relevant things to talk about, and people to talk about them. To do
this, the director is constantly managing a list of topics ("conversational
subjects") which it might be interesting to talk about.

In this proposal, we will call such subjects "live". The idea is that if
somebody has just mentioned visiting Barcelona, then Barcelona might become
a live subject. At other times, it would just be an odd thing to bring up,
so it would be a subject but not a live one.

## New kinds

Several new kinds are built in. Most users will never need to refer to
these and will be blissfully unaware of the implementation under the surface,
but they do have names, and expert users will be able to play with them.

### Dialogue beat, dialogue line, dialogue choice

`dialogue beat` is an enumerated kind. Each value represents a single beat
(see below). Most beats are nameless in the source text; internally, those
will be given intentionally obscure names such as `nameless beat 13`, which
can be printed at runtime for debugging purposes but not referred to in the
source text as constants.

`dialogue line` and `dialogue choice` are similarly enumerated kinds.

The containment relation can be tested for dialogue lines, so that `L is in B`
is a meaningful condition for any dialogue line `L` and dialogue beat `B`; however,
it cannot be asserted with `now`. Beats and the lines in them are all set at
compile time.

The either/or property `recurring/non-recurring` exists for dialogue beats
just as it does for scenes, and has a similar meaning: see below.

The either/or property `performed/unperformed` exists for all of dialogue
beats, dialogue lines and dialogue choices. `performed` means that the line
has been spoken in the course of the current game; so at the start of play,
all beats, lines and choices are `unperformed`.

The either/or property `elaborated/unelaborated` exists for dialogue lines
only: see below. The value property `content` also exists for dialogue lines,
and has the kind `text`.

Values of `dialogue beat` and `dialogue line` cannot be created with assertions,
but only by using the dialogue section syntax described below. This is an
error:

	The Marcellus gets anxious beat is a dialogue beat.

Note that `recurring` and `performed` can be modified in play using `now`:
so, for example, `now the Marcellus gets anxious beat is performed` is legal.
This is intended only for expert users wanting to tweak conversational
behaviour in unusual ways.

### Performance styles

`performance style` is a simple enumerated kind of value. By default, it
has just one value: `spoken normally`. But the user can create additional
styles with assertions like so:

	Spoken angrily and spoken softly are performance styles.

Performance styles exist as a hook for ambitious systems which, for example,
change character art or pose for speakers who become angry, or which apply
emotional parameters to autogenerated voice performance, in the event the game
is using a text-to-speech.

Inform will not allow the following words to be used after "spoken" in the
names of instances of this kind:

	"and", "or", "if", "unless", "before", "after", "now", "to", "this", "without"

So for example it will reject:

	Spoken before thinking is a performance style.

This is to prevent ambiguity: see below.

## Dialogue sections

Inform already has annotations, in brackets, attached to heading names. The
new annotation "dialogue" marks that a section contains dialogue. For example:

	Section 2 - On the Battlements (dialogue)

The US spelling "dialog" will also be accepted:

	Section 2 - On the Battlements (dialog)

This is a point of difference with the names of the kinds `dialogue beat`
and `dialogue line`, where British English spellings must be used, but reflects
that hardly any users will ever see or type the names of those kinds, whereas
all users of dialogue will need to type the above annotation quite often.

Note that the annotation will throw a problem if used on other levels of
heading, such as chapters. Since sections are the lowest-level headings in
Inform source text, it follows that a dialogue section cannot be interrupted
by a subheading.

Dialogue sections contain a very different syntax from other parts of the
source text. Rules, tables, equations and assertions are all forbidden here,
and in their place are beats, lines and choices.

Formally: other than commentary, a dialogue section must consist only of a
sequence of 0 or more "dialogue beats", or "beats" for short. Each beat
occupies one or more paragraphs, of which the first must be in round brackets
and is called the "cue", and all others must be "dialogue lines" or "choices".
For example, this is a complete dialogue section:

	Section 2 - On the Battlements (dialogue)

	(About the paranormal.)

	Marcellus: "What, has this thing appear'd again to-night?"

	Bernardo: "I have seen naught but [list of things in the Battlements]."
	
	Marcellus: "Horatio says 'tis but our fantasy."

This dialogue section contains one beat, which contains one cue and three lines.

### Cues

The purpose of the cue is to specify when the beat of dialogue should be played,
though that may depend also on information elsewhere in the source text.

The cue sentence(s) must be placed in round brackets, and should end with its
full stop inside the brackets. This is an error:

	(About the paranormal).

The cue must contain at least one sentence, but must not contain a paragraph
break. Each sentence must be one of the types outlined below in (1) to (8),
with each type used at most once. An extensive example:

	(About the paranormal. After the haunting beat. If the Ghost is in Elsinore.
	Recurring. This is the Marcellus gets anxious beat.)

As syntactic sugar, these sentences can also be combined with the use of commas
or `and` to divide them. For example:

	(After the haunting beat and if the Ghost is in Elsinore, about the
	paranormal. Recurring. This is the Marcellus gets anxious beat.)

#### (1) Naming sentences in cues

Each beat provides a value for the kind `dialogue beat` (see above), but most
beats are nameless. By providing a sentence like this, the value is given
a name. For example:

	(This is the Marcellus gets anxious beat.)

The name here is `Marcellus gets anxious beat`. The name has to end with
`beat`, in English at least, for namespace reasons, and has to be unique:
two different beats cannot have the same name.

The name `starting beat` is reserved. If a beat is given this name, then
it is performed at the start of play. This is especially useful for non-command
parser stories, but can be used in any story. In command-parser IF, the starting
beat is performed before the player's first command.

There is no analogous `ending beat`: stories can have multiple endings. But see
`ending the story...` in the description of lines below.

#### (2) About sentences in cues

`About` sentences tell the director what a beat of dialogue is talking about.
This does two things: first, it tells the director which beat to select if
a given topic comes up in conversation; and second, it tells the director
when new topics have been brought up by one beat which might then flow into
future beats.

For example, in a command parser game, **ASK HORATIO ABOUT THE GHOST** might
cause the director to be asked for dialogue to appear next. The director might
then perform this:

	(About the paranormal.)

	Player: "Do you really think ghosts exist?"

	Horatio: "They're not dreamt of in my philosophy, unlike [list of
	scientific concepts]."

At its simplest, an about sentence consists of the word `about`, followed by
a list of one or more conversation subjects:

	(About the paranormal.)
	(About Marcellus and the paranormal.)

`About X, Y and Z` means that the beat involves all of the subjects
`X`, `Y` and `Z`. This means that it can be performed when any of those subjects
are live, and that once it has been performed, the others all become live.
This enables conversation to flow, by allowing those other subjects to
be talked about in subsequent beats. For example, the first beat below
is `about Velma and the robbery`. As we begin, Velma is live, but the
robbery is not. Performance of the first beat changes that:

	> ASK ABOUT VELMA
	Fred: "Velma totally hasn't been the same since that bullion robbery."
	> ASK ABOUT BULLION
	Fred: "Velma used to work the Lufthansa cargo desk at Stuttgart, and..."

Using `about either ... or ...`, though, we can instead say that the beat
is a suitable response to the subjects named, but does not make any of them
live. For example, this beat might be `about either Daphne, Scooby or Scraggy`:

	> ASK ABOUT DAPHNE
	Fred: "Just another loser I used to drive around with."

This dialogue can appear because Daphne is a live subject, but does not
make either Scooby or Scraggy live.

If a general description is used rather than a single explicit name, then
this is taken as an `either... or...` situation unless the keyword `every`
is used. For example, this beat might be `about every dwarf`:

	> ASK ABOUT SLEEPY
	Snow White: "I have eight? no, seven indentured miners right now. No
	point remembering their names, they die so quickly. I just go by
	appearances. The present lot are Doc, Grumpy, Happy, Sleepy, Bashful,
	Sneezy, and Dopey. Might let Sneezy go, actually, don't want a sudden
	cave-in to knock out all seven at once. You can only buy fresh dwarfs
	on Wednesdays, you see, so one bad sneeze on a Thursday and..."

And this might be `about any dwarf` or just `about a dwarf`:

	> ASK ABOUT SLEEPY
	Snow White: "He's a varmint like all the rest of them."

For a beat which responds to a set of different subjects but then _also_
mentions new subjects, the `mentioning` clause can be used (see below).
Thus `about any dwarf and mentioning diamonds and pageants` might do for:

	> ASK ABOUT SLEEPY
	Snow White: "He's a varmint like all the rest, but the diamonds are in
	these crazy low tunnels, and I can't afford to muss my hair. I'm still
	super-active on the pageanting circuit, you know."

In this context, a "conversation subject" is nothing more than a description
of an object. In practice that will want to be a concept, a thing or just
possibly a room, but probably not a region or scene. Note that a subject is
a description, which can be vaguer than a name. For example, if the source
text contained, say,

	A concept can be frightening or safe.
	The paranormal is a frightening concept.
	Garden design is a safe concept.

...then:

	(About any frightening concept.)

would have the beat considered if the paranormal were in play, but not for
garden design. Similarly:

	(About any woman in the Dining Room.)

One might, for example, create a `discrediting` relation from concepts to
people, and then have:

	(About any concept which discredits Gordon.)
	
	Douglas (to Carolyn): And you say the marriage wasn't a success?

For clarity, a conversation subject is not allowed to contain a comma, or
the words `and` or `or`. If the user wants a description so complex that it
needs these, she should set up some adjective and then describe with that.

Still to be determined: can callings be used in descriptions of subjects
used in "about" sentences? If so, where would the variables they create be
in scope?

#### (3) Condition sentences in cues

These restrict the availability of dialogue beats, making them only available
when the given condition is met (or not met):

	(If Denmark is rotten.)
	(Unless Hamlet has the skull.)

Arbitrary conditions can be slow to test, so the condition will be tested only
when the beat is being seriously considered for use. (That is, if an `about`
sentence restricts it, then the `if` condition would be checked only when the
conversational subjects for the beat actually arose.)

#### (4) Chaining sentences in cues

These say that the new beat can be performed only when another beat has already
been performed at some point in the past; or, has not. For example:

	(After the Marcellus gets anxious beat.)

This is equivalent, in fact, to writing the condition sentence:

	(If the Marcellus gets anxious beat is performed.)

...but is simpler to read and understand. Similarly:

	(Before the Marcellus gets anxious beat.)

The special one-word sentence `Later` can only be used on the second or
subsequent beats in a dialogue section, and means the same as `after` applied
to the previous beat in that section. Thus

	(About fish.)
	...
	
	(Later, about chips.)
	...

means the same thing as

	(About fish. This is the fishmonger beat.)
	...
	
	(After the fishmonger beat, about chips.)
	...

`Immediately after` is equivalent to `after`, except that it allows the
beat to be performed only the very next turn (in parser IF terms) after
the named beat was performed. The one-word `Next` is the corresponding
version of `Later`, applying `immediately after` to the previous beat.
For example:

	(About Pygmalion and mentioning Cyprus.)

	Galatea: "I haven't seen him around since I left
	Cyprus."

	(Next, about Cyprus.)

	Player: "Do you think he's still there?"

	Galatea: "Maybe! I don't yet have a good understanding
	of object permanence!"

	(About Cyprus.)

	Player: "Did you like Cyprus?"

	Galatea: "Yes, it was great, if you don't mind the weather."

The effect here is that we'd see

	> ASK GALATEA ABOUT PYGMALION
	Galatea: I haven't seen him around since I left Cyprus.

	> ASK HER ABOUT CYPRUS
	Player: Do you think he's still there?

	Galatea: Maybe! I don't yet have a good understanding of
	object permanence!

`Immediately before` is, of course, impossible, and is rejected with a
problem.

#### (5) Supplying properties for beats

The name of a property which a `dialogue beat` can have (an either-or
property, or conceivably a value of an enumerated property) can be a
cue sentence all by itself. In particular, this means that:

- The cue `(Recurring.)` gives the beat the `recurring` property. This means
that it can be performed only once, or rather, that it will not be considered
for performance if it has the `performed` property.

- The cue `(Spontaneous.)` gives the beat the `spontaneous` property. A spontaneous
beat can be chosen by the director, when the director is active and looking
for conversations to start, even if it has no `about` subjects. (See below.)

But note that new properties for dialogue beats can also be created with
regular Inform sentences like:

	A dialogue beat can be testing only or meant for production.

And this enables beats to be tagged in arbitrary ways:

	(Testing only.)
	
	Horatio: "Sorry to break the fourth wall, but will this program ever work?"		

#### (6) Requiring sentences in cues

Dialogue cannot be performed if the speakers are out of the player's earshot:
for example, the player is in a Zeppelin flying over Antwerp in 1921 while the
two speakers are in 16th-century Denmark.

The director therefore needs to test whether person A can hear dialogue by
person B. This is very like visibility, already determined in the existing
role model, and in indeed most of the time audibility and visibility will be
the same thing for our purposes. But we will probably want to build out
the customisability of this concept as part of this work. For now, that's
not specified, but we will somehow implement the verb `A can hear B`.

Given that, we could force dialogue to be performed by people actually present
with something like:

	(about the duel, if the player can hear Bernardo)

But that's clumsy. Instead, the director makes a _guess_ at who needs to be
present. Consider this beat:

	(about the duel)

	Bernardo: "Yo!"
	
	Marcellus: "Eh?"

	Horatio (now Fortinbras is in Elsinore): "Aha!"
	
	Fortinbras: "Yikes!"

The guess here is that Bernardo, Marcellus and Horatio are required but that
Fortinbras is not, because he only speaks after the first time that something
potentially happens. (And that's right in this case.)

There will be times when this guess is wrong. In any case, when speakers are
named only vaguely (see below), even these guesses will be difficult. As a
fallback, then, authors can write

	(about the duel, requiring Horatio and Bernardo)

And this sets the list of speakers required to be audible in order for the beat
to begin. (When the beat is being performed, any speaker not in fact audible has
their lines unperformed, but the others continue to speak. If that causes weird
non-sequiturs, well, the `requiring` list should have been better made.)

### Lines

After the cue paragraph, a beat consists of lines. Each line occupies a
single paragraph, and takes one of these forms:

	SPEAKER: "TEXT."

	SPEAKER (PERFORMANCE DETAILS): "TEXT."

#### The speaker

`SPEAKER` must be one of two possibilities.

`Narration` means that the line is not dialogue, but bridging narration in
the story.

	Narration: "Jojo walks over to the jeep."

Note that the text still appears in quotation marks in this case. This is
standard Inform text, potentially with substitutions in square brackets, and
follows all of the usual language conventions for text.

If `SPEAKER` is anything other than the word `Narration`, then it is a
description of who speaks: this can be any Inform description of a thing, so
it need not be a literal name of a person, though it usually will be.

	Marcellus:
	A woman:
	Somebody who is not Marcellus: 
	Th0e tallest person in the Dining Room:

If multiple objects (which are not out of play) would match the description,
the object maximimising the following score is chosen:

	+8 for being audible to the player
	+4 for being the interlocutor (see below) of the previous speaker in the beat
	+2 for being of the kind "person"
	+1 for being different to the previous speaker in the beat

If multiple objects still have equal scores, a random choice is made.

If no person (who is not out of play) matches, the line of dialogue is skipped,
but no run-time problem is thrown.

#### Performance details

If given, the performance details are one or more clauses, divided by commas,
and placed in round brackets after the speaker name. Thus:

	SPEAKER (CLAUSE1, CLAUSE2, ..., CLAUSEn): "TEXT."

Clauses can be any of the following: note that, in English at any rate, the
first word of the clause determines which of the following is parsed.

(1) `if` or `unless` plus an Inform condition. The line is skipped (simply
omitted: no run-time problem is thrown) if the condition fails. In testing
the condition, the local variable "speaker" is set to the speaker. Thus:

	A random guard (if the speaker can see Hamlet): "Yo, Prince!"

(2) `before` or `after` plus an Inform action constant. If it's `before ACTION`,
then `ACTION` is tried after the line is performed. If it's `after ACTION`, the
`ACTION` is tried before the line is performed - and in that case, if the action
fails, the line is silently not performed (and no run-time problem is thrown).
Unless otherwise specified, the actor of the action is the speaker, so this
is an exception to the general rule of Inform that the default actor is the
player. For example:

	Gravedigger (after taking the shovel): "Here goes another shallow one."

would be performed as:

	The gravedigger takes the shovel.
	
	Gravedigger: "Here goes another shallow one."

As with `try`, the keyword `silently` can also be used. Given the following:

	Gravedigger (after silently taking the shovel): 
		"'Here goes another shallow one,' says the thin man lugubiously, picking up the shovel."

possible outcomes might be:

	"Here goes another shallow one," says the thin man lugubiously, picking up the shovel.

if he succeeds in taking it, or text printed by some rule failing the action:

	The gravedigger reaches for the shovel, but Horatio grabs it first.

if he does not.

The actor need not be the speaker, but the action has to succeed whoever the
actor may be:

	Gravedigger (to Hamlet, after Hamlet silently taking the shovel):
		"Hey, I saw you picking up that shovel! That's a union job, sweet Prince!"

(3) `now` plus a condition, which is asserted before the line is spoken.
For example, suppose the source text elsewhere says `A person can be scared.`
Then:

	Marcellus (now Marcellus is scared): "Oh my."

(4) `to` plus a named object, who is considered the "interlocutor", that is,
the person addressed. For example:

	Marcellus (to Horatio): "The Prince looks moody tonight, i'faith."

The line will not be performed unless Marcellus can hear Horatio.

(5) `without speaking` makes the line essentially narration. Thus:

	Marcellus (without speaking, to Bernardo): "Marcellus points a horrified finger, and nudges Bernardo."

The advantage of this over the simpler:

	Narration: "Marcellus points a horrified finger, and nudges Bernardo."

is that it sets the speaker and interlocutor variables: this may be useful
when state is being built on top of conversations by augmenting the director's
activity rules.

(6) `this is the ... line` provides a name for the value used at run-time to
represent this line of dialogue: they are ordinarily nameless.

(7) `mentioning ...` says that the line makes a conversation subject live
when it is performed. For example:

	Marcellus (mentioning the paranormal): "I be mighty afear'd of the paranormal."

(8) The name of a performance style, but with the word `spoken` removed. Thus,
if the user has created the style `spoken with asperity`, then:

	Marcellus (with asperity): "I've had it with these goddam ghosts."

This will set the activity variable `style` to `spoken with asperity` when the
line is performed (see below), rather than its default value, `spoken normally`.

(9) `ending the story` or `ending the story in ...` work like the existing
Inform phrases `end the story` and `end the story in ...`. The story ends
after the line is performed. Note that if the is conditional and is not
performed, the story does not end.

#### The line performed

The performed line is an Inform text, in double-quotes. It can come in two
forms, "unelaborated" and "elaborated". As examples of these:

	Marcellus: "What, has this thing appear'd again to-night?"

	Marcellus: "Marcellus gives a shiver. 'What,' he exclaims, 'has this thing appear'd again to-night?'"

Inform determines at compile-time whether the line is plain or fancy by looking
for at least one pair of single quotation marks at word boundaries. (Note that
the apostrophe in "appear'd" is not at a word boundary.) At runtime, the
either/or property "elaborated" for the dialogue line holds the value of this.

There is one exception to this: if the speaker is given as `Narration`, then
the line is always elaborated.

Elaboration affects how the line is performed, that is, what text is shown
to the player when the line has been spoken. This is done with the new
`performing dialogue` activity, which has three activity variables, `speaker`,
`interlocutor`, `style` and `line` (of kinds `thing`, `thing`, `performance
style` and `text` respectively). The default is:

	For performing dialogue with an elaborated dialogue line:
		say the content of the line.

	For performing dialogue with an unelaborated dialogue line:
		if the speaker is the player:
			say "'[content of the line]'";
		otherwise:
			say "[The speaker]: '[content of the line]'".

	After performing dialogue:
		now the line is performed.

Unless the user has meddled with this activity in some way, then, the line
examples above would be performed as:

	Marcellus: "What, has this thing appear'd again to-night?"

	Marcellus gives a shiver. "What," he exclaims, "has this thing appear'd again to-night?"

This can contain text substitutions in square brackets, in the usual way.
When such text is printed, the activity variable `speaker` will exist for it,
and will hold the identity of the speaker performing the line. (If the line
is narration, it will still exist, but will be equal to `nothing`.) This will
also be the value of `self` internally, i.e., references to properties which
do not specify an owner will be taken as properties of the speaker. Thus:

	A person in the Lounge:
		"[The speaker] self-importantly [declare]: 'The greatest TV show of all
		time is [if female]Gilmore Girls[if not]24[end if].'"

might be performed as:

	Henry self-importantly declared: "The greatest TV show of all time is 24."

Note that the default rules for performing dialogue ignore the performance
style, i.e., the current value of `style`. Performance styles exist to allow
the user to build more state on top of the dialogue engine: for example, the
user could give characters numerical scores for their current levels of anger,
and increase these when a character says something "angrily". Such changes
should be made in `After performing dialogue` rules.

We haven't reached agreement on whether the concept of elaboration should be
used by default. These views have both been argued:

(a) That authors would prefer or find easier to understand a system where
everything is elaborated, or else everything is unelaborated, so that the
decision is made for all of the dialogue in a project at once - use one format
throughout, or use the other throughout. In production games, especially
for interesting web or app presentations, consistency here is important.

(b) That the elaborated/unelaborated system gets things right automatically,
and allows users to write thousands of simple lines with just a handful
of elaborated ones mixed in; it also allows for the possibility that an
extension might contain dialogue which the author has not written herself.

A compromise may be for the compiler to silently track elaborated/unelaborated,
but for the default rules in the `performing dialogue with` activity to make
no use of it; Examples in the documentation can then show how to get the effect.

### Choices

During a beat, dialogue ordinarily flows without stopping (see "continuity
of performance" below), but will pause for the player to exercise choices
at "choice" sentences, if there are any.

These begin with a double hyphen and often a run of them is presented together,
making a set of options. For example:

	-- (perform bluffing it out beat) "Pretend everything is fine"
	
	-- (perform facing them down beat) "React with horror"

This goes to the player and asks for a choice between the options available.
The syntax here, then, is one of:

	-- (DETAILS) "TEXT"

	-- (DETAILS) ACTION-PATTERN

	-- (DETAILS) otherwise

	-- (DETAILS) again

	-- (DETAILS) stop

	-- (DETAILS)

where in each case the `(DETAILS)` are optional.

Once again, the details can affect both whether a choice is offered at all and
what will happen if it is both offered and then chosen. Details can be given
in any order and are divided with commas, once again:

(1) `if ...` and `unless ...` apply conditions on whether the choice is offered.
When a run of choices is being looked at, the variable `choice count` keeps
a running tally of those found acceptable to be offered so far (compare the
function `CHOICE_COUNT()` in Ink). Thus `if the choice count is less than 3`
would check whether less than three choices had so far been found possible.

`if` and `unless` are illegal for an `-- otherwise` choice, which must always
be possible.

(2) `perform ...` followed by a beat name means two things. If the beat has been
performed and is not recurring, then the option is not offered. Otherwise,
the option is offered, and if chosen then the current beat ends and the named
new beat immediately begins. In this process, beats are not nested.

(3) `perform ... and continue` is similar, but continues the current beat after
the named new beat finishes. In this process, beats are nested.

(4) `this is the ... choice`  provides a name for the value used at run-time to
represent this choice point in the dialogue: they are ordinarily nameless.

Now for the "selector", that is, what the player does to indicate that this
choice should be taken. Inform requires that the run of choices offered at
any point must either consist entirely of choices selected by texts:

	-- "Launch into a complicated excuse."
	-- "Run for it!"

or entirely of choices selected by actions, which can be but need not be
actions to do with talking:

	-- examining a door
	-- asking Clark about "ticket [even number]"

In either case, an `-- otherwise` choice can also be offered, but if so it
must appear last and must not be the only choice in the run.

`-- again` always forms a run on its own, always being the only choice
available - it is in fact an unconditional branch, and returns to the
previously offered choice position, re-offering the choices. (That's to
say, the narrator works out which choices are available - which may have
changed in the intervening time - and then offers those which are.)

What the director does to find out the player's next intention depends on
whether we have a run of text choices, or a run of action choices.

#### Text choices

Here the director essentially needs to print out the possibilities and ask
the player to choose one, though this is very customisable.

Since choices can be only sometimes available (e.g. having `if` conditions),
the director first determines which choices in the run are actually on offer.
If the result is an empty list - they are ruled out for one reason or another,
and there isn't an `-- otherwise` option - then the director offers no
choice at all. The beat continues from after the run of choices.

Otherwise, the list of possibilities is passed as a value of kind `list of
dialogue choices` to the activity `offering a dialogue choice`. The default
implementation in a command-parser game expecting keyboard input would
probably print something like:

	(1) Pretend everything is fine...
	
	(2) React with horror...
	
	(Press 1 or 2)

and wait for keyboard input. In a Vorple game, something more point-and-click
is likely, and the choices may be erased from the screen when made, say.

#### Action choices

This is only suitable for interactive fiction which has some sort of interface
allowing the player to choose actions: the traditional example, of course,
being keyboard input run through a command parser.

The director pauses its work and allows the game to go through its usual turn
cycle: for parser IF, that means printing a command prompt, accepting keyboard
input, action processing, every turn rules, and so on. Once that is done, the
action just processed is used to determine the choice made, and the beat
continues.

Note that the action is indeed processed, so if the player's command called
for things to happen, then they will indeed happen. For example:

	-- examining a door
	
		Horatio: "The river's out of the window, Ophelia. Not the doors."

	-- otherwise
	
		Narration: "Horatio looks annoyed not to make a snarky remark."

might result in either

	> EXAMINE BLUE DOOR
	The blue door bears the arms of the old King, hastily crossed out.
	
	Horatio: "The river's out of the window, Ophelia. Not the doors."

or

	> OPEN BLUE DOOR
	You open the blue door.
	
	Horatio looks annoyed not to make a snarky remark.

But in both cases the player made an action happen. To avoid this, use
`instead of`. Thus:

	-- instead of examining a door
	
		Horatio: "The river's out of the window, Ophelia. Not the doors."

	-- instead of doing something

		Narration: "Horatio intervenes, annoyed not to make a snarky remark."

Now the result would be either

	> EXAMINE BLUE DOOR
	Horatio: "The river's out of the window, Ophelia. Not the doors."

or

	> OPEN BLUE DOOR
	Horatio intervenes, annoyed not to make a snarky remark.

And in this case no action occurs. This is likely to be especially useful
if the actions referred to are conversational ones, where we would otherwise
get odd clashes between the parser's usual handling of ASK X ABOUT Y and
our new one.

With a run of action choices, an `-- otherwise` is used if none of the
other choices matched.

For example:

	(this is the proposal beat)

	Clark: "Will you marry me, Kate?"

	-- instead of saying yes
	
		Player: "Yes, Clark! I adore you!"

		Clark: "I'm so glad!"

	-- instead of saying no
	
		Player: "No. Not a chance. You're always pushing wheelbarrows
		about and I don't think you really love me."

		Clark: "Bummer."

	-- instead of asking Clark about "hat"

		Player: "I'll consider it if you stop wearing that hideous hat."

		Clark (after taking off the hideous hat): "Done!
		Darling, we'll be so happy!"
		
	-- otherwise

		Clark: "[one of]Don't change the subject...[or]I'm starting to think
		you're not interested.[or]I'm really regretting proposing during
		this baseball game.[stopping]"
		
		-- again

Note the position of the `-- again`, indented underneath the `-- otherwise`.
This little conversational predicament continues until the player puts
Clark out of his misery, one way or another. Had the `-- again` been
unindented, the result would be that the whole thing goes on forever.

`-- stop` halts the whole beat when reached, breaking out right to the
top level. So the above would be equivalent to:

	(this is the proposal beat)

	Clark: "Will you marry me, Kate?"

	-- instead of saying yes
	
		Player: "Yes, Clark! I adore you!"

		Clark: "I'm so glad!"
		
		-- stop

	-- instead of saying no
	
		Player: "No. Not a chance. You're always pushing wheelbarrows
		about and I don't think you really love me."

		Clark: "Bummer."
		
		-- stop

	-- instead of asking Clark about "hat"

		Player: "I'll consider it if you stop wearing that hideous hat."

		Clark (after taking off the hideous hat): "Done!
		Darling, we'll be so happy!"
		
		-- stop
		
	-- otherwise

		Clark: "[one of]Don't change the subject...[or]I'm starting to think
		you're not interested.[or]I'm really regretting proposing during
		this baseball game.[stopping]"
		
	-- again

Here, that final `-- again` makes it look as if the loop goes on forever,
but the three `-- stop` choices mean that in fact it works as before.

Nothing can follow an `-- again` or `-- stop` at the same indentation level,
and nothing can appear beneath it: either results in problem messages.

### Nested choices

As has already appeared in several examples, indentation can be used to
avoid having to construct elaborate conversation trees out of named beats.
For example:

	Bernardo (now Bernardo is scared): "Yikes! Ghost!"

	-- "Pretend everything is fine"
	
		Player: "It's just Scotch mist, blown east from old Aberdeen."
		
		Bernardo: "Nay, sire, it's magical!"
		
		-- (if the shortbread is carried) "Assuage his night terrors"
		
			Player (before Bernardo eating the shortbread): "Here, take this old Highlands cure."

			Bernardo (now Bernardo is not scared): "Hoots mon, that's better."

		-- "Reprimand Bernardo"

			Player: "I don't care if it's haunted, guard the Battlements."
		
		Polonius (now Polonius is in the Battlements): "I never come up here."

	-- "React with horror"
	
		Narration (before going north): "You panic and run for the woods."

This is actually five different beats, since the dependent parts under each
choice are all beats in their own right.

The obvious rules apply: speeches and choices can occur only at the current
indentation level or at a level 1 deeper than the previous speech or choice.

Polonius's line of dialogue is interesting as an example here: this is where
the beat continues after going through either one of the two previous choices.
(This point is what would, in Ink, be called a "gather".) That happens
with no need for special syntax because the indentation makes it clear, since
it's a dialogue line occurring at an indentation showing that it cannot be
part of the "Reprimand Bernardo" thread.

Note that if two runs of choices are to follow each other with no intervening
dialogue or narration, there would be no way to convey that with indentation
alone. The special choice line `--` alone provides the necessary division. Thus:

	Narration: "What is your favourite colour?"

	-- "Blue"
	
		Narration: "You are a Democrat or a Conservative voter."

	-- "Red"
	
		Narration: "You are a Republican or a Labour voter."

	--
	
	-- "Accept this verdict"

		...
	
	-- "Dispute this verdict"
	
		...

Here, the bare `--` line divides this into two two-choice questions in
a row, with nothing coming in between. This is rarely likely to be useful,
but should be possible.

### Branches and subroutines, of a sort

A bare `--` line which has bracketed `perform` instructions is, however,
very useful. For example:

	Marcellus: "What, has this thing appear'd again to-night?"

	-- (perform the spectral reappearance beat)

This effectively ends the current beat and starts another one; no choice
is presented to the player, of course. In effect, this is GOTO for beats.
And this is GOSUB:

	Marcellus: "What, has this thing appear'd again to-night?"

	-- (perform the spectral reappearance beat and continue)

	Horatio: "Thank the Lord that's over."

### Dependent dialogue

Lastly, dialogue can conditionally lead to other dialogue without the need
for explicit player choices. Again, this is done with indentation:

	Bernardo (now Bernardo is scared): "Yikes! Ghost!"
	
	Hamlet (if Hamlet is not scared): "It's just Scotch mist, blown east from old Aberdeen."
		
		Bernardo: "Nay, sire, it's magical!"

	Hamlet (if Hamlet is scared, without speaking, before going north):
		"Hamlet panics and flees for the woods."
	
		Ghost: "Where is my son anyway?"

There are no player choices here. Instead, the dialogue can play out in
different ways, depending on which conditions are true. As it happens,
in this example the two conditions on Hamlet's lines are mutually exclusive,
so there are two possibilities: either this...

	Bernardo: "Yikes! Ghost!"
	
	Hamlet: "It's just Scotch mist, blown east from old Aberdeen."
		
	Bernardo: "Nay, sire, it's magical!"

or this...

	Bernardo: "Yikes! Ghost!"
	
	Hamlet panics and flees for the woods.
		
	Ghost: "Where is my son anyway?"

## The role of the director

As noted above, the "director" is the subsystem at runtime which chooses when
dialogue is performed and does the necessary business to implement the
behaviour described above. The director will be implemented in Inter, and
provided as a kit called `DialogueKit`, rather than in Inform source text
provided by an extension.

The above sections describe how the director chooses speakers and performs
lines, but not when, and this is the question we now take up.

### Continuity of performance

When a beat is performed, its lines are normally all performed in sequence.
Individual lines may be omitted if they fail their conditions (see above),
but the usual effect is that all lines are performed. So the performance of
the beat:

	(About the paranormal. This is the Marcellus gets anxious beat.)

	Marcellus: "What, has this thing appear'd again to-night?"

	Bernardo: "I have seen naught but [list of things in the Battlements]."
	
	Narration (now Marcellus is in Dunsinane):
		"Marcellus panics and runs for the safety of the woods."

would be something like:

	Marcellus: "What, has this thing appear'd again to-night?"

	Bernardo: "I have seen naught but Hamlet, that funky shield and the Moon."
	
	Marcellus panics and runs for the safety of the woods.

In other words, action is continuous within a beat. So the choice made by
the director is not so much when to produce a single line of dialogue
as it is when to produce a beat.

Note that `DialogueKit` must correctly handle the performance of one beat
within the performance of another, returning where it left off. Though it
is unlikely that such nested beats will occur often or deeply, they need to
work in order to handle situations where one conversation involves somebody
doing something in the middle of an anecdote, and where that action triggers
off further dialogue about what has just happened.

### Using phrases to begin beats

Three new phrases are provided by the Standard Rules to ask the director to
perform a beat.

(1) Rules written by the author can explicitly call for it, using the phrase:

	To perform (B - a dialogue beat): ...

For example:

	After the player examining Marcellus in the Battlements:
		perform the Marcellus gets anxious beat.

Note that unless the beat is recurring, it will only be performed once, and
subsequent uses of `perform` on it will silently do nothing.

If the beat has `if` or `after` conditions attached, and these conditions are
not met, then `perform` will silently do nothing. But there is no need for any
of the `about ...` subjects to be relevant to the dialogue manager.

(2) More obliquely, the director can also be invited to perform dialogue related
to a given room, thing or concept, in which case it must choose a suitable beat
(or decide that nothing is suitable, and perform nothing).

	To decide whether or not dialogue/dialog about (O - an object) intervenes: ...

asks the director to find an unperformed (or recurring) beat which is `about`
the object O. If the director finds one, it performs this beat, and
returns `true`; if not, it does nothing and returns `false`. For example:

	Before examining (T - a thing):
		if dialogue about T intervenes, stop the action.

In particular, this phrase is used in the default implementation of the
action `asking somebody about`, in such a way that **ASK MARCELLUS ABOUT THE
PARANORMAL** would cause the above beat to be performed, since it is marked
as being "about the paranormal".

This phrase provides a useful implementation for handling **ASK X ABOUT Y**,
where X is a person and Y is a concept, as outlined nearer the top of this
proposal.

Note that if the project contains no dialogue sections, and therefore will
not use `DialogueKit` and will not have a director, the two phrases above will
still both compile (unless the `dialogue` language feature has been turned
off altogether), but will do nothing (return `false` in case (2)). The
example rule:

	Before examining (T - a thing):
		if dialogue about T intervenes, stop the action.

will therefore happily compile in a dialogue-free story, but do nothing.

### The director acting autonomously

The director has two modes, "active" and "passive". When "passive",
the director begins beats only in response to other beats already playing
(when choices are taken), or in response to the phrases just described.
This makes the director rather like a conversation-handler in a traditional
command parser IF game, and gives all control to the author.

In "active" mode, however, the director has additional autonomy to try to keep a
conversation flowing whenever there are people around to talk, and things of
interest for them to talk about. This is arguably a better simulation of real
life.

As in real life, though, incessant conversation is not always what you
want, so we provide controls:

	To make the dialogue/dialog director active: ...

	To make the dialogue/dialog director passive: ...

By default, at the start of play, the director is passive.

A new `dialogue direction rule` will be present towards the end of the
turn processing rulebook. One task of this rule will be to keep multi-turn
beats running (see above). Another, but only in active mode, will be to
look around for possible beats to begin, but this will only be done if
in the current turn no beat has been, or is currently being, performed. 
Even in active mode, the director will give up and hope for better times
if no good conversational gambit presents itself. Active mode does not
mean a guarantee that somebody will speak in every turn.

When the director is active and there is such a lull, the dialogue
direction rule will run the new dialogue selection activity to choose
a beat to perform. (This is an activity based on nothing which produces
a dialogue beat.) By default this activity will contain only a single
`for` rule, providing the default choice algorithm.

This in turn tries, as efficiently as it can, to find:

(i) a live subject, and

(ii) a dialogue beat which is either unproduced or else recurring and
which is about that beat (for more on exactly what that means, see
the discussion of `about` above), such that

(iii) the speakers required for that beat to be performed (see the
discussion of `requiring` above) are all audible.

A typical story may contain thousands of beats, so an efficient method
must be used to select from them, but note that many beats are choice
branches from other beats and are not themselves `about` anything,
so are not dialogue entry points of the sort looked at here.

Different selection strategies could be used, depending on the level
of sophistication of knowledge modelling: we could try to measure salience
with some scoring system, or try to bring forward beats which we want to
make happen for plot reasons (where some pirate leans forward and tells
a story in a tavern, say), or making a graph of related concepts and
route-finding through it to make the conversation lead to a goal. But
the default is likely to be simple so that users can get started without
creating elaborate plots or knowledge models.

#### How subjects become live

Subjects can become live in the following ways:

(a) By being in the `about` or `mentioning` lists of dialogue beats or lines
which are performed. Thus, if somebody says something about perfume, then the
`perfume` subject (if there is one) becomes live.

(b) By the following phrase being used:

	To make (T - a thing) a live conversational subject: ...

This adds T to the director's current list of active topics. For example,

	After pressing the button:
		now the pyramid is in the location;
		make the pyramid a live conversational subject;
		"A platinum pyramid appears out of nowhere!"

Or indeed:

	After printing the name of a concept (called the notion):
		make the notion a live conversational subject.

Or again:

	When Railway Encounter begins:
		make the steam train a live conversational subject.

It's tempting to do this with an either/or property for being "live",
but that would not enable an efficient list to be kept by the director.

#### How subjects cease to be live

Two ways. First, the author can do it by phrase:

	To make (T - a thing) a dead conversational subject: ...

Thus:

	When Railway Encounter ends:
		make the steam train a dead conversational subject.

Similarly:

	To clear conversational subjects: ...

makes everything dead, and wipes the slate clean. This may be useful at the
end of scenes.

Second, the director itself performs a `clear conversational subjects` whenever
it starts a spontaneous beat (see below).

There may in practice be other times when it does this: the general idea is
that the number of live subjects should be low at all times. They really
reflect only fleeting conversational states, not long-term knowledge.

#### Spontaneous beats

If the director is active, and has tried to run the dialogue selection activity
but this has come back with nothing, then it next looks for beats marked as
spontaneous. For example:

	(spontaneous, mentioning Corsica)
	
	Napoleon: "'You know,' the Emperor says abruptly, 'I was born in Corsica.'"

The only difference between spontaneous beats and others is that the director
can select them without needing any subject to be live. They are still subject
to other constraints: this one, for example, requires Napoleon to be audible.

Or for example:

	(This is the interrogation dialogue beat.)
	
	Interrogator: "We have a number of questions for you, [player]."
	
	Interrogator: "Shall we start with the matter of how you broke in?"
	
	-- (perform the break-in detail beat) Describe how you broke in

	-- (perform the secret testing beat) Distract him with questions
	
	(Spontaneous, after the interrogation dialogue beat, if the break-in
	detail beat is not performed.)
	
	Interrogator: "I notice you didn't explain your method of entry."

## Relationship to Scenes

Scenes are mentioned surprisingly little in this proposal - surprisingly,
that is, considering that scenes are on the face of it an obvious way to
group and organise dialogue.

In fact, we strongly agree with this, but the features above dovetail well
with existing scene support in Inform. For example, the following all seem
natural possibilities:

	When the Confrontation scene begins:
		perform the Cyrano brags about his nose beat;
		make the Gallic nose a live conversational topic.

	The Confrontation scene ends in dueling when the glove slap
	beat is performed.
	
	When the Confrontation scene ends in dueling:
		perform the Cyrano draws his sword beat.
	
	When the Confrontation scene ends in dueling:
		clear conversational subjects.

All the same, we might contemplate allowing a scene to be identified with
a beat. For example, if a beat opens with the cue `this is the ... scene`
instead of `this is the ... beat`, then _both_ a scene and a beat are
created. Beginning the scene automatically starts the beat performing;
the scene ends when the beat finishes performing. (If the scene ends for
some other reason during the performance, the director halts the beat.)

	(this is the duel scene)
	
	Osric: "A palpable hit!"

	...

## Exporting dialogue for voice performers or localisation

Commercial games authors frequently need to export all of the dialogue
in a game to a simple, flat file (often, this is a CSV file which Excel
can read as a spreadsheet).

Inform should provide a feature to do this. Each line would be given:

- a unique identifier or UUID, and not as clunky or long as a hex checksum
but something more like `D34`,
- a speaker (if identified, or if not, source text specifying the speaker),
- the text of the line (accepting that this may, of course, include square
bracketed substitutions),
- the performance style.

Our proposal is that this file would be output to the Materials folder
on every compilation (of a project which actually contains some dialogue).
On subsequent compilations, Inform would aim to use the same UUIDs for the
same lines of dialogue, whenever possible. This is very useful for studios
keeping track of voice performances. In order to do this, then, Inform will
read in the file from the Materials folder (if it exists) and attempt to
match that, as far as possible, with the dialogue it sees: it will then
reuse those UUIDs.
