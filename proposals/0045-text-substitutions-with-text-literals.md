# (IE-0045) Text substitutions with text literals

* Proposal: [IE-0045](0045-text-substitutions-with-text-literals.md)
* Discussion PR link: [#45](https://github.com/ganelson/inform-evolution/pull/45)
* Authors: Graham Nelson
* Status: Implemented but unreleased
* Related proposals: --

## Summary

This change to Inform's lexical syntax enables text literals to appear inside
text substitutions for the first time, making it possible for a substitution to
use a phrase which takes text arguments. Since those text literals can
themselves contain substitutions, substitutions can for the first time be nested.

## Motivation

Suppose this has been defined:

	To say intone (T - text):
		say "[T], [T], [T]".

Then consider this text literal:

	"The bishop says: [intone X]."

Provided `X` is the name of a `text` variable, this will work. This compiles:

	When play begins:
		let X be "Amen";
		say "The bishop says: [intone X]."

and prints `The bishop says: Amen, Amen, Amen.` But it is clumsy to have to
create the spurious local variable `X`. The situation is worse outside the
context of a phrase:

	The whispering sound is always "whisper".

	The Cathedral is a room. "In this echoey space, you can endlessly
	hear the [intone whispering sound] of priests."

In each case we had to resort to creating a name (of a variable or constant)
because there was no way to incorporate a text literal into the text
substitution. This:

	"The bishop says: [intone "Amen"]."

fails because the first `"` truncates the text literal to `The bishop says: [intone `,
and then the remaining characters throw problem messages.

This restriction has long been a slight nuisance, and is particularly annoying
for text substitutions defining hyperlinks. This, for example,

	The prayer board is here.
	"The [link to command "EXAMINE BOARD"]prayer board[end link] catches your eye."

fails to compile because the quotation marks around `"EXAMINE BOARD"` are
syntactically invalid in Inform text literals.

## Components affected

- [x] Changes to natural-language syntax.
- [ ] No change to inbuild.
- [x] Changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [x] Changes to documentation.
- [ ] No change to the GUI apps.

## Change to text literal syntax

In this proposal, text literals are allowed in substitutions between `[` and `]`,
but need to use the backtick `` ` `` instead of the double-quotation mark `"`
as delimiter. The two examples above become:

	The Cathedral is a room. "In this echoey space, you can endlessly
	hear the [intone `whisper`] of priests."

	When play begins:
		say "The bishop says: [intone `Amen`]."

This proposal will call these nested text literals _backticked texts_. Here,
the backticked texts were `whisper` and `Amen`.

Backticked texts follow the same rules as any other Inform text literal, except
that they are ended by `` ` `` not `"`. For example, inside text literals, the
apostrophe `'` is sometimes read as if it were `"`, and this works inside
backticked texts too:

	"The bishop says: [intone `'Amen'`]."

expands to:

	The bishop says: "Amen", "Amen", "Amen".

### Nested substitutions

Being texts, backticked texts can contain substitutions, which means that we
have substitutions within substitutions. Consider:

	To say praise for (T - text):
		say "all praise [T]".
	
	When play begins:
		let x be "tennis";
		say "Firstly, [praise for x].";
		say "Then, [praise for `golf`].";
		say "Then, [praise for `'crazy' golf`].";
		say "Then, [praise for `golf not [x]`].";
		say "Then, [praise for `golf not [praise for `hammer-throwing`]`].";

This produces:

	Firstly, all praise tennis.
	Then, all praise golf.
	Then, all praise "crazy" golf.
	Then, all praise golf not tennis.
	Then, all praise golf not all praise hammer-throwing.

Note that the last of these texts:

	"Then, [praise for `golf not [praise for `hammer-throwing`]`]."

contains a backticked text inside a substitution inside a backticked text inside
a substitution inside a regular text:

	"Then, ______________________________________________________."   regular text
	       [praise for _________________________________________]     substitution
	                   `golf not ______________________________`      backticked text
	                             [praise for _________________]       substitution
	                                         `hammer-throwing`        backticked text

In principle, this nesting can go arbitrarily deep, always alternating substitution
brackets and backticks. This, for example:

	"[intone `[intone `[intone `[intone `[intone `[intone `[intone `X`]`]`]`]`]`]`]"

expands to a comma-separated list of 2187 copies of `X`.

Note that a regular text opens with `"` and ends with the earliest subsequent `"`,
whereas a backticked text opens with `` ` `` and ends with the earliest subsequent
`` ` `` _which is not nested inside square brackets_. Thus

	`golf not [praise for `hammer-throwing`]`

is a single backticked text: the four backtick characters are read as opening,
opening, closing, and closing.

### The backtick itself

The backtick character is significant only inside `[` and `]` pairs. Thus:

	"I spy `backticks`, [intone `sirrah`]."

expands to:

	I spy `backticks`, sirrah, sirrah, sirrah.

But since a backtick ends a backticked text literal, this does mean that
literal backticks can't be used in those. However, there are ways round this:

	"The clock murmurs [intone `[unicode U+0060]tick[unicode U+0060]`]."

expands to:

	The clock murmurs `tick`, `tick`, `tick`.

## Impact on existing projects

There is no impact on source texts which do not contain backtick characters
outside of double-quotation marks.

Those which do, however, are now prone to misreadings. For example, suppose
these definitions exist:

	To say `bad idea`: say "good idea".

	`dream` is a text that varies. `dream` is "dream of fairground rides".

Both of these declarations were previously legal in Inform. But in that case,
what would the following mean?

	"I had a [`bad idea`]."
	"I had a [`dream`]."
	
Under this proposal, they should print "I had a bad idea" and "I had a dream"
respectively, whereas in past releases they would have printed "I had a good idea"
and "I had a dream of fairground rides".

To mitigate this potential problem, the following restrictions have been
imposed, to head the issue off with compiler problem messages:

-	The fixed wording of a `To` phrase is now required not to contain backtick
	characters: so for example

		To say `bad idea`: say "good idea".

	now throws a problem message.

-	The names of global declarations are also forbidden to contain backticks:

		`dream` is a text that varies.
		The ```Markdown Lounge``` is a room.
	
	now throw problem messages.

Those new problems seem acceptable because they are unlikely to affect many
people, and are easy to work around by renaming.

However, backticks continue to be allowed in literal notations, because it
seems more likely that they will be wanted there. Here, we are simply going
to have to accept that in some contrived situations the meaning of existing
source code would be changed. For example:

	A map reference is a kind of value. `999` specifies a map reference.

	To test maps:
		say "The treasure is buried at [`247`]."

This would previously have printed:

	The treasure is buried at `247`.

It continues to compile but now prints:

	The treasure is buried at 247.

This scenario is fairly unlikely. It requires a notation to contain backticks
which pair properly, and then only occurs if a text substitution uses that notation
in a context allowing text as well as the kind in question. It's far more likely
that a literal notation containing backticks would now cause problem messages
if used in text substitutions: this is acceptable because the author's attention
is thus drawn to the issue.

### Note on syntax-colouring

This syntax change has been chosen to minimise the effect on existing Inform
syntax-colouring algorithms. However, nested substitutions may now look
slightly incorrectly coloured:

	"Then, [praise for `golf not [praise for `hammer-throwing`]`]."

In the past it was guaranteed that the substitution beginning `[` would
end at the next `]`: now, as in this example, it does not necessarily do so,
and so this text should ideally be syntax-coloured slightly differently.

Suppose we use three colours: `m` for normal material, `t` for text, `s`
for a substitution within text. Then for example:

	say "Look, [the location] is all lit up";
	mmmmtttttttsssssssssssssstttttttttttttttm

At present, most syntax-colourers for Inform will produce:

	"Then, [praise for `golf not [praise for `hammer-throwing`]`]."
	tttttttsssssssssssssssssssssssssssssssssssssssssssssssssssstttt

because they will assume that the `]` closing the inner substitution is
actually the close of the whole substitution. Better would be:

	"Then, [praise for `golf not [praise for `hammer-throwing`]`]."
	tttttttsssssssssssssssssssssssssssssssssssssssssssssssssssssstt
