# (IE-0018) Use options and kit configuration

* Proposal: [IE-0018](0018-use-options-and-kit-configuration.md)
* Discussion PR link: [#18](https://github.com/ganelson/inform-evolution/pull/18)
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: [IE-0013](0013-annotations-for-kit-linking.md)
* Implementation: Implemented but unreleased

## Summary

To make use options cleaner, more reliable, more flexible, and better able to
be used to configure how kits and extensions work.

## Motivation

Since 2005, Inform has enabled use options (which could be set with source text)
to configure the run-time functioning of its support code (which is written in
Inform 6 notation, not in source text). For example, `Use serial comma` changed
the way lists are printed. Such options were declared with sentences like:

	Use predictable randomisation translates as (- Constant FIX_RNG; -).
	Use hallucination time of at least 1024 translates as
		(- Constant DREAMY_TIME = {N}+3; -).

The idea here is that if the source text ever specifies `Use predictable randomisation`
then the Inform 6 code fragment `Constant FIX_RNG;` will be compiled into the
project; and if not, not. However:

* The use of `(-` and `-)` and Inform 6 notation is always clumsy, and hard
for relative novices to follow.
* But particularly so here, since we have to spell out that a constant is being made.
* If we don't do that, then arbitrary code could be written in the inclusion, with
unpredictable consequences.
* It is unclear which kit is being affected.
* It is easy to get these constant names wrong, leading to silent errors.
* Confusion easily follows if the kit wrongly makes an `#Ifdef` test on such
a constant, because the existence of the constant is not known when the kit
itself is being built. (It is known when the project is being built, but that
will be later on.) This can cause subtle errors through use options not taking
effect. See, for example, Jira bug [I7-2306](https://inform7.atlassian.net/browse/I7-2306).
* To get around that, the compiler provided hard-wired support for some use
options affecting the standard kits, but that made their ostensible declarations
misleading: and it meant that the standard kits were getting help which new
kits facing the same issue could not use.
* Contradictions due to mutually exclusive settings were not detected: for
example, if a project says both `Use fast route-finding` and `Use slow route-finding`,
perhaps because two extensions disagree about what they want.
* A few use options which really instructed the compiler about how to compile
the project, and had nothing to do with kit code, made spurious symbols as a
side-effect.

## Components affected

- [x] Some changes to the natural-language syntax.
- [x] Minor changes to inbuild.
- [x] Major changes to inform7.
- [x] Minor changes to inter.
- [ ] No change to the Inter specification.
- [x] Changes to implementation of the runtime kits.
- [x] Changes to implementation of the Standard Rules and Basic Inform.
- [x] Changes to documentation.
- [ ] No change to the GUI apps.

## New syntax for declarations

### Use options as values

### Configuration flags and values

In the new model for use options, `Use ... translates as ...` declarations
no longer use `(-` and `-)` insertions after `translates as`. Instead, they
follow these models:

	Use drifting lilypads translates as a configuration flag.
	Use horny skin translates as a configuration value.
	Use scaly skin translates as a configuration value.
	Use minimum jump height translates as a configuration value.
	Use maximum jump height of 6 translates as a configuration value.
	Use frog count of at least 10 translates as a configuration value.

A "configuration flag" is either set or not set, and can only have the values
0 or 1; a "configuration value" can have any integer value, except that those
declared as "at least..." must be non-negative, since the smallest possible
"at least" value is 0.

Flags and values are simple to set:

	Use drifting lilypads.
	Use maximum jump height of 17.

The distinction between these:

	Use minimum jump height translates as a configuration value.
	Use maximum jump height of 6 translates as a configuration value.

is that `maximum jump height` has a default value of 6 (i.e., will be that if
the source text never specifies anything), whereas `minimum jump height` has
a default value of 0.

A problem message rejects this as a contradiction:

	Use maximum jump height of 17.
	Use maximum jump height of 22.

This is legal:

	Use frog count of at least 10 translates as the configuration value FROG_COUNT.
	Use frog count of at least 175.
	Use frog count of 200.

But this is rejected as contradictory:

	Use frog count of at least 10 translates as the configuration value FROG_COUNT.
	Use frog count of at least 175.
	Use frog count of 160.

As has been true in Inform for many years, use options are values at runtime,
with the kind `use option`. But not much could be done with them. There are
now two new adjectives, `chosen` and `unchosen`, applying to them: a chosen
option is one which the source text has specified (other than in its declaration
line, that is). So if we have:

	Use fried omelettes translates as a configuration flag.
	Use pepper count translates as a configuration value.
	Use pepper count of 17.

then `pepper count` is `chosen`, but `fried omelettes` is `unchosen`. So for
example:

	When play begins:
		if fried omelettes option is chosen:
			say "I'd better get some butter.";

will print something.

In addition, the phrase `numerical value of U`, where `U` is a use option, will
give its current value. (For a flag, that will be 0 or 1.) If the option is
unchosen, the value will be the default, of course. For example:

	When play begins:
		repeat with U running through chosen use options:
			if the numerical value of U > 1:
				say "[U] has been set to [numerical value of U].";
			otherwise:
				say "[U] is on.";
		say "Not chosen: [list of unchosen use options].";

These new features make it possible both the declare and test use options
entirely in natural language, without need for any Inform 6 syntax, and should
make it easier to provide clean use options for extensions.

A small change from Inform v10 is that use option names which had numerical
values would be printed with their values appended, like so:

	dynamic memory allocation option [8192]

This was a clumsy way to get around being unable to access the value, and it
is not done any more: Inform simply prints the name as

	dynamic memory allocation option

### Configuration flags and values from inclusions

The above makes it simple to declare use options and use them in natural language,
but we still may want to access them from `(-` ... `-)` inclusions, that is,
from Inter code written in I6 syntax. To do that, simply supply an identifier
name, like so:

	Use drifting lilypads translates as the configuration flag DRIFTING_LILYPADS.
	Use horny skin translates as the configuration value SKIN_TYPE = 1.
	Use scaly skin translates as the configuration value SKIN_TYPE = 2.
	Use frog count of at least 10 translates as the configuration value FROG_COUNT.
	Use maximum jump height of 6 translates as the configuration value JUMP_HEIGHT.

The flag/value names, such as `DRIFTING_LILYPADS` or `FROG_COUNT`, must be written
without quotation marks, must begin with an upper-case letter, must consist of at
most 20 characters, and must contain only upper-case (English) letters, digits
and underscores.

In fact this may be useful even without any need to access Inter, because it
makes it possible for Inform to detect contradictions when mutually exclusive
options are set at once. For example:

	Use horny skin translates as the configuration value SKIN_TYPE = 1.
	Use scaly skin translates as the configuration value SKIN_TYPE = 2.
	Use horny skin.
	Use scaly skin.

now throws a problem message, since Inform can see that these two options
contradict each other - they would try to set the same value two different ways.

An important difference between old- and new-style use option declarations is
that the new ones ensure that the constant is always present in the Inter
code compiled, whether or not the option is used. In v10, this:

	Use drifting lilypads translates as (- Constant DRIFTING_LILYPADS; -).

caused `DRIFTING_LILYPADS` to be defined only if a project actually said
`Use drifting lilypads.` in its source text. As a result, code in
some kit trying to find this out had to test whether or not `DRIFTING_LILYPADS`
existed. For reasons to do with linking and kits being built in advance, this
was unreliable. In the new system, this:

	Use drifting lilypads translates as the configuration flag DRIFTING_LILYPADS.

causes the constant `DRIFTING_LILYPADS_CFGF` to be defined whether or not the
project ever says `Use drifting lilypads.`. If it does, the value of the constant
is 1, and if it doesn't, the value is 0. As a result, code trying to find out
should now test `if (DRIFTING_LILYPADS_CFGF) ...`, or `#iftrue DRIFTING_LILYPADS_CFGF == 1;`,
not `#ifdef DRIFTING_LILYPADS_CFGF;` -- which would always be true.

Similarly, a configuration value is always created, and by default has whatever
numerical value was given in the first relevant declaration. Thus:

	Use horny skin translates as the configuration value SKIN_TYPE = 1.
	Use scaly skin translates as the configuration value SKIN_TYPE = 2.
	Use frog count of at least 10 translates as the configuration value FROG_COUNT.
	Use maximum jump height of 6 translates as the configuration value JUMP_HEIGHT.

result in the default values `SKIN_TYPE_CFGV == 1`, `FROG_COUNT_CFGV == 10`
and `JUMP_HEIGHT_CFGV == 6`. In particular, when giving names to alternative
values for the same constant, the sequence matters. If we had written:

	Use scaly skin translates as the configuration value SKIN_TYPE = 2.
	Use horny skin translates as the configuration value SKIN_TYPE = 1.

then `SKIN_TYPE_CFGV == 2` (i.e., scaly skin) would be the default.

Note that configuration flags are always suffixed `_CFGF`, and values `_CFGV`.

Within the source text of a single extension (or within the main source text),
it's safe to use conditional compilation in `(-` ... `-)` inclusions. For
example:

	Use drifting lilypads translates as the configuration flag DRIFTING_LILYPADS.

	Include (-
		#Iftrue DRIFTING_LILYPADS_CFGF == 1;
		Array Drifts table [ "Lilies drift."; "Lilies sink."; "Lilies rise." ];
		#Endif;
		[ Greet;
			print "Mweep!^";
			#Iftrue DRIFTING_LILYPADS_CFGF == 1;
			print (string) Drifts-->(random(3)), "^";
			#Endif;
		];
	-).
	
	To greet: (- Greet(); -).

This is a very contrived example, but the point of interest is that the array
`Drifts` exists only when the use option `Use drifting lanyards` has been
chosen. If it were a very much larger array, that might be a saving of memory
for projects not choosing the option.

Note that Inform v10 only supported one form of `#Iftrue`:

	#Iftrue SKIN_TYPE_CFGV == 1;

It now also supports the inequality operators:

	#Iftrue SKIN_TYPE_CFGV < 2;
	#Iftrue SKIN_TYPE_CFGV > 1;
	#Iftrue SKIN_TYPE_CFGV <= 2;
	#Iftrue SKIN_TYPE_CFGV >= 1;
	
And similarly for `#Iffalse`. The same sort of conditional compilation trick
is not safe in the source code to kits: see below.

### Associating use options with kits

The main use of use options, though, is to define values which affect the
operation of a kit of Inter code. Almost all of the use options in the Standard
Rules or in Basic Inform are doing this, for `BasicInformKit`, `WorldModelKit`
or `CommandParserKit`. A use option which is providing configuration for a kit
should always say so, saying which kit. The syntax is exactly as above, but
ending with `in K`, where `K` is the kit name.

	Use drifting lilypads translates as the configuration flag DRIFTING_LILYPADS in AmphibianKit.
	Use horny skin translates as the configuration value SKIN_TYPE = 1 in AmphibianKit.
	Use scaly skin translates as the configuration value SKIN_TYPE = 2 in AmphibianKit.
	Use frog count of at least 10 translates as the configuration value FROG_COUNT in AmphibianKit.
	Use maximum jump height of 6 translates as the configuration value JUMP_HEIGHT in AmphibianKit.

The kit name has to be written in camel case, and end with `Kit`, following the
standard conventions for this.

When a configuration flag or value is tied to a kit, two things are different:

* The constant name is moved into that kit's namespace, so, for example,
we have `AmphibianKit\`DRIFTING_LILYPADS` not `DRIFTING_LILYPADS`.
* As a result, if two different kits both have a configuration value called,
say, `MAX_CAPACITY`, both can be used without a conflict occurring, because
one will be `FirstKit\`MAX_CAPACITY_CFGV` and the other `SecondKit\`MAX_CAPACITY_CFGV`.
* A problem message is thrown if the name is not one of those listed in the
kit's metadata as being expected.

For example, the JSON metadata file for `AmphibianKit` might read:

	{
		"is": {
			"type": "kit",
			"title": "AmphibianKit",
			"version": "5.2"
		},
		"kit-details": {
        	"configuration-flags": [ "DRIFTING_LILYPADS" ],
        	"configuration-values": [ "SKIN_TYPE", "FROG_COUNT", "JUMP_HEIGHT" ]
		}
	}

and then the above declarations would all work, but

	Use mayflies translates as the configuration flag MAYFLIES_AVAILABLE in AmphibianKit.

would be rejected because `AmphibianKit` does not support `MAYFLIES_AVAILABLE`.
Similarly, it's an error to use as a value what the kit declares as a flag
(unless the value being stored is either 0 or 1).

This raises the question of what happens if the JSON metadata asks for a
configuration flag or value which no use option talks about. The answer is
that Inform silently creates this, with the value 0, so that no link failure
can occur. For example, if we had had

	"configuration-values": [ "SKIN_TYPE", "FROG_COUNT", "JUMP_HEIGHT", "SECRET_POND" ]

then whenever Inform compiled a project with `AmphibianKit` it would define
the constant `AmphibianKit\`SECRET_POND_CFGV` to 0. Should a knowledgeable
user come along and write

	Use secret pond of 10 translates to configuration value SECRET_POND in AmphibianKit.
	Use secret pond of 7.

...this would then take effect.

Because it is not safe to use `#ifdef`, `#ifndef`, `#iftrue` or `#iffalse` on
symbol names linked in from outside of a kit, Inform automatically throws an
error on any attempt to use these directives with kit-linked configuration flags
or values. Thus, for example, in a kit:

	#ifdef AmphibianKit`DRIFTING_LILYPADS_CFGF;
	...
	#endif

would throw a problem message. Instead, the idea is to use the value, not the
existence, of these symbols in the kit:

	if (AmphibianKit`DRIFTING_LILYPADS_CFGF) {
		....
	}

Note that there's no prohibition on one kit being able to see the configuration
values of another: for example, WorldModelKit can say

	if (BasicInformKit`AMERICAN_DIALECT_CFGF) {
		....
	}

and get the right answer.

### Compiler features

Sometimes the truth is that a use option doesn't translate to anything: it's
only an instruction to the compiler to do something differently. For such
cases, a new syntax is available:

	Use engineering notation translates as a compiler feature.

There are just a few of these and they exist only for their side-effects.
See below.

### Debugging log

A new debugging log aspect, "use options", has been added:

	Include use options in the debugging log.

This shows all settings of all configuration flags and values in the project,
together with any I6 code injected by old-style use option declarations.

## Impact on existing projects

Although the existing syntax is likely to be deprecated soon, it has not been
withdrawn. Old-style use option declarations work as before, so the changes
are almost all additive.

If a project or one of its extensions explicitly refers to one of the kit
constant symbols which previously expressed whether a use option was in force,
this may not not compile. For example, the use option previously defined like this:

	Use predictable randomisation translates as (- Constant FIX_RNG; -).

is now defined like this:

	Use predictable randomisation translates as the configuration flag FIX_RNG
	in BasicInformKit.

The following table shows how all use options in the Standard Rules and Basic
Inform have changed their implementation. As noted above, the previous declarations
were somewhat misleading since the compiler was secretly causing a number of these
to fill in a special constant called `KIT_CONFIGURATION_BITMAP`: this constant
no longer exists. People really shouldn't have been using that: well, now they can't.

Use option                        | Previously set                       | Now sets
--------------------------------- | ------------------------------------ | --------------------------------
ineffectual	                      | (nothing)                            | (nothing)
index figure thumbnails           | `MAX_FIGURE_THUMBNAILS_IN_INDEX`     | (nothing)
engineering notation              | `USE_E_NOTATION`                     | (nothing)
telemetry recordings              | `TELEMETRY_ON`                       | (nothing)
no deprecated features            | `NO_DEPRECATED_FEATURES`             | `BasicInformKit\`NO_DEPRECATED_CFGF`
dynamic memory allocation         | `DynamicMemoryAllocation`            | `BasicInformKit\`STACK_FRAME_CAPACITY_CFGV`
maximum text length               | `TEXT_TY_BufferSize` (1)             | `BasicInformKit\`TEXT_BUFFER_SIZE_CFGV`
maximum things understood at once | `MATCH_LIST_WORDS` (1)               | `BasicInformKit\`MULTI_OBJ_LIST_SIZE_CFGV`
authorial modesty                 | `AUTHORIAL_MODESTY`                  | `BasicInformKit\`AUTHORIAL_MODESTY_CFGF`
numbered rules                    | `NUMBERED_RULES`                     | `BasicInformKit\`NUMBERED_RULES_CFGF`
predictable randomisation         | `FIX_RNG`                            | `BasicInformKit\`FIX_RNG_CFGF`
command line echoing              | `ECHO_COMMANDS`                      | `BasicInformKit\`ECHO_COMMANDS_CFGF`
memory economy                    | `MEMORY_ECONOMY`                     | `BasicInformKit\`MEMORY_ECONOMY_CFGF`
printed engineering notation (3)  | (didn't exist)                       | `BasicInformKit\`PRINT_ENGINEER_EXPS_CFGF`
American dialect                  | `DIALECT_US`                         | `BasicInformKit\`AMERICAN_DIALECT_CFGF`
serial comma                      | `SERIAL_COMMA` (1)                   | `WorldModelKit\`SERIAL_COMMA_CFGF`
no scoring                        | `USE_SCORING` = 0 (1)                | `WorldModelKit\`SCORING_CFGV` = 0
scoring                           | `USE_SCORING` = 1 (1)                | `WorldModelKit\`SCORING_CFGV` = 1
default route-finding (4)         | (didn't exist)                       | `WorldModelKit\`ROUTE_FINDING_CFGV` = 0
fast route-finding                | `FAST_ROUTE_FINDING`                 | `WorldModelKit\`ROUTE_FINDING_CFGV` = 1
slow route-finding                | `SLOW_ROUTE_FINDING`                 | `WorldModelKit\`ROUTE_FINDING_CFGV` = 2
full-length room descriptions     | `KIT_CONFIGURATION_LOOKMODE` = 2 (2) | `WorldModelKit\`ROOM_DESC_DETAIL_CFGV` = 2
abbreviated room descriptions     | `KIT_CONFIGURATION_LOOKMODE` = 3 (2) | `WorldModelKit\`ROOM_DESC_DETAIL_CFGV` = 3
VERBOSE room descriptions         | `DEFAULT_VERBOSE_DESCRIPTIONS`       | `WorldModelKit\`ROOM_DESC_DETAIL_CFGV` = 2
BRIEF room descriptions           | `DEFAULT_BRIEF_DESCRIPTIONS`         | `WorldModelKit\`ROOM_DESC_DETAIL_CFGV` = 1
SUPERBRIEF room descriptions      | `DEFAULT_SUPERBRIEF_DESCRIPTIONS`    | `WorldModelKit\`ROOM_DESC_DETAIL_CFGV` = 3
undo prevention                   | `PREVENT_UNDO`                       | `CommandParserKit\`UNDO_PREVENTION_CFGF`
manual pronouns                   | `MANUAL_PRONOUNS`                    | `CommandParserKit\`MANUAL_PRONOUNS_CFGF`
unabbreviated object names        | `UNABBREVIATED_OBJECT_NAMES`         | `CommandParserKit\`UNABBREVIATED_NAMES_CFGF`
gn testing version                | `GN_TESTING_VERSION`                 | (removed entirely)

(1) In these cases the original name is preserved as an alias, so that references
to, for example, `SERIAL_COMMA` will still work. But `#ifdef SERIAL_COMMA` should
still be avoided as unreliable (just as it was in v10): `if (SERIAL_COMMA)` is
the way to go.
(2) These options are duplicative, of course, and arise from twenty years of
confusion. At one time `KIT_CONFIGURATION_LOOKMODE` was called `I7_LOOKMODE`,
though not in v10.
(3) A new use option, enabling something always intended to be configurable in
`BasicInformKit`, but which for some reason hadn't been carried through in v10.
(4) A new use option, providing the default behaviour of route-finding, i.e.,
fast on Glulx and slow on the Z-machine.

It should be noted that reimplementing the standard use options in this way
caused several little-used options from v10 to begin working again: in particular,

	Use numbered rules.
	Use manual pronouns.
	Use fast route-finding.
	Use slow route-finding.

were all ineffectual in v10 due to linking errors.

## A note on conditional compilation in the built-in kits

Conditional compilation using `#ifdef` and the like is not recommended inside
kits, with the exception of testing for `DEBUG`, `TARGET_GLULX`, `TARGET_ZCODE`
and `WORDSIZE`, which are a function of the architecture being built for, and
which are therefore definitely known. The built-in kits did not follow this
rule in v10 and that led to some bugs. They now do.

For testing purposes, one exception is made. A kit can safely perform conditional
compilation on its own constants, because that doesn't involve future knowledge
when the kit is built. So, for example, the source code for the list-writer
in `WorldModelKit` contains the line:

	! Constant LKTRACE_LIST_WRITER;

This of course is commented out. Further down, code like this appears:
 
 	#Ifdef LKTRACE_LIST_WRITER; print "[There are ", no_groups, " groups.]^"; #Endif;

If the definition is un-commented-out and the kit rebuilt, this print statement
will take effect. This is all intended for "tracing", that is, for debugging
text to be printed out showing the operation of a complicated algorithm. By
convention, the built-in kits all use tracing constants with names which
begin `LKTRACE_`. (LK stands for "local to kit".)

## So has all the special-casing gone from the compiler?

Not quite. A few use options still have side-effects inside the compiler, in addition
to creating constants. For example, "no deprecated features" is declared thus:

	Use no deprecated features translates as the configuration flag NO_DEPRECATED
	in BasicInformKit.

This is all true and it causes the constant `BasicInformKit\`NO_DEPRECATED_CFGF` to
be declared as 1 not 0, but it also causes the compiler to watch out for deprecated
phrase usage, and so on. This is a complete list, at present, of use options with
compiler side-effects:

	authorial modesty
	dynamic memory allocation
	fast route-finding
	memory economy
	no deprecated features
	no scoring
	numbered rules
	scoring
	slow route-finding
	unabbreviated object names

Some of the side-effects are pretty modest: for example, the only side-effect
of `Use scoring.` and `Use no scoring.` is to change whether a problem
message is issued in response to something like `increase the score by 2`.

In addition, a few others are declared as "compiler features" alone:

	ineffectual
	engineering notation
	telemetry recordings
	index figure thumbnails

Of course, `Use ineffectual.` does nothing (it exists purely to be a default value
for the use option kind at runtime), so in a sense it has neither effects nor
side-effects.

## New problem messages

The following new problem messages (and thus test cases) have been added:

	PM_UOFlagSaysKit
	PM_UOForMissingKit
	PM_UONotInKit
	PM_UOsMutuallyExclusive
	PM_UOExplicitValueTooSmall
	PM_UOValueConflicts
	PM_UOFlagWithValue
	PM_UOValueOneWord
	PM_UOValueNotDecimal
	PM_UOValueExact
	PM_UOSymbolBad
	PM_UONumerical
	PM_UOKitNameBad
