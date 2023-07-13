# (IE-0026) Units and number bases

* Proposal: [IE-0026](0026-units-and-number-bases.md)
* Discussion PR link: [#26](https://github.com/ganelson/inform-evolution/pull/26)
* Authors: Graham Nelson
* Language feature name: --
* Status: Draft
* Related proposals: --
* Implementation: In progress

## Summary

To make it easier to deal with binary and hexadecimal numbers, and to permit
notations for units which employ them.

## Motivation

Notations for units have not really increased in flexibility since around 2010,
but could be more expressive. For example, it is not currently possible to
write the standard CSS notation for colours in Inform.

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

### Recap

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

### Changing the number base

But suppose we want to use a number base other than 10? This is now possible:

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

Those marked `(*)` existed in Inform 10: the rest are new. A few notes:

* `with leading zeros` and `without leading zeros` override the default
for a part, which is to print without on the first part and with on all
subsequent ones.
* `N to M` and the digit counts override the default range for a part.
(By default, this is unlimited for the first part and 0 to whatever number
is used to write the part, for the subsequent ones.) At present, `N` is
required to be `0`, and `M` is required to be at least 1.
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
	red level (2 hexadecimal digits), green level (2 hexadecimal digits)
	and blue level (2 hexadecimal digits).

This enables constants like `#4169E1`, royal blue, to be written, understood
and parsed back.

The part names in the angle-bracket escapes have to exactly match those given
in the tail of the sentence, and in the same order.
