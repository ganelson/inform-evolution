# (IE-0026) Units and number bases

* Proposal: [IE-0026](0026-units-and-number-bases.md)
* Discussion PR link: [#26](https://github.com/ganelson/inform-evolution/pull/26)
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: --
* Implementation: In progress

## Summary

To make notations for units much more expressive and varied, and to make
it possible to use unusual number bases and digits.

## Motivation

Notations for units have not really increased in flexibility since around 2010,
but could be more expressive. For example, it is not currently possible to
write the standard CSS notation for colours in Inform, or the standard
notation for squares on a chessboard.

## Components affected

- [x] Minor change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor additions to BasicInformKit.
- [x] Minor additions to Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

For the most part, these changes are additive; nothing in the test suite was
troubled. But:

* As with any new phrases, the phrases here for writing numbers in different
number bases might have names clashing with user-defined phrases.
* Similarly, anyone with a room (say) named `Binary 101` might have to
rephrase.
* Authors who define notations including angle brackets will now have to escape
those, which will mean a small but easy one-line change.
* A bug in Inform 10 caused (decimal) literal numbers with leading zeros to be
valid as literal values of `number`: thus `00001` was read as `1`. This bug has
been fixed, but it is halfway conceivable that somebody may have relied on the
old behaviour. If so, they'll find a problem message and an easy fix.

## Printing numbers in different bases

Basic Inform now provides `say` phrases for printing in any number base from
2 to 36, using the upper-case letters A to Z for the digits 10 to 35. In
addition, they can optionally pad this number out to any number of digits
with leading 0s. Thus:

	"[N in hexadecimal]"
	"[N in decimal]"
	"[N in unsigned decimal]"
	"[N in octal]"
	"[N in binary]"
	"[N in base B]"
	"[N in M digit/digits]"
	"[N in M hexadecimal digit/digits]"
	"[N in M decimal digit/digits]"
	"[N in M octal digit/digits]"
	"[N in M binary digit/digits]"
	"[N in M base B digit/digits]"

Only base 10 numbers are printed as signed, and only when no digit count is
required. `"[N in decimal]"` and `"[N in base 10]"` are equivalent to `"[N]"`
and print its value as a signed decimal number.

Thus, -1 prints as FFFFFFFF in hex, -1 in decimal, 4294967285 in unsigned decimal,
37777777777 in octal, 11111111111111111111111111111111 in binary, 102002022201221111200
in base 3 and 1Z141Y3 in base 36.

## Literal cardinal numbers in different bases

At any point in the Inform syntax where a general literal value can be used,
it was always of course possible to write a (signed) decimal literal.

It is now also possible to write binary, octal or hexadecimal literals. These
are unsigned, are permitted to have leading zeros (which do not affect their
value), and must be prefixed by the base name. For example:

	hexadecimal 4EF12C
	binary 001110101101
	octal 077

There seems no need for `decimal ...`, nor for other bases.

For example, the output of this Basic Inform program:

	To begin:
		let Q be hexadecimal 4EF12C;
		showme Q;
		say "You notice that [Q] is the same number as hexadecimal [Q in hexadecimal].";
		showme binary 110110;
		showme "[binary 110110 in binary]";
		showme hexadecimal A4;
		showme "[hexadecimal A4 in hexadecimal]";
		showme octal 177;
		showme "[octal 177 in octal]";

...is:

	"Q" = number: 5173548
	You notice that 5173548 is the same number as hexadecimal 4EF12C.
	number: 54
	text: 110110
	number: 164
	text: A4
	number: 127
	text: 177

## Units with number bases

A unit is created by making a new kind and then supplying a notation for it. For
example:

	Time signature is a kind of value. 1/99 time specifies a time signature.

This allows the author to write `3/4 time` as a constant value of the kind
`time signature`, for example. The "specification" for the notation, `10/99 time`,
is very compact for what it expresses, but it does have limitations. This says
that we have a number, a slash, then a number from 0 to 99, and the word `time`.
By default, though, it prints back as `3/04 time`. To get around this sort of
thing, Inform already has a way to give the different numerical "parts"
non-standard behaviour. A better definition would be:

	Time signature is a kind of value. 1/99 time specifies a time signature
	with parts beats and bar length (without leading zeros).

This was all true in Inform v10, but now we move to new possibilities.

### Changing the number base

Suppose we want to use a number base other than 10? This is now possible:

	CSS colour is a kind of value. #FF_FF_FF specifies a CSS colour in hexadecimal.

The base can be `in binary`, `in octal`, `in decimal`, `in hexadecimal` or
`in base B` for any value of `B` from 2 to 36. Note that when looking for
"parts" in the specification, Inform then looks for digits of that number
base, which is why it sees three parts here: `FF`, `FF` and `FF` being
hex numbers.

### Options available for each part

It is also possible to specify a number base for each individual part, and
in general the range of options which can be specified for a part is now:

	optional (*)
	preamble optional (*)
	with leading zeros
	without leading zeros (*)
	in BASE
	N to M
	up to D BASE digit/digits
	up to D digit/digits
	D BASE digit/digits
	D digit/digits
	digits "TEXT"
	values "TEXT"
	corresponding to KIND

Those marked `(*)` existed in Inform 10: the rest are new. A few notes:

* `with leading zeros` and `without leading zeros` override the default
for a part, which is to print without on the first part and with on all
subsequent ones.
* `N to M` and the digit counts override the default range for a part.
(By default, this is unlimited for the first part and 0 to whatever number
is used to write the part, for the subsequent ones.) At present, `N` is
required to be non-negative, and `M` is required to be greater than `N`.
* An alternative way to specify the range is to write, say, `2 digits`,
which would mean `0 to 99` if the number base for the kind is decimal.
* If a part uses both, e.g., `0 to 39, 2 digits`, then the explicit range,
in this case 0 to 39, wins out. An impossible combination such as
`0 to 199, 2 digits` is rejected with a problem message.
* Giving the range as, say, `2 digits` implies `with leading zeros`, and
giving it as `up to 2 digits` implies `without leading zeros`.
* `BASE` can, again, be any of `in binary`, `in octal`, `in decimal`,
`in hexadecimal` or `in base B` from 2 to 36.
* When parsing this notation, digits 10 to 35 are recognised as either
`a` to `z` or `A` to `Z`, but print back as `A` to `Z`.
* If a number of digits is specified, then this affects parsing as well
as printing. Thus `0D` matches `2 hexadecimal digits` but `D` does not,
and nor does `D38`. `D` does however match `up to 2 hexadecimal digits`.
* For `digits ...` and `values ...`, see below.

### Nonstandard digit sets

The option `digits "TEXT"` tells Inform to use the supplied digit characters
in place of the regular ones. The text must contain exactly the number of
characters which equals the number base (8 for octal, and so on), must have
no repeats, and must not use spaces or square brackets.

For binary, the default is `digits "01"`, for decimal it is `digits "0123456789"`,
for hexadecimal `digits "0123456789ABCDEF"` and so on.

Note that if non-standard digit sets are used, then numbers must be written
as unsigned, and are printed accordingly.

A dull use of this feature would be to print hex with lower-case letters on
the digit values 10 to 15, rather than Inform's customary upper-case: this
could be achieved with `digits "0123456789abcdef"`.

More interestingly, this Basic Inform program:

	A Thai number is a kind of value.
	<value> specifies a Thai number with parts value (digits "๐๑๒๓๔๕๖๗๘๙").

	To say (N - a number) in Thai digits:
		if N is negative:
			say "ลบ";
			if N is -2147483648:
				say the Thai number with value part 214748364;
				say the Thai number with value part 8;
			otherwise:
				say the Thai number with value part (0 minus N);
		otherwise:
			say the Thai number with value part N.

	To begin:
		say "-17 = [-17 in Thai digits].";
		say "-2147483648 = [-2147483648 in Thai digits].";
		say "-2147483648 = [-2147483648 in unsigned Thai digits].";
		say "90125 = [90125 in Thai digits]."

prints:

	-17 = ลบ๑๗.
	-2147483648 = ลบ๒๑๔๗๔๘๓๖๔๘.
	90125 = ๙๐๑๒๕.

More ludically:

	A tomb wall pattern is a kind of value.
	<pattern number> specifies a tomb wall pattern with parts pattern number
	(4 base 4 digits, digits "▙▛▜▟").

	To begin:
		repeat with S running from ▙▙▙▙ to ▟▟▟▟:
			say "[S] ";
		say ".... is the full set."

prints all 256 possible wall carvings using a row of 4 shapes each of which
can be of 4 types.

### Use of escaped part names

While admirably compact, the current notation for notations makes it
impossible to express either literal characters which happen to be digits,
because they will be mistaken for parts, or two consecutive parts which
have no characters in between, because they will be mistaken for one big part.

There is therefore now an angle-bracketed escape notation within notations.
Angle brackets must either contain a double-quoted string to be taken
literally, or else the name of one of the parts. For example:

	An agent is a kind of value.
	<"00">7 specifies an agent.

allows constants like `007` and `0011` to be used and printed back, with
the double-0 prefix having no numerical meaning but signalling that this is
an agent, not a number.

This makes it impossible to have a literal `"` character, but those are
impossible for Inform to read in literals anyway, so this is no loss. But
literal angle brackets can be written, so that

	<"<">99<">">

is the new way to write the old notation

	<99>

As an example of named parts:

	CSS colour is a kind of value.
	#<red level><green level><blue level> specifies a CSS colour with parts
		red level (2 hexadecimal digits),
		green level (2 hexadecimal digits) and
		blue level (2 hexadecimal digits).

This enables constants like `#4169E1`, royal blue, to be written, understood
and parsed back.

The part names in the angle-bracket escapes have to exactly match those given
in the tail of the sentence, and in the same order.

## Explicit sets of values

In modern chess, the squares are written a1 to h8, with the letter representing
the file (thus, the a-pawn is the one beginning on a2) and the number the rank
(thus, Black's king spends the early game hiding on the 8th rank). This used
to be called "algebraic notation", though no actual algebra is done with it.

How to achieve this, making an Inform kind whose values are exactly the 64
squares `a1` to `h8`? A devious option would be:

	A chessboard square is a kind of value.
	<file><rank> specifies a chessboard square with parts
		file (1 octal digit, digits "abcdefgh") and
		rank (1 to 8).

But this is not ideal. We would find that `file part of a1` was 0, whereas
`rank part of a1` was 1: it seems clumsy to number files from 0 and ranks from 1.
The following would fix that, but hardly seems elegant:

	A chessboard square is a kind of value.
	<file><rank> specifies a chessboard square with parts
		file (1 to 8, digits "0abcdefgh9") and
		rank (1 to 8).

where, of course, the digits `0` and `9` can never be needed for the file,
because its range is constrained as `1 to 8`.
	
This is tidier:

	A chessboard square is a kind of value.
	<file><rank> specifies a chessboard square with parts
		file (values "a, b, c, d, e, f, g, h") and
		rank (1 to 8).

The file part is automatically constrained as 1 to 8 because there are 8
notations in the text `"a, b, c, d, e, f, g, h"`: this is a comma-separated
list in which each individual term must not include spaces or square brackets,
or of course commas. There must be at least two values provided, and they
do not need to be only one character long, even though in this example they are.

If explicit values are given, the various options to do with digits, leading
zeros and ranges are all forbidden, since the value set defines the range of
values exactly.

The following little game works with any of the three definitions above:

	Capablanca's Dining Room is a room.

	When play begins:
		showme c4;
		showme d7;
		repeat with S running from a1 to h8:
			say "[S] ";
		say "...and that's all."

	Square pressing is an action applying to one chessboard square.

	Carry out square pressing: say "You press square [chessboard square understood]."

	Understand "press [chessboard square]" as square pressing.

	Test chess with "press a1 / press a0 / press h8 / press c3 / press i3".

Which all works as expected:

	chessboard square: c4
	chessboard square: d7
	a1 a2 a3 a4 a5 a6 a7 a8 b1 b2 b3 b4 b5 b6 b7 b8 c1 c2 c3 c4 c5 c6 c7 c8
	d1 d2 d3 d4 d5 d6 d7 d8 e1 e2 e3 e4 e5 e6 e7 e8 f1 f2 f3 f4 f5 f6 f7 f8
	g1 g2 g3 g4 g5 g6 g7 g8 h1 h2 h3 h4 h5 h6 h7 h8 ...and that's all.

	Capablanca's Dining Room
	> press a1
	You press square a1.

	> press a0
	You can't see any such thing.

	> press h8
	You press square h8.

	> press c3
	You press square c3.

	> press i3
	You can't see any such thing.

The descriptive English notation would also be possible:

	A chessboard square is a kind of value.
	<file><rank> specifies a chessboard square with parts
		file (values "QR, QN, QB, Q, K, KB, KN, KR") and
		rank (1 to 8).

But since a quaint feature of descriptive notation is that the square which
White calls KB2 is the square Black calls KB7, and we cannot know who is
asking, we had probably better avoid that.

### Corresponding to kinds

The two parts of a `chessboard square`, as defined above, are each numbers from
1 to 8. There's much to be said for that, but suppose we want them to
correspond to something other than a number. Here is yet another definition:

	A chessboard file is a kind of value. The chessboard files are
	the a-file, the b-file, the c-file, the d-file, the e-file, the f-file,
	the g-file and the h-file.

	A chessboard rank is a kind of value. The chessboard ranks are
	the first rank, the second rank, the third rank, the fourth rank, 
	the fifth rank, the sixth rank, the seventh rank, the eighth rank.

	A chessboard square is a kind of value.
	<file><rank> specifies a chessboard square with parts
		file (values "a, b, c, d, e, f, g, h", corresponding to chessboard files) and
		rank (1 to 8, corresponding to chessboard ranks).

We now find that, for example:

	"chessboard square with file part the d-file rank part the sixth rank" = chessboard square: d6
	"rank part of e7" = chessboard rank: seventh rank

What `corresponding to K`, where `K` is a kind, does is to tell Inform that
the numerical value internally stored to represent the meaning of a part should
be treated as a value of kind `K`. By default, `K` is `number`.

`K` can be set to any numerical kind, or to any enumeration with the right number
of instances. `chessboard rank` above has 8 instances, which matches the range
of 8 possible values for `rank`: if these numbers had not matched, a problem
message would have been issued.

If `K` uses floating-point numbers, then the integer is automatically floated
when `X part of V` is executed, and similarly `V with X part W` will automatically
round the floating-point number `W` to the nearest integer and supply that.

### A draft example: Aircraft tail numbers

Since 1956, pilots have identified their aircraft over radio channels using
a standard international alphabet, which has several names and numerous parent
organisations. It goes like so:

	The international aviation alphabet is a kind of value. Alpha, Bravo,
	Charlie, Delta, Echo, Foxtrot, Golf, Hotel, India, Juliet, Kilo, Lima, Mike,
	November, Oscar, Papa, Quebec, Romeo, Sierra, Tango, Uniform, Victor,
	Whiskey, X-ray, Yankee, Zulu is the international aviation alphabet.

(Some people spell `Alfa` and `Juliett`. I do not.) European civil aircraft
are normally given "tail numbers" like this:

	A tail number is a kind of value.
	Aircraft <country>-<first><second><third><fourth> specifies a tail number
	with parts
		country (values "A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z",
			corresponding to the international aviation alphabet),
		first (values "A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z",
			corresponding to the international aviation alphabet),
		second (values "A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z",
			corresponding to the international aviation alphabet),
		third (values "A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z",
			corresponding to the international aviation alphabet), and
		fourth (values "A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z",
			corresponding to the international aviation alphabet).

This is all a little clumsy and repetitious, but no matter, because now we
can write assertions like:

	The Mont Blanc rescue helicopter is always F-ZBQG.

or text like:

	"Hello Shanwick, this is [callsign of G-ERTI]."

which prints out as:

	Hello Shanwick, this is Golf Echo Romeo Tango India.

### A draft example: Harvard star classifications

In 1866 the Italian priest Angelo Secchi, having determined for the first time
that the Sun was indeed a star, found a way to sort stars by their spectra,
that is, by the blend of colours of light with which they shone. He used:

	A stellar type is a kind of value.
	Secchi class <class> specifies a stellar type with parts
		class (values "I, II, III, IV, V, VI, VII").

At Harvard, Henry Draper and, more particularly, Edward Pickering carried out
a larger survey and subdivided Secchi's classes so that there were now letters 
`A` to `Q`, with `A` to `D` being versions of `Secchi class I`, and so on.
Exasperatingly, they omitted `J` but included `O`. Antonia Maury, the first
woman to be credited as an astronomical observer, decided that `A` and `B`
were the wrong way round, and moved back to Roman numbers, `I` to `XXII`.
Annie Jump Cannon brokered a compromise between Maury and Pickering. She reverted
to letters, but ordered them better by what was now understood to be surface
temperature, and grouped them simply as `O` (hottest), `B`, `A`, `F`, `G`, `K`, `M`
(coolest): the traditional mnemonic being "oh be a fine girl, kiss me".
Each class was subdivided 0 to 9, so that `G2` was closer to `G` than `K`,
while `G9` was very nearly K-type.

In 1943 the action moved to Wisconsin, where the University of Chicago's
Yerkes Observatory found a way to sort stars not by temperature but by
luminosity: these classes were `0` (brightest), `Ia`, `Iab`, `Ib`, `II`, `III`,
`IV`, `V`, `VI`, `VII` (dimmest).

Modern stellar classification is a combination:

	A temperature class is a kind of value. The temperature classes are O-type,
	B-type, A-type, F-type, G-type, K-type and M-type.

	A luminosity class is a kind of value. The luminosity classes are hypergiant,
	luminous supergiant, intermediate-size luminous supergiant, less luminous
	supergiant, bright giant, giant, subgiant, main-sequence, subdwarf and white
	dwarf.

	A stellar type is a kind of value. <class><temperature><luminosity> specifies a
	stellar type with parts
		class (values "O, B, A, F, G, K, M", corresponding to temperature classes),
		temperature (1 decimal digit, corresponding to a real number) and
		luminosity (values "0, Ia, Iab, Ib, II, III, IV, V, VI, VII", corresponding
			to luminosity classes).

In fact this is still a simplification because people sometimes subdivide
the number further or add "peculiarity" suffixes like `He wk`, or add extreme
additional types for truly odd stars, but it'll do.

Between 1910 and 1940 Annie Jump Cannon was to classify 350,000 stars by hand,
reaching a speed of three spectra per minute, a feat unequalled before or since.
Here's a smaller survey:

	A star is a kind of thing. A star has a stellar type. Some stars are defined
	by the Table of Astral Survey Work.

	Table of Astral Survey Work
	name			stellar type
	S Monocerotis		O7V
	Rigel			B8Ia
	Deneb			A2Ia
	Zeta Leonis		F0III
	Beta Aquilae		G8IV
	Epsilon Eridani		K2V
	Herschel's Garnet	M2Ia

Which we can now try out:

	To classify (star in question - a star):
		say "Annie Jump Cannon classifies [star in question] as [stellar type
		of the star in question]. 'Hm, yes - [luminosity part of stellar
		type of the star in question].'"

	When play begins:
		showme O00;
		showme O0Ia;
		showme K9Iab;
		showme the class part of K9Iab;
		showme the temperature part of K9Iab;
		showme the luminosity part of K9Iab;
		showme the stellar type with class part A-type temperature part 4.0 luminosity part subgiant;
		classify Beta Aquilae;
		classify Herschel's Garnet.

