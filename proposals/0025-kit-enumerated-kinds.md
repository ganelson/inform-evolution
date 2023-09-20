# (IE-0025) Kit-enumerated kinds

* Proposal: [IE-0025](0025-kit-enumerated-kinds.md)
* Discussion PR link: [#25](https://github.com/ganelson/inform-evolution/pull/25)
* Authors: Graham Nelson
* Status: Accepted
* Related proposals: [IE-0033](0033-kit-set-properties.md)
* Implementation: Implemented but unreleased

## Summary

Extends the Neptune mini-language, by which kits can add new kinds of value to the
Inform language, to allow enumerations to be created whose run-time values are
not necessarily 1, 2, 3, ..., and no longer need to be contiguous.

## Motivation

For example, the Glk interface layer has numerous enumerated values defining
its functions. At present there's no tidy way to create a valid Inform kind,
`Glk function`, say, which enumerates these: and there is no non-verbose way
to associated Inter identifier names with them.

This proposal begins from the point of view that such an enumeration should
be defined by a kit, and exported to Inform source text, rather than via versa.
This is done within the Neptune language for declaring new kinds.

## Components affected

- [ ] No change to the natural-language syntax.
- [x] Minor change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [ ] No changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

None.

## Details

See `inform/inform7/Tests/Extension/Araminta Intest/Directorial Testing-v2_7.i7xd`
in the main repository for a working example, and in particular see its `TestKit`.
This kit has the JSON metadata:

	{
		"is": {
			"type": "kit",
			"title": "TestKit"
		},
		"kit-details": {
			"provides-kinds": [ "Testy.neptune" ]
		}
	}

This simply says that it provides a Neptune file to declare one or more kinds.
Inside the kit's `kinds` directory, the following is `Testy.neptune`:

	new base COLOUR_TY {
		conforms-to: ENUMERATED_VALUE_TY
		singular: colour
		plural: colours

		instance: red                  = RED_COL    =    7
		instance: purple               = PURPLE_COL =   31
		instance: chartreusey lavender = MAUVE_COL  =  101
	}

At least one instance must be provided, and they must be given in strictly
increasing value order. Values are actually optional. This alternative:

		instance: green = GREEN_COL
		instance: taupe = TAUPE_COL

would create `green` and `taupe` with values 1 and 2. This:

		instance: cyan = CYAN_COL = 10
		instance: blue = BLUE_COL
		instance: grey = GREY_COL = 20
		instance: pink = PINK_COL

creates the values as 10, 11, 20, 21.

Values can alternatively be given in Inform 6 hexadecimal or binary notation:

		instance: cerise = CERISE_COL = $1f0
		instance: hue mask = HUE_MASK = $$1110001

### Referring to enumerated values in Inform source text

The effect is that if Inform compiles a project using this extension, then
it includes this kit, which exports this kind of value. So, for example,

	Include Directorial Testing by Araminta Intest.

	Laboratory is a room. East of the Laboratory is the Boudoir.

	A room has a colour called colour scheme.
	
	The colour scheme of the Boudoir is chartreusey lavender.
	
	After looking: say "The wallpaper has a somewhat [colour scheme of the location] tint."

Kinds like this behave like other enumerations. So, for example,

	the first value of colour
	the last value of colour
	the colour after red
	the colour before purple
	a random colour
	a random colour from red to purple
	the list of colours

all work as expected. Enumerated values like these can be said and understood
by the command parser.

### New phrases

Two new phrases have been added to Basic Inform:

	numerical value of (V - enumerated value)
	sequence number of (V - enumerated value)

so that, for example, `numerical value of purple` is 31, while the
`sequence number of purple` is 2.

These phrases work on regular enumerations, too: given

	Material is a kind of value. Cotton and satin are materials.
	
then `numerical value of cotton` evaluates to 1, and `numerical value of satin` to 2.
For regular enumerations, the sequence number is the same as the numerical value.

Note that a kit-provided enumeration matches `enumerated value` in the Inform
type-checker, just as a regular one does. So for example this is the definition
of `numerical value of ...` in Basic Inform:

	To decide what number is the numerical value of (V - enumerated value): (- {V} -).

### Referring to enumerated values from kit code

Code in the kit can use identifiers such as `RED_COL` or `PURPLE_COL` without
needing to define them using `Constant ...` declarations: they are already
provided by the linker. So for example, the kit could include the function:

	[ TestFunction;
		print "The red is ", RED_COL, "^";
		print "The mauve is ", MAUVE_COL, "^";
	];

Code in inclusions from natural language can use either the natural language
form or the identifier:

	To test out bracket-plus: (-
		print MAUVE_COL, " = ", (+ chartreusey lavender +), "^";
	-).

If executed, this will print `101 = 101`.

### Restriction on properties

The one significant restriction on kinds of this form is that (unless the enumeration
happens to work out to have the runtime values 1, 2, 3, ...) properties cannot be
created for such a kind, so that

	A colour has a number called luminosity.

...throws a problem message.
