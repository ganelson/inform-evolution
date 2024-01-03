# (IE-0009) Dialogue Sections

* Proposal: [IE-0009](0009-dialogue-sections.md)
* Discussion PR link: [#9](https://github.com/ganelson/inform-evolution/pull/9)
* Authors: Emily Short and Graham Nelson
* Status: Accepted
* Related proposals: [IE-0010](0010-concepts.md)
* Implementation: Implemented but unreleased

## Summary

An extensible system for dialogue, combining natural-language playtext in
"dialogue sections" of the source text with an active runtime component,
the "director", implemented with a new kit, `DialogueKit`.

This proposal was redrafted in early October 2022 in the light of experience.
Unless otherwise noted, all functionality here is implemented in draft on
the `master` branch of the repository, but not in any release branch as yet.

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
projects. For example, an existing project using the term `dialogue beat` might no
longer compile.

However, most of the changes below take effect only in Inform projects which
contain at least one dialogue section, or where the `dialogue` feature is
explicitly activated by project (or kit) metadata. Since this will not be
true for any existing projects, very little of IE-0009 will affect them.

Runtime support will be delivered by a new Inter kit, `DialogueKit`, in the
standard Inform distribution. This is included in a build only if the `dialogue`
feature is active and only if the project actually contains dialogue sections.
Note that `DialogueKit` requires `WorldModelKit`.

An exception to the above is that there is a new relation in the world model:
`audibility`, which can be tested with the verb `to be able to hear`. This
relation is added whether or not dialogue is present. For the moment, the
definition of `audibility` is the same as that for `visibility` except that
there is no need for light.

## Dialogue sections

Most Inform source text imitates prose writing: it's intended to look like
a novel or a short story, and is divided up by headings into chapters,
sections and so on. A section whose heading is annotated `(dialogue)` has
a special syntax, unlike all other section in an Inform story, which means
that it holds "playtext" instead of prose: that is, it will look like the
printed script of a play. For example, here is a modest dialogue Inform
story containing some setup in a regular section, and then a dialogue section:

	Section 1 - Elsinore
	
	Elsinore is a room. Marcellus and Bernardo are men in Elsinore.
	The ghost is a man.

	Section 2 - On the Battlements (dialogue)

	(This is the starting beat.)

	Marcellus: "What, has this thing appear'd again to-night?"

	Bernardo: "I have seen naught but [list of things in the Battlements]."
	
	-- "Ask Bernardo what he saw."
	
		Bernardo (mentioning the ghost): "A ghastly apparition, my lord."
		
		<-
		
	-- "Dismiss this nonsense."
	
		Narration (mentioning Fortinbras): "You sternly tell the guards
		to watch for Fortinbras, not their shadows."

The US spelling "dialog" would also be accepted as the annotation:

	Section 2 - On the Battlements (dialog)

This annotation throws a problem if used on other levels of heading, such as
chapters. Since sections are the lowest-level headings in Inform source text,
it follows that a dialogue section cannot be interrupted by a subheading.

Dialogue sections can be marked with the name of a scene, in this case `Haunting`,
to say that everything in the section can only be performed when that scene is
happening:

	Section 2 - On the Battlements (dialog during Haunting)

Dialogue sections are not allowed to contain any of the usual ingredients of
Inform source text (assertion sentences, rules, tables, equations). Instead
they must contain only the following ingredients, used over and over, each one
a paragraph of its own.

- "Dialogue beats", or "beats" for short, are conversational exchanges arising
from a given subject, and can be vary in length from one-liners to elaborate
discussions. Beats are introduced by cues in round brackets: there's just one
cue in the above example, `(This is the starting beat.)`, and so `Section 2`
above contains only one beat.

- "Dialogue lines", or "lines" for short, are mostly single speeches each
performed by an individual person, who is usually explicitly named. Marcellus
speaks one line above, Bernardo two, and there is also a line of `Narration`,
which is special in that nobody in-story is speaking it, but otherwise behaves
just like other lines. Lines broadly have the syntax `Speaker: "Speech."`

- "Dialogue choices", or "choices" for short, are different options which the
player might take at this point in the conversation. There are two here, and
they offer the two possible options at what amounts to a single decision for
the player to make. Choices are introduced with a double-dash `--`.

- "Flow markers", which are special notations to control the flow of the
conversation. Here there is just one, `<-`, which means "go back to the last
decision point and ask again".

## Indentation

Note the use of indentation. Beat cues cannot be indented: there is no sense
in which they belong to each other. But all lines, choices and flow markers
can be. They can go at most one tab deeper than the previous paragraph.

For example, because Bernardo's line "A ghastly apparition, my lord."
is indented from the choice above it, it will be performed only if that
choice is taken. And suppose we have this:

	Bernardo (to Marcellus): "What say you, brother?"
	
		Marcellus: "Horatio says 'tis naught but their fantasy."

The first line here will be performed only if the player can hear Bernardo,
and Bernardo can hear Marcellus. The second will be performed only if the
first line has been (and also if the player can hear Marcellus). This is
better than writing just:

	Bernardo: "What say you, brother?"
	
	Marcellus: "Horatio says 'tis naught but their fantasy."

because it ensures that if Bernardo is present but Marcellus is not, or vice
versa, then the we don't see just half of a failed conversation. In general,
then, if one line is a direct reply to another one, and there is any uncertainty
about who might or might not be present, indentation should be used:

	Speaker One: "A speech."
	
		Speaker Two: "A reply."
		
		Speaker One: "A rebuttal."
		
			Speaker Three: "An intervention."

			Speaker Two: "A cross response."

### Nested choices

Indentation can go quite deep, though eventually it becomes bad style and
flow markers like `-> perform the duel beat` can be used instead (see below).
It certainly allows scope for nested choices. For example:

	Bernardo (now Bernardo is scared): "Yikes! Ghost!"

	-- "Pretend everything is fine"
	
		Marcellus: "It's just Scotch mist, blown east from old Aberdeen."
		
		Bernardo: "Nay, sire, it's magical!"
		
		-- (if the shortbread is carried) "Assuage his night terrors"
		
			Marcellus (before Bernardo eating the shortbread): "Here, take this old Highlands cure."

			Bernardo (now Bernardo is not scared): "Hoots mon, that's better."

		-- "Reprimand Bernardo"

			Marcellus: "I don't care if it's haunted, guard the Battlements."
		
		Polonius (now Polonius is in the Battlements): "I never come up here."

Polonius's line of dialogue is interesting as an example here: this is where
the beat continues after going through either one of the two previous choices.
(This point is what would, in Ink, be called a "gather".) That happens
with no need for special syntax because the indentation makes it clear, since
it's a dialogue line occurring at an indentation showing that it cannot be
part of the "Reprimand Bernardo" thread.

## Active versus passive

The playtext in dialogue sections is compiled into the story file as a fairly
elaborate data structure, and a runtime component called the "director" makes
use of this data, deciding when and how to perform it. `DialogueKit`, a new
Inter kit linked only into stories which have dialogue, implements the
director, but it works through activities and is therefore customisable.

The director is sometimes "active", sometimes "passive". By default it is
passive, which means that it only begins dialogue when the author specifies
this. In active mode, it can also fill conversational lulls by trying to
find relevant things to talk about, and people to talk about them. To do
this, the director tracks a list of "live conversational subjects" which
it might be interesting to talk about. The idea is that if somebody has
just mentioned visiting Barcelona, then Barcelona might become a live
subject.

The mode can be switched with these phrases:

- `make the dialogue/dialog director active`
- `make the dialogue/dialog director passive/inactive`

When the director is passive, beats are performed only as follows:

- if there is a beat called the `starting beat`, it is performed at the
start of play, after the initial room description but before the player
has been asked for a command;
- in response to the phrase `perform B`, where `B` is a named beat;
- in response to the phrase `if dialogue about X intervenes`, where `X` is
a potential conversational subject;
- when a beat which is already being performed calls explicitly for another
beat to be performed with a `->` marker.

A new rule in the startup rulebook, `performing opening dialogue beat rule`,
handles the first of these.

An example of using `if dialogue about X intervenes` would be a rule like so:

	Before examining a thing (called T):
		if dialogue about T intervenes, stop the action.

When the director is active, beats are also performed if there is a lull in
conversation (in that no other dialogue has been performed this turn), and if
the director can find a suitable beat to perform. "Suitable" means that:

- the beat either has not been performed before or is recurring, and
- all required speakers for the beat are within earshot, and
- any `if`/`unless` conditions are met, and
- any sequencing conditions are met.

The director will if possible try to find a suitable beat which is also relevant
to the currently live conversational subjects, except that it will not
choose a beat for which the player is the first speaker unless that beat
is marked as `involuntary`.

If the director fails to find such a beat, it will next try to find a suitable
beat which is marked as `spontaneous`, and will perform that. This is considered
a complete change of subject, so if the director has to do this then it will
also empty the list of live conversational subjects.

Failing both attempts, the director will give up.

A new rule in the turn sequence rulebook, `dialogue direction rule`,
handles all of this.

The director sometimes has to manage quite a complex situation, especially
if dialogue beats are causing each other to be performed. The new debugging
command DIALOGUE causes the director to print out explanation of what it is
doing; DIALOGUE ALL causes even fuller ones.

## Live conversational subjects

The director is always tracking the list of live subjects, even though it
only makes use of this list in active mode. So if it is switched into
active mode having been passive up to now, it may be starting with some
subjects already live.

As values, subjects have to be objects of some kind: rooms, people, things.
As a convenience for representing abstract ideas such as "philosophy", which
may still be talked about, see the proposal [IE-0010](0010-concepts.md) on
Concepts.

This is a list best thought of as pretty ephemeral, and with a rapid turnover.
Newly raised subjects appear at the beginning, and less fresh ones at the end.
The list is truncated so that it can never exceed 20 subjects: if it does, the
oldest is silently forgotten. If the director (in active mode) starts a
spontaneous dialogue beat - in effect, changing the subject altogether - the
list is wiped altogether.

Subjects automatically become live when a line `mentioning` them is performed:

	Bernardo (mentioning Barcelona): "The castles in Barcelona are warmer at night."

In addition, the following phrases allow story authors to intervene:

- `make (T - an object) a live conversational subject`. Adds `T` to the list.
- `make (T - an object) a dead conversational subject`. Removes `T` if present.
No error is produced if `T` was already not there.
- `clear conversational subjects`. Empties the list. May be useful for a
story making a drastic change of subject.
- `live conversational subject list`. Produces the current list. 
- `alter the live conversational subject list to (L - list of objects)`

This could be used for devious tricks such as:

	After printing the name of (T - an object) when performing a dialogue line:
		make T a live conversational subject.

With such a rule in force, performing this line:

	Bernardo: "The castles in [Barcelona] are warmer at night."

would automatically add Barcelona to the list.

Emptying the list regularly is one way to be especially sure that no residual
topics linger impossibly long (e.g. if the story jumps from the night before to
next morning). For example, an author may want to write a rule like:

	When a scene begins:
		clear conversational subjects.

But this is not done by default: some authors like to use scenes as big wodges
of time, and others as fleeting ones, so we leave this up to authors.

One way to track the list, for debugging purposes, is to add a rule like so:

	Every turn:
		showme the live conversational subject list.

which prints this out at runtime.

## New kinds

`DialogueKit` builds several new kinds into Inform, in order to make it
possible to describe what's going on in playtext.

### Dialogue beat

`dialogue beat` is an enumerated kind. Each value represents a single beat.
Most beats are nameless in the source text; internally, those are given
intentionally obscure names such as `beat-13`, which can be printed at
runtime for debugging purposes.

Note that the British English spelling "dialogue beat" must be used, not
the US English "dialog beat".

Values of `dialogue beat` cannot be created with assertions, and only come
into being when cues are written in a dialogue sections. So an assertion
such as `The welcome beat is a dialogue beat` produces a problem message.
There is exactly one dialogue beat value at runtime for every cue written
in the source text; their enumeration sequence corresponds to the order in
which the source text declares them.

`dialogue beat` comes with the following either-or properties built in:

- `performed` or `unperformed`. Has this been performed yet?
- `recurring` or `non-recurring`. Can the director choose this beat more
than once in the same play-through? (Note that this same property has the
same name, and a similar meaning, for scenes and lines.) By default, no.
- `voluntary` or `involuntary`. This affects only beats for which the
player is the first speaker, and only when the director is in active mode.
Can such a beat be triggered just because it seems relevant? A voluntary
beat can only be started in this way if the person playing the story typed
some appropriate command. By default, beats are voluntary.
- `spontaneous` or `unspontaneous`. Only has an effect when the director
is in active mode. Can the director bring this beat up out of nowhere to
fill a gap in the conversation, even though it has no relevance at the
moment? By default, no.

Note that a recurring beat does not automatically make the lines (or choices)
within it recurring: in fact, the default is that they will not be. If that
convention annoys you, the following rule will reverse it:

	When play begins:
		repeat with L running through dialogue lines:
			if L is in a recurring dialogue beat:
				now L is recurring.

Being properties, these can be changed during play using `now`:

	When the conference scene begins:
		now the fire alarm beat is spontaneous.

In addition, the following adjectives can be tested at run-time:

- `available` rather than `unavailable` if the beat currently meets its
preconditions for being performed - its after or before, if and unless conditions.
- `relevant` rather than `irrelevant` if the beat is about a topic in the
current list of live conversational subjects.
- `being performed` if the beat is currently in mid-performance.

The following phrases are available:

- `perform (B - a dialogue beat)`. Immediately performs the beat, regardless
of what subjects are live. Works whatever mode the director is in (i.e., whether
it is active or passive), and regardless of whether the beat has been performed
before, even if it is non-recurring.
- `list of speakers required by (B - dialogue beat)`. Results in a list of
objects, which is the set of speakers of lines of dialogue in `B`.
- `first speaker of (B - dialogue beat)`. The first specific person who speaks
a line in `B`.
- `opening line of (B - dialogue beat)`. The first `dialogue line` in `B`. Use
this with care: it's unlikely but just possible that `B` contains no lines.
- `showme the beat structure of (B - dialogue beat)`. For debugging. In effect,
beats are miniature programs, but can be quite intricate: this prints a listing.

The following relations are available:

- `B is about S` is true if `S` matches something in the `about` list for `B`.
Note that this can be true for multiple values of `S`. This relation is called
`topicality`.
- `B is performable to S` is true if `B` is either `recurring` or `unperformed`,
and if all of the speakers on the `requiring` list for `B` can be heard by `S`.
This relation is called `performability`.

Note that the command parser is able to recognise dialogue beat names, and of
course actions can be set up for them. For example:

	Performing is an action out of world applying to one dialogue beat.

	Carry out performing:
		say "[the dialogue beat understood] requires [list of speakers required by the dialogue beat understood].";
		perform the dialogue beat understood.

	Understand "perform [dialogue beat]" as performing.

Which creates a debugging command PERFORM.

### Dialogue line

`dialogue line` is an enumerated kind. Each value represents a single line.
Most lines are nameless in the source text; internally, those are given
intentionally obscure names such as `line-13`, which can be printed at
runtime for debugging purposes.

Values of `dialogue line` cannot be created with assertions: see the similar
discussion for beats above.

`dialogue line` comes with the following either-or properties built in:

- `narrated` or `unnarrated`. Is this a `Narration:` line?
- `performed` or `unperformed`. Has this been performed yet?
- `recurring` or `non-recurring`. Can the director perform this line more
than once in the same play-through? (Note that this same property has the
same name, and a similar meaning, for scenes and beats.) By default, no.
- `elaborated` or `unelaborated`. Is the speech more than a simple piece
of reported speech?

The `elaborated` property is initially set by the Inform compiler. For
example:

	Katie: "I suppose."
	
	Katie: "Your daughter gives her most grown-up pout. 'Whatever.'"

The first line there is unelaborated, the second elaborated - because it
contains internal quotation marks, so it's read as being a mixture of
narration and speech, rather than purely being speech. It affects only how
the line is performed. An unelaborated line is printed as `Speaker: "Speech."`,
whereas an elaborated line is printed exactly as written, the assumption
being that it will contain its own indication of who's speaking and which
words are actually said. So we see:

	Katie: "I suppose."
	
	Your daughter gives her most grown-up pout. "Whatever."
	
To qualify as elaborated, a speech must have at least two quotation marks `'`
at word boundaries. The source text can override this:

	Katie (unelaborated): "You call this 'peanut butter' but it's really icky."

Or, of course, the properties could be changed during play, or indeed the way
that lines are performed could be changed using the activity (see below) so
that elaboration is never taken into account.

In addition, the following adjectives can be tested at run-time:

- `available` rather than `unavailable` if the line currently meets its
preconditions for being performed - its if and unless conditions.
- `non-verbal` rather than `verbal` if it is a non-verbal communication,
written as being performed `without speaking`.

The following phrases are available:

- `textual content of (L - dialogue line)`. Produces a `text` of the speech.
It is only really safe to use this, in general, in a rule belonging to the
performing activity rulebooks, because the text may make reference to the
current `speaker` and `interlocutor`, for example - variables which exist
only inside of those activities.

Note that there is no phrase to produce the speaker or interlocutor of a
dialogue line, because not all lines give these in any explicit way. For
example, a line could be written:

	A woman other than Katie (to an animal): "Do you need feeding now?"

The following relation is available:

- `L is in B` if the dialogue line `L` (see below) is part of the beat `B`.
The existence of this relation makes it possible to iterate over the contents
of `B` (`repeat with L running through dialogue lines in B`), or extract, say,
`the list of dialogue lines in B` or `the number of dialogue lines in B`.
Of course a flat list cannot convey the full richness of structure in `B`,
but might still be useful:

	repeat with L running through dialogue lines in the opening beat:
		now L is unperformed.

### The performing activity for dialogue lines

The `performing` activity has the following definition:

	Performing something is an activity on dialogue lines.

and has three activity variables: `speaker`, `interlocutor` and `style`.
For narration, the `speaker` will be nobody, but otherwise it will be a
definite person. (By the time this activity is running, a definite choice
has been made about who is speaking.) The `interlocutor` is by default
nobody, unless the line is marked as spoken `to` somebody. The `style`
is a `performance style` value (see below), and by default is `spoken normally`.

The actual performance is carried out by the following rule:

	For performing a dialogue line (called L)
		(this is the default dialogue performance rule):
		if L is narrated or L is elaborated or L is non-verbal:
			say "[textual content of L][line break]";
		otherwise:
			say "[The speaker]";
			if the interlocutor is something:
				say " (to [the interlocutor])";
			say ": '[textual content of L]'[line break]".

Note that this ignores the `style`, by default: see below. The existence
of this activity makes it easy to customise how dialogue looks on screen:
for example, it could display little pictures of the speakers, or do more
elaborate JavaScript trickery when presented as a website. As a very
simple example:

	For performing a narrated dialogue line (called L):
		say "[italic type][textual content of L][roman type][line break]".

Note that the `speaker` and `interlocutor` might be inanimate objects:
people do sometimes talk to microphones, and phones talk to people. So
the above rule is careful not to refer to them using `somebody` rather
than `something`, as that might exclude these.

### Dialogue choice

`dialogue choice` is an enumerated kind. Each value represents a single choice.
Most lines are nameless in the source text; internally, those are given
intentionally obscure names such as `choice-13` or `flow-21`, which can be
printed at runtime for debugging purposes.

Values of `dialogue choice` cannot be created with assertions: see the similar
discussion for beats above.

`dialogue choice` comes with the following either-or properties built in:

- `performed` or `unperformed`. Has this been chosen yet?
- `recurring` or `non-recurring`. Can the director offer this choice more
than once in the same play-through? By default, no.

Note that flow markers such as `<-` and `-> perform the broken window beat`
are internally represented as dialogue choices too, even though they are
mandatorily taken and are not offered to the player to think about. This
can actually be useful, since it means flow markers can have names, and
conditions attached to them, and so on.

In addition, the following adjectives can be tested at run-time:

- `flowing` rather than `offered` if the choice is a `->` or `<-` flow
control marker rather than an option offered to the player.
- `story-ending` if the choice is an `-> end the story ...`.

The following relation is available:

- `C is in B` if the dialogue choice `C` is part of the beat `B`.
The existence of this relation makes it possible to extract, say,
`the list of unperformed dialogue choices in B`.

### Textual decisions and the current choice list

A "decision" is needed when the director is performing a beat and comes to
a run of choices. (Offered choices, that is: flow markers it will act on
without the player's intervention.)

For example:

	Border agent: "Business or pleasure?"
	
	-- "Business."
	
		Agent: "Then where is your C-34d section (xciii) visa exemption waiver proffer certificate D?"

	-- "Pleasure."
	
		Agent: "Fill out the address of your hotel and the name of your vessel, like it's the 1930s still."

After performing the border agent's first line, the director sees that there
are two possible options to choose from. These options are inspected one at
a time to see if they are actually available: if an option has been performed
before (and is non-recurring) then it won't be, for example. As they are found
to be available, they are accumulated into the "current choice list".

This can be accessed with the phrase `current choice list`, in fact, and that
enables options to be dependent on what previous options have already been
found. For example:

	-- "Hurl myself at the window."

	-- "Tunnel through the wall."

	-- (if the current choice list is empty) "Admit to being out of ideas."

The first time this decision is performed, two options will be available -
the first two. The player will choose one. If the decision is performed
again, the player will be offered the one not chosen earlier. Should the
decision come up a third time, the new option "Admit to being out of ideas."
will be offered instead.

If the choice list does end up empty, the director will skip the decision
entirely, printing nothing. Otherwise, it must ask the player what to do,
and for that it uses:

	Offering something is an activity on lists of dialogue choices.

The default behaviour is:

	For offering a list of dialogue choices (called L)
		(this is the default offering dialogue choices rule):
		let N be 1;
		repeat with C running through L:
			say "([N]) [textual content of C][line break]";
			increase N by 1.

which produces a simple numbered list.

### Performance styles

`performance style` is another enumerated kind of value. By default, it
has just one value: `spoken normally`. But the user can create additional
styles with assertions in the usual way, like so:

	Spoken angrily and spoken softly are performance styles.

Performance styles exist as a hook for ambitious systems which, for example,
change character art or pose for speakers who become angry, or which apply
emotional parameters to autogenerated voice performance, in the event the game
is using a text-to-speech engine.

So by default they do nothing. Here is a very simple use:

	Spoken furiously is a performance style.

	Before performing a dialogue line (called L):
		if the style is spoken furiously, say "[bold type]".

	After performing a dialogue line (called L) :
		if the style is spoken furiously, say "[roman type]".

And then, say:

	Elizabeth: "We had to pay the ransom, you know."

	James (spoken furiously): "Even I wouldn't pay fifty thousand pounds for me!"

Inform will not allow the following words to be used after "spoken" in the
names of instances of this kind:

	"and", "or", "if", "unless", "before", "after", "now", "to", "this", "without"

So for example it will reject:

	Spoken before thinking is a performance style.

This is to prevent ambiguity: see below.

## The complete syntax for a cue to a dialogue beat

The cue sentence(s) must be placed in round brackets, and should end with its
full stop inside the brackets. This is an error:

	(About the paranormal).

The cue must contain at least one sentence, but must not contain a paragraph
break. Each sentence must be one of the "clauses" outlined below. An extensive
example:

	(About the Ghost. After the haunting beat. If the Ghost is in Elsinore.
	Recurring. This is the Marcellus gets anxious beat.)

As syntactic sugar, semicolons can be used instead of full stops. For example:

	(After the haunting beat; about the paranormal; recurring.)

In the initial draft of this proposal, commas were also allowed as syntactic
sugar to divide clauses, but this led to too much ambiguity in practice.

### The name clause for a cue

Each beat provides a value for the kind `dialogue beat` (see above), but most
beats are nameless. By providing a sentence like this, the value is given
a name. For example:

	(This is the Marcellus gets anxious beat.)

The name here is `Marcellus gets anxious beat`. The name has to end with
`beat` or `scene`, in English at least, for namespace reasons, and has to be
unique: two different beats cannot have the same name.

If the name ends in `scene`, Inform creates both a beat and a scene. For
example,

	(This is the fire drill scene.)

creates both a beat called `fire drill beat` and also a scene called `fire drill scene`.
The two are different values (of kinds `dialogue beat` and `scene` respectively),
but their fates are tied together. The scene automatically starts and ends when
the beat begins and ends its performance, and vice versa. (If some other
condition causes the scene to end while the performance is mid-way, then the
performance stops right there.)

The name `starting beat` is reserved. If a beat is given this name, then
it is performed at the start of play, as noted above.

There is no analogous `ending beat`: stories can have multiple endings. But see
`-> end the story...` in the description of flow markers below.

### The about clause for a cue

`About` clauses tell the director what a beat of dialogue is talking about.

For example:

	(About the ghost and Elsinore.)

	Horatio: "The battlements of Elsinore have been lately haunted, 'tis true."

At its simplest, an about sentence consists of the word `about`, followed by
a list of one or more conversation subjects -- in this example, two. Subjects
can be any kind of object; see the discussion of the current list of live
subjects above.

`About X, Y and Z` means that the beat involves all of the subjects `X`, `Y`
and `Z`. This means that it can be performed when any of those subjects
are live, and that once it has been performed, the others all become live.
This enables conversation to flow, by allowing those other subjects to
be talked about in subsequent beats. For example, the first beat below
is `about Velma and the robbery`. As we begin, Velma is live, but the
robbery is not. Performance of the first beat changes that:

	(about Velma and the robbery)

	Fred: "Velma totally hasn't been the same since that bullion robbery."

	(about the robbery)

	Fred: "Velma used to work the Lufthansa cargo desk at Stuttgart, and..."

Note that `About` clauses can also give vague descriptions. For example:

	A concept can be frightening or safe.
	The paranormal is a frightening concept.
	Garden design is a safe concept.

...then:

	(About any frightening concept.)

would have the beat considered if the paranormal were in play, but not for
garden design. Similarly:

	(About any woman in the Dining Room.)

Or one might, for example, create a `discrediting` relation from concepts to
people, and then have:

	(About any concept which discredits Gordon.)
	
	Douglas (to Carolyn): And you say the marriage wasn't a success?

Descriptions like this will match the current live subject list, for purposes
of choosing relevant beats, but will not add to the live list when the beat
is performed. For example:

	(About a dwarf.)
	
	Snow White: "I have eight? no, seven indentured miners right now. No
	point remembering their names, they die so quickly. I just go by
	appearances. The present lot are Doc, Grumpy, Happy, Sleepy, Bashful,
	Sneezy, and Dopey. Might let Sneezy go, actually, don't want a sudden
	cave-in to knock out all seven at once. You can only buy fresh dwarfs
	on Wednesdays, you see, so one bad sneeze on a Thursday and..."

...would match any of the dwarfs for purposes of relevance, but not make
any of them live topics when the beat is performed. Writing it this way, on
the other hand, would do:

	(About Doc, Grumpy, Happy, Sleepy, Bashful, Sneezy, and Dopey.)

And then, for example:

	(About Bashful.)

	Snow White: "He's a varmint like all the rest of them."

If a beat is bringing up unexpected new information, which should not be
matched for relevance purposes when the beat is being chosen, `mentioning`
should instead be used on the dialogue itself. For example:

	(About a dwarf.)

	Snow White (mentioning diamonds): "He's a varmint like all the rest, but
	the diamonds are in these crazy low tunnels, and I can't afford to muss
	my hair. I'm still super-active on the pageanting circuit, you know."

The original draft of this proposal had a more convoluted syntax here which
allowed `about either ... or ...`, had a keyword `any`, and so on, but that
now seems fussy. (An eighth dwarf in waiting.)

### The if and unless clauses for a cue

These restrict the availability of dialogue beats, making them only available
when the given condition is met (or not met):

	(If Denmark is rotten.)

	(Unless Hamlet has the skull.)

### The after, before, later, next, and immediately after clauses for a cue

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
	
	(Later; about chips.)
	...

means the same thing as

	(About fish. This is the fishmonger beat.)
	...
	
	(After the fishmonger beat; about chips.)
	...

`Immediately after` is equivalent to `after`, except that it allows the
beat to be performed only the very next turn (in parser IF terms) after
the named beat was performed. The one-word `Next` is the corresponding
version of `Later`, applying `immediately after` to the previous beat.
For example:

	(About Pygmalion and mentioning Cyprus.)

	Galatea: "I haven't seen him around since I left Cyprus."

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

(`Immediately before` is, of course, impossible.)

### The requiring clause for a cue

Consider this beat:

	(about the Zeppelin)
	
	Hans: "See our prodigious rate of climb, Ludwig!"
	
	Ludwig: "Jawohl, Hans."

There would not be much point in performing this beat unless Ludwig and Hans
were within earshot of the player, because none of the lines could actually
be spoken, and so it would complete silently.

Because of that, each beat has a list of "required speakers", and the director
when in active mode will only select a beat for performance if all of those
speakers are present (more exactly, if the player can hear them).

By default, the required speakers are all those whose names explicitly appear
as speakers of lines within the beat. But if that is not what the author
wants, the cue can be more explicit:

	(about the Zeppelin; requiring Hans)
	
	Hans: "See our prodigious rate of climb, Ludwig!"
	
		Ludwig: "Jawohl, Hans."

This beat can now play even if Hans is not there, though of course he would
then receive no reply.

When the director is in passive mode, this list is ignored, though it can still
be accessed using the phrase `list of speakers required by (B - dialogue beat)`
(see above).

The original draft of this proposal had a more complicated algorithm for
determining the required speaker list, but it seems better for the author
to have to write explicitly what's wanted in any case of possible doubt.

### Supplying properties as clauses for a cue

The name of a property which a `dialogue beat` can have (an either-or
property, or conceivably a value of an enumerated property) can be a
cue sentence all by itself. In particular, this means that:

- The cue `(Recurring.)` gives the beat the `recurring` property. This means
that it can be performed more than once.

- The cue `(Spontaneous.)` gives the beat the `spontaneous` property. A spontaneous
beat can be chosen by the director, when the director is active and looking
for conversations to start, even if it has no `about` subjects.

But note that new properties for dialogue beats can also be created with
regular Inform sentences like:

	A dialogue beat can be testing only or meant for production.

And this enables beats to be tagged in arbitrary ways:

	(Testing only.)
	
	Horatio: "Sorry to break the fourth wall, but will this program ever work?"		

## The complete syntax for a dialogue line

A dialogue line occupies a single complete paragraph, and takes one of these forms:

	SPEAKER: "TEXT."

	SPEAKER (PERFORMANCE DETAILS): "TEXT."

If given, the performance details are one or more clauses, divided by full stops
or semicolons, and placed in round brackets after the speaker name. Thus:

	SPEAKER (CLAUSE1; CLAUSE2; ..., CLAUSEn): "TEXT."

### The speaker

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
	The tallest person in the Dining Room:

If multiple objects would match the description, the object maximising the
following score is chosen:

	+16 for being on-stage (i.e. in play)
	+8 for being audible to the player
	+4 for being the interlocutor of the previous speech in the beat
	+2 for being of the kind "person"
	+1 for being different to the previous speaker in the beat

If multiple objects still have equal scores, a random choice is made from
those objects. If none at all match, the line of dialogue is skipped and not
performed at all, but no run-time problem is thrown. Because it was not
actually said, it is not given the `performed` property.

### The name clause for a line

Each line provides a value for the kind `dialogue line` (see above), but most
lines are nameless. By providing a clause like this, the value is given
a name. For example:

	Marcellus (this is the reluctant admission line): "Something is rotten."

The name has to end with `line`, in English at least, for namespace reasons,
and has to be unique: two different lines cannot have the same name.

Most lines never need naming, of course, but this can be useful for testing,
say, `if the reluctant admission line is unperformed`.

### The to clause for a line

`to` plus a description of an object specifies the "interlocutor" for the
speech - the person to whom it's specifically addressed. Most lines are
delivered to the whole room in general, and have no interlocutor, so this
clause is optional. It's useful for lines which make sense only in the
presence of somebody else. For example:

	Marcellus (to Horatio): "The Prince looks moody tonight, i'faith."

The line will not be performed unless Marcellus can hear Horatio.

### The mentioning clause for a line

`mentioning ...` says that the line makes a conversation subject live
when it is performed. For example:

	Marcellus (mentioning the ghost): "I be mighty afear'd of the Ghost."

A whole list of subjects can be given: `mentioning the Ghost and Denmark`
would give two, for example.

### The if and unless clauses for a line

`if` or `unless` plus any Inform condition. The line is omitted as being
unavailable if these conditions fail. It is then skipped and not performed
at all, but no run-time problem is thrown. Because it was not actually said,
it is not given the `performed` property.

### The before and after clauses for a line

These clauses say that an Inform action should be tried along with the
performance of the line. Note that `before` means the line is performed
before the action, and `after` means it is performed after.

Unless otherwise specified, the actor of the action is the speaker, so this
is an exception to the general rule of Inform that the default actor is the
player. For example:

	Gravedigger (after taking the shovel): "Here goes another shallow one."

would be performed as:

	The gravedigger takes the shovel.
	
	Gravedigger: "Here goes another shallow one."

In the case of `after`, the action must succeed, or else the line is not
performed after all. For example, if some rule made it impossible for any
person to take the shovel, the Gravedigger would not in fact get it, and
therefore his line would make no sense: so it would not be performed.

The actor need not be the speaker, but the action has to succeed whoever the
actor may be:

	Gravedigger (to Hamlet, after Hamlet silently taking the shovel):
		"Hey, I saw you picking up that shovel! That's a union job, sweet Prince!"

As with `try`, the keyword `silently` can also be used:

	Gravedigger (after silently taking the shovel): 
		"'Here goes another shallow one,' says the thin man, picking up the shovel."

This would suppress the uninteresting text "The gravedigger takes the shovel.".

### The now clause for a line

`now` plus a condition allows the world model to be changed immediately after
the line is performed. For example:

	Marcellus (now Marcellus is in the Banqueting Hall): "Oh my. I'm running for it!"

It's legal to pile up multiple `after`, `before` and `now` clauses on the
same line, at your own risk:

	Marcellus (after taking the ghost detector; after examing the ghost; now
	Marcellus is in the Banqueting Hall; before jumping):
		"Marcellus grabs the equipment, takes a quick reading, and panics.
		'Oh my. I'm running for it!' And he makes a running jump."

The sequence is: `after` actions first, in declaration order; then actual
performance of the line; then `now` effects; then `before` actions, in
declaration order. If any of the `after` actions should fail, the process
stops there.

### The without speaking clause for a line

`without speaking` makes the line non-verbal: something like a gesture, which
is still performed by somebody.

	Marcellus (without speaking): "Marcellus throws up his hands, appalled."

The difference between this and `Narration:` is that there is a speaker,
`Marcellus`. Because this is non-verbal, he needs to be visible, not audible.
Similarly there can be an interlocutor, who must be visible to the speaker
rather than audible to him. For example:

	Marcellus (without speaking, to Bernardo): "Marcellus points a horrified finger, and nudges Bernardo."

In the event that you need audibility after all, you can still finagle things:

	Marcellus (elaborated, to Bernardo): "Marcellus points a horrified finger, and nudges Bernardo."

But this is not an elegant device.

### The style clause for a line

Using just the name of a `performance style`, but with the word `spoken` removed,
indicates that the line is performed this way. By default it is performed in
the style `spoken normally`, as if the clause `normally` had been written. A line
must have a single style.

Thus, if the user has created the style `spoken with asperity`, then:

	Marcellus (with asperity): "I've had it with these goddam ghosts."

This will set the activity variable `style` to `spoken with asperity` when the
line is performed: see above.

### Supplying properties as clauses for a line

The name of a property which a `dialogue line` can have (an either-or
property, or conceivably a value of an enumerated property) can be a
clause all by itself. In particular, this means that:

- `Marcellus (recurring): ...` gives the line the `recurring` property. This means
that it can be performed more than once.

### The speech text for a line

The performed line is an Inform text, in double-quotes.

This can contain text substitutions in square brackets, in the usual way for
Inform text, giving access to an enormous range of effects.

When such text is printed, the activity variable `speaker` will exist for it,
and will hold the identity of the speaker performing the line. (If the line
is narration, it will still exist, but will be equal to `nothing`.) Similarly
for `interlocutor` and `style`. So for example:

	A person in the Lounge:
		"[The speaker] self-importantly [declare]: 'The greatest TV show of all
		time is [if the speaker is female]Gilmore Girls[if not]24[end if].'"

might be performed as:

	Henry self-importantly declared: "The greatest TV show of all time is 24."

## The complete syntax for a dialogue line

During a beat, dialogue ordinarily flows without stopping (see "continuity
of performance" below), but will pause at points where a decision from the
player is called for.

Decisions consist of a run of one or more choices written `-- ...`. There
are three forms of choice: textual, action-based and automatic.

	-- (DETAILS) "TEXT"

	-- (DETAILS) after ACTION-DESCRIPTION

	-- (DETAILS) before ACTION-DESCRIPTION

	-- (DETAILS) instead of ACTION-DESCRIPTION

	-- (DETAILS) otherwise

A decision cannot have a mixture of the two sorts of option. In the first
sort, there's a run of textually presented options, like this:

	-- "Pretend everything is fine"
	
	-- "React with horror"

This is the sort of choice offered by the "offering activity" (see above).

In the second sort, there's a run of action-matching options, like so:

	-- after examining something

	-- before taking or pushing the ball

	-- otherwise

With this second sort, there's no offer as such: the player can type a command
in traditional command-parsing form, and then we see what she has done and
react accordingly. An `-- otherwise` clause matches only if none of the others
do, and is optional. It is a shorthand for `-- before doing something` if the
previous choices included a `-- before ...`, or for `-- instead of doing something`
if the previous choices included an `-- instead of` but no `-- before ...`,
or for `-- after doing something` if all the previous choices were `-- after ...`.

In each case the `(DETAILS)` are optional, and consist of one or more clauses.
As with beats and lines, these can be divided by full stops or semicolons.

The third sort of choice is "automatic", which means that the dialogue system
chooses an option automatically, i.e., without consulting the player in any way.
For example:

	-- step through

		Liza: "One."

	-- or

		Liza: "Two."
	
	-- or

		Liza: "Three."

In an automatic choice, the first choice explains how to make the decision, and
the rest must all be just `-- or`. In this case, what `step through` means is
that the first time this choice is encountered, the first option is chosen (so
Liza says "One."); the second time, the second option; the third time, the third
option; and on the fourth and subsequent times, nothing is done.

An alternative is:

	-- cycle through

		Liza (recurring): "One."

	-- or

		Liza (recurring): "Two."
	
	-- or

		Liza (recurring): "Three."

This is like `step through` except that the fourth time, we go back to Liza
saying "One.", and the fifth time she says "Two.", and so on, round and round.
Note that the choices in an automatic choice are always recurring, but that
any dialogue within them is not. This is why we have to mark Liza's lines as
`recurring` here.

There are altogether five of these automatic mechanisms:

- `step through`: each in turn, but only once through, then do nothing
- `step through and stop`: each in turn, but repeat the last one indefinitely
- `cycle through`: each in turn, going back to the start after the last
- `shuffle through`: a random permutation, and then another random
permutation when that completes, and so on
- `choose randomly`: make an entirely random choice every time

### The if and unless clauses for a choice

`if` or `unless` plus any Inform condition. The choice is omitted as being
unavailable if these conditions fail.

When such conditions are being tested, the phrase `current choice list` produces
the list of those found to be available so far. (See above.) This allows for
decisions to reshape themselves with the circumstances: compare the function
`CHOICE_COUNT()` in Ink, which is often used for similar reasons. `CHOICE_COUNT()`
would be equivalent to taking the length of the `current choice list`.

### The name clause for a choice

Each choice provides a value for the kind `dialogue choice` (see above), but most
choices are nameless. By providing a clause like this, the value is given
a name. For example:

	-- (this is the desperate choice) "Run for it in blind hope"

The name has to end with `choice`, in English at least, for namespace reasons,
and has to be unique: two different choices cannot have the same name.

Most choices never need naming, of course, but this can be useful for testing,
say, `if the desperate choice is unperformed`.

### Supplying properties as clauses for a choice

Exactly as for beats and lines, the name of a property which a `dialogue choice`
can have (an either-or property, or conceivably a value of an enumerated
property) can be a clause all by itself. For example,

	-- (recurring) "Go back to the visitor centre"

would be offered every time this decision came up, rather than being offered
only until the player had chosen it for the first time.

### Action decisions

These are only suitable for interactive fiction which has some sort of interface
allowing the player to choose actions: the traditional example, of course,
being keyboard input run through a command parser.

The director pauses its work and allows the game to go through its usual turn
cycle: for parser IF, that means printing a command prompt, accepting keyboard
input, action processing, every turn rules, and so on. Once that is done, the
action just processed is used to determine the choice made, and the beat
continues.

Note that the action is indeed processed, so if the player's command called
for things to happen, then they will indeed happen. For example:

	-- after examining a door
	
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
either `before` or `instead of`. Thus:

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

With a run of action choices, an `-- otherwise` is chosen if none of the
other choices matched. If there is no `-- otherwise`, and none of the choices
match, then the decision is left hanging for another turn and reconsidered then.

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
		
		<-

Note the position of the `<-` flow marker, indented underneath the `-- otherwise`.
This little conversational predicament continues until the player puts
Clark out of his misery, one way or another. Had the `<-` been unindented,
the result would be that the whole thing goes on forever.

## Flow markers

Flow markers are written `<-` or `->`. These are simple but very useful.
They also support all of the clauses which dialogue choices can take: in
particular, they can be conditional. For example,

	<- (unless Polonius is in Elsinore)

	-> (if Hamlet is not in the Graveyard) stop

There are intentionally few forms of flow marker: it's important for users
of dialogue to find them easy to understand. 

Inform issues problem messages in response to various nonsensical ways of
using flow markers: for example, using `-> another choice` somewhere other
than immediately sandwiched between decisions, or placing material which
follows `<-` or `-> stop` at the same indentation level.

### Flow back

`<-` on its own goes back to either the most recent decision point in the
same beat, or (if there hasn't been a decision) to the start of the beat.

As with control mechanisms in all programming languages, that makes it
possible to get into endless loops:

	(This is the ill-advised beat.)
	
	Fatboy Slim (recurring): "Right about now, the funk soul brother."
	
	Fatboy Slim (recurring): "Check it out now, the funk soul brother."

	<-

But it is extremely useful for situations like this one:

	-- "Option which turns out not to work."
	
		Recording angel: "That didn't work. Try again."
		
		<-

	-- "Another option which turns out not to work."
	
		Recording angel: "No, try again."
		
		<-

	-- "The one option which works."
	
		Recording angel: "Well done."

In effect this repeats the decision up to three times until the player heads
down the track we want.

### Flow through another beat

`-> perform B`, where `B` is the name of a beat, causes the director to
perform that beat at this point. Note that after `B` finishes, the original
beat continues where it left over. This nesting can go up to 20 beats deep.

Again, it's a bad idea to do this:

	(This is the equally ill-advised beat.)
	
	Fatboy Slim (recurring): "Right here, right now."
	
	-> perform the equally ill-advised beat

Instead, it's good to use `-> perform ...` as a way of incorporating what
amounts to a complicated sub-scene which is only needed in some situations.

	-- "Climb the chimney"
	
		Narration: "You get only a few feet, and covered in soot."
		
		<-
		
	-- "Get into the wardrobe."
	
		Narration: "Hmm, there seems no end to these fur coats."
		
		-> perform the Narnia visit beat

		Narration: "Goodness, back at that wardrobe you only dimly remember."

### Flow to a stop

`-> stop` causes the current beat to finish right here, and is convenient
when a drastic choice short-circuits what might otherwise have been a
lengthy conversation.

	-- "Run for the Numidian desert and live off locusts and honey"
	
		-> stop

	-- "Ask if the eternal fate of the soul is determined at death"
	
		Augustine: "The purgatorial fires purify only those who die in communion."
		
		-- "But ..."

		...

Note that it stops only the current beat. If one beat is performing another one,
and the second one stops, the first one then resumes.

### Flow to the end of the story

There are four variants of this:

	-> end the story          
	-> end the story finally
	-> end the story saying "TEXT"
	-> end the story finally saying "TEXT"

These correspond exactly to the Inform phrases which `end the story`. All
beats are abandoned at this point, of course, so this is even more drastic
than `-> stop`.

### Flow to another choice

Finally, and only seldom needed, `-> another choice` can be used to clear
up an occasional ambiguity. For example:

	Narration: "What is your favourite colour?"

	-- "Blue"
	
		Narration: "You are a Democrat or a Conservative voter."

	-- "Red"
	
		Narration: "You are a Republican or a Labour voter."

	-> another choice
	
	-- "Accept this verdict"

		Narration: "You should. I am the infallible author."
	
	-- "Dispute this verdict"
	
		Narration: "You shouldn't. I am the infallible author."

This makes clear that there are two decisions of two options each. Without
it, we would have one decision of four options. In practice, this very seldom
arises, because normally there is some narration or dialogue in between the
two decisions anyway.

## The talking about action

A new action, `talking about`, represents speakers attempting to make
conversation about a given topic. The actor is the speaker, and this need
not be the player. The topic has to be an object, but this can be, for example,
a room or a concept (in the sense of [IE-0010](0010-concepts.md)) instead of
something tangible.

The best way to document this is probably just to show its implementation:

	Talking about is an action applying to one object.

	The talking about action has a list of dialogue beats called the leading beats.

	The talking about action has a list of dialogue beats called the other beats.

	Before an actor talking about an object (called T):
		repeat with B running through available dialogue beats about T:
			if B is performable to the actor:
				if the first speaker of B is the actor:
					add B to the leading beats;
				otherwise:
					add B to the other beats;

	Carry out an actor talking about an object (called T)
		(this is the first-declared beat rule):
		if the leading beats is not empty:
			perform entry 1 of the leading beats;
			continue the action;
		if the other beats is not empty:
			perform entry 1 of the other beats;
			continue the action;
		if the player is the actor:
			say "There is no reply.";
			stop the action;
		otherwise:
			say "[The actor] [talk] about [T].";
			stop the action.

That is: an action such as `Kafka talking about the trial` will cause two lists
of dialogue beats to be made. `leading beats` will list all the beats which
could conceivably be performed next, and which have Kafka as the first speaker;
`other beats` will be those which have somebody else speaking first.

The `first-declared beat rule` then chooses which to beat perform. This picks
the first leading beat if there is one, and then the first other beat if not,
and otherwise gives up.

Authors of dialogue-based stories may well want to replace this rule with
variations of their own. For example, this:

	First carry out an actor talking about an object (called T):
		if the leading beats is not empty:
			let N be the number of entries in the leading beats;
			if the actor is not the player:
				perform entry 1 of the leading beats;
				continue the action;
			if N is 1:
				perform entry 1 of the leading beats;
				continue the action;
			say "Choose...";
			say "[conditional paragraph break]";
			let X be 0;
			repeat with B running through the leading beats:
				increase X by 1;
				say "([X]). [textual content of opening line of B][line break]";
			say "[conditional paragraph break]";
			let Y be the chosen dialogue number up to X; 
			perform entry Y of the leading beats;
			continue the action;
		if the other beats is not empty:
			let N be the number of entries in the other beats;
			let R be a random number from 1 to N;
			perform entry R of the other beats;
			continue the action;
		if the player is the actor:
			say "There is no reply.";
		otherwise:
			say "[The actor] [talk] about [T].";
		stop the action.

...offers the player a choice if there are multiple possible beats he/she
could originate, and if it has to resort to the `other beats` it makes a
random selection rather than always choosing the first.

## Exporting dialogue for voice performers or localisation

(This feature is currently unimplemented.)

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
