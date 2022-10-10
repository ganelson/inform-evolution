# (IE-0006) New annotations for I6 syntax

* Proposal: [IE-0006](0006-i6-syntax-annotations.md)
* Discussion PR link: [#6](https://github.com/ganelson/inform-evolution/pull/6)
* Authors: Graham Nelson, David Kinder, and Andrew Plotkin
* Language feature name: None
* Status: Draft
* Related proposals: [IE-0013](0013-annotations-for-kit-linking.md) and
[Corresponding I6 proposal](https://github.com/DavidKinder/Inform6/issues/189).
* Implementation: Implemented but unreleased

## Summary

A standard way to mark up Inform 6 syntax to provide annotations to help tools
with linking or compilation, but which do not change its basic meaning.

## Motivation

The motivating examples for this syntax change have now been spun off into
their own proposal, [IE-0013](0013-annotations-for-kit-linking.md). What it
comes down to, though, is that the original I6 language does not have a
way to express instructions for linking (analogous to C's `extern` keyword,
or `use` in many more modern languages). The I6-reader inside of `inter`
and therefore `inform7` therefore needs to extend the I6 language with some
syntactic extensions, and it seems good to have a clean notation for that.

There are also ambitions to improve the `inform6` compiler by allowing a
system of type indications, which would also need new syntax.

It seems better to have a common syntax, so this proposal puts forward a
purely syntactic description of how "annotations" can be applied to directives,
and then read either

(a) By `inter` or `inform7` when it processes `Include (- ... -)` or compiles
kits from source, or

(b) By `inform6` itself.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No change to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No change to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None.

## Syntax

Any I6 directive read by the I6-syntax reader inside `inter` can begin with
one or more annotations introduced by the keyword `+`. As examples of the syntax:

	+replacing(from BasicInformKit) [ SquareRoot num;
		"Nobody cares about square roots, son.";
	];
	
	+private +xyzzy Constant SECRET_NUMBER = 666;

The reader removes annotations at the front of the directive, producing errors
if they are syntactically invalid, and then goes on to parse as if the
annotations were not there:

	[ SquareRoot num;
		"Nobody cares about square roots, son.";
	];
	
	Constant SECRET_NUMBER = 666;

The reader also produces errors if it sees annotations it does not understand,
but which annotations it does understand, and what effect they have, are not
the subject of this proposal: this proposal is about syntax alone.

The rules are as follows:

* An annotation can be preceded by white space, but then begins with a `+` sign.
An identifier follows which matches the regular expression `[A-Za-z_][A-Za-z0-9_]+`,
that is, it begins with a letter or underscore, then consists of letters, digits
or underscores.

* Whitespace can follow the identifier, but cannot appear between the `+` and
the identifier.

* Following this, the annotation can optionally contain "details", which must
appear inside a matched set of round brackets `(` and `)`.

* Inside the brackets is a list of "terms". Syntactically, the text in the
brackets is divided up into "tokens", divided up by whitespace. A token is a
run of non-whitespace, or some text in single quotation marks: `91`, `'my text'`
and `aquarius` are all tokens. The backslash escape `\'` can be used to write
a single quotation mark inside quoted text: `'like \'this\''`.

* Outside of quotation marks, the characters `(` and `)` must be used in a
properly nested way; if a `(` appears at the top level, it begins a new token, 
which only ends with its matching `)`.

* Outside of quotation marks and brackets, a comma `,` acts as a term divider.

* Terms are key-value pairs, but can be written in two different ways.

* If a term is a single lexical token, this is considered a value, and the
key is the placeholder `_`. For example, the annotation `+eat(brioche)`
has one term with key `_` and value `brioche`.

* If a term has two or more lexical tokens, the first token is the key name,
and the rest becomes the value. For example `+bake(temperature 220 Celsius)` has
one term with key `temperature` and value `220 Celsius`.

* In each term, the key must match the regular expression `[A-Za-z_][A-Za-z0-9_]+`.

* Identifier and key names should always be read case-sensitively: `+xyzzy`
and `+XYZZY` are different annotations.

Note that there are no maxima. A directive can have any number of annotations.
An annotation can have any number of terms.

## Examples

The following are legal:

	ANNOTATION                          TERMS IF ANY
	+example      
	+example()              
	+example(2)                         _ = 2
	+example (2)                        _ = 2
	+example(   2  )                    _ = 2
	+example(2, 4)                      _ = 2               _ = 4
	+example(2, (4, 17))                _ = 2               _ = (4, 17)
	+example(two)                       _ = two
	+example(number 2)                  number = 2
	+example(using colour)              using = colour
	+example(using colour vision)       using = colour vision
	+example(using 'colour')            using = colour
	+example(using 'k\'5\'')            using = k'5'
	+example(using 'x, )')              using = x, )
	+example(using 'x', using 'y')      using = x           using = y
	+example('using' x)                 using = x

The following are not legal:

	+                                   ! No identifier
	+(17)                               ! No identifier
	+66time                             ! Identifier begins with digit
	+bizarre<thing>                     ! Identifier has illegal characters <, >
	+example(3, (4)                     ! Mismatched brackets
	+example(3)(4)                      ! Mismatched brackets
	+example(3, )                       ! Empty term
	+example(, 4)                       ! Empty term
	+example(3, , 5)                    ! Empty term
	+example(using k'5')                ! Single quote not at start or end of token
	+example(using'colour')             ! Single quote not at start or end of token

Again, note that the rules above are purely syntactic. They say nothing about
which identifiers might be allowed in any given context, or what terms would
then be allowed for those identifiers. Syntactically, something like 
`+bake(temperature 110, temperature 180)` or `+bake(galaxy Pinwheel, food molybdenum)`
are fine. Semantically, of course, if `+bake` is allowed at all then it is very
unlikely to permit `temperature` to be repeated, or `galaxy` to be given at all,
or `food` to be set to `molybdenum`. But those are not syntax errors.