This prints:

	stellar type: O00
	stellar type: O0Ia
	stellar type: K9Iab
	"class part of K9Iab" = temperature class: K-type
	"temperature part of K9Iab" = real number: 9.0
	"luminosity part of K9Iab" = luminosity class: intermediate-size luminous supergiant
	"stellar type with class part A-type temperature part 4.0 luminosity part subgiant" = stellar type: A4IV
	Annie Jump Cannon classifies Beta Aquilae as G8IV. "Hm, yes - subgiant."
	Annie Jump Cannon classifies Herschel's Garnet as M2Ia. "Hm, yes - luminous supergiant."

And stellar types, like other notations, can also be understood:

	Observing is an action applying to one stellar type.

	Carry out observing: say "You observe a star of type [stellar type understood]."

	Understand "observe [stellar type]" as observing.

	Test me with "observe K9Iab / observe O00 / observe M9VII / observe F2Ia".

which plays as follows:

	> observe k9iab
	You observe a star of type K9Iab.

	> observe o00
	You observe a star of type O00.

	> observe m9vii
	You observe a star of type M9VII.

	> observe f2ia
	You observe a star of type F2Ia.

### A draft example: Ordnance Survey grid references

The Ordnance Survey National Grid reference system (OSGB) was developed in 1936
for the great retriangulation, which led to the building of many concrete pillars
on British hilltops and to today's maps. The full range of values is enormous,
since OSGB can specify any point in the British Isles to an accuracy of 1m.
Fortunately, we only want to use positions in the town of Oxford, which sits
fully inside a modest rectangular portion of square `SP`. So:

	Length is a kind of value. 1.0m specifies a length. 1.0km specifies a length
	scaled up by 1000. Area is a kind of value. 1.0 sq m specifies an area. A length
	times a length specifies an area.

	Grid reference is a kind of value. SP <easting> <northing> specifies a grid
	reference with parts
		easting (45000 to 54999, corresponding to lengths) and
		northing (5 digits, 3000 to 8999, corresponding to lengths).

	A room has a grid reference called map position.

	Radcliffe Camera is a room with map position SP 51495 06392.
	Queen's Lane Coffee House is a room with map position SP 51735 06319.
	Iffley Lock is a room with map position SP 52460 03628.
	Wytham Woods is a room with map position SP 46227 08076.
	St Hugh's College is a room with map position SP 50825 07750.

In a reference like `SP 51495 06392`, the two five-digit numbers are called
"easting" and "northing", and are in effect x- and y-coordinates measured in
meters. (Note that these two parts of the notation are `corresponding to lengths`,
which uses real arithmetic, but the conversions are all automatic.) So we
can perform some arithmetic:

	To decide what length is the grid distance from (GR1 - grid reference) to (GR2 - grid reference):
		let 𝚫n be the northing part of GR1 minus the northing part of GR2;
		let 𝚫e be the easting part of GR1 minus the easting part of GR2;
		let D be the real square root of ((𝚫e times 𝚫e) plus (𝚫n times 𝚫n));
		decide on D.

	To decide what length is the distance as the crow flies from (R1 - room) to (R2 - room):
		let GR1 be the map position of R1;
		let GR2 be the map position of R2;
		decide on the grid distance from GR1 to GR2.

And now, say:

	let GR be SP 51014 07322;
	showme GR;
	showme the northing part of GR;
	showme the easting part of GR;
	showme the distance as the crow flies from the Radcliffe Camera to Queen's Lane Coffee House;
	showme the distance as the crow flies from the Radcliffe Camera to Iffley Lock;

prints:

	"GR" = grid reference: SP 51014 07322
	"northing part of GR" = length: 7.322km
	"easting part of GR" = length: 51.014km
	"distance as the crow flies from the Radcliffe Camera to Queen's Lane Coffee House" = length: 250.85654m
	"distance as the crow flies from the Radcliffe Camera to Iffley Lock" = length: 2.92761km

This accuracy is all a little spurious due to the curvature of the earth and the
difficulty of deciding which table at the Queen's Lane Coffee House defines its
position, and so on, but it's right to within, let's say, 5m.

	Visiting is an action applying to one grid reference.

	Check visiting:
		let X be the grid reference understood;
		let D be the grid distance from the map position of the location of the player to X;
		if D < 5m, say "You're there already, near enough." instead.

	Carry out visiting:
		let X be the grid reference understood;
		let D be the grid distance from the map position of the location of the player to X;
		say "You walk [D] to [X]";
		let the closest room be a room;
		let the closest approach be -1m;
		repeat with R running through rooms which are not the location of the actor:
			let D be the grid distance from the X to the map position of R;
			if the closest approach < 0m or the closest approach > D:
				now the closest approach is D;
				now the closest room is R;
		if the closest approach < 5m:
			say ", which takes you neatly to...";
		otherwise:
			say ", but there's nothing much there, so you sidle a further [closest approach] over to [the closest room] instead, at [map position of the closest room].";
		now the actor is in the closest room.

	Understand "visit [grid reference]" as visiting.

	Test me with "visit SP 51014 07322 / visit SP 51735 06319 / visit SP 51735 06319".

Which produces:

	> visit sp 51014 07322
	You walk 7.40594km to SP 51014 07322, but there's nothing much there, so you
	sidle a further 467.87283m over to St Hugh's College instead, at SP 50825 07750.

	St Hugh's College

	> visit sp 51735 06319
	You walk 1.69584km to SP 51735 06319, which takes you neatly to...

	Queen's Lane Coffee House

	> visit sp 51735 06319
	You're there already, near enough.
