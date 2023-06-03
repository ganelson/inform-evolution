# (IE-0019) Unicode command parser support

* Proposal: [IE-0019](0019-unicode-command-parser.md)
* Discussion PR link: [#19](https://github.com/ganelson/inform-evolution/pull/19)
* Authors: Andrew Plotkin
* Language feature name: --
* Status: Accepted in principle
* Related proposals: [IE-0005](0005-removing-translates-into-unicode.md)
* Implementation: Complete

## Summary 

Update the internal processing of the command parser to use word arrays (when
compiling to Glulx) instead of byte arrays.

## Motivation

At runtime, typed commands are parsed by code which is mostly found in
`CommandParserKit`, though this also makes use of functions in `BasicInformKit`.

The command parser reads player input into a byte array (named `buffer`).
This means that characters outside the Latin-1 character set (U+00 to U+FF)
cannot be read or parsed.

The Glk layer has functions to read and write any Unicode text, using arrays of
32-bit integers instead of bytes. Similarly, the Inform 6 compiler is able to
build a dictionary of Unicode words. If the parser were updated to use 32-bit
arrays, there would be an end-to-end path for parsing Unicode input.

The "[Unicode Parser][uniparser]" extension is an early version of this work
(for Inform 9.2 and 9.3).

[uniparser]: https://github.com/erkyrath/i7-exts/blob/master/Unicode%20Parser.i7x

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Many changes to `BasicInformKit`, `CommandParserKit`, `WorldModelKit`.
- [x] Minor changes to the Standard Rules and Basic Inform, and to the Punctuation Removal extension.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Work to be done

### Standard extensions

Pass the `$DICT_CHAR_SIZE=4` setting to the I6 compiler, indicating that the
game dictionary should use 32-bit arrays to represent dict words. (Glulx only.)

This goes in `basic_inform/Sections/Preamble.w`, but I think it will require an
`allow_DICT_CHAR_SIZE` tweak in `Target Virtual Machines.w` and `Use Options.w`.
(Following the pattern of `allow_MAX_LOCAL_VARIABLES`.)

We will need to set the I6 constants `DICT_WORD_SIZE` (default 9) and
`DICT_ENTRY_BYTES` (computed from `DICT_WORD_SIZE`, default 48). In the I6
compiler they are auto-created symbols, derived from the `$DICT_WORD_SIZE`
compiler setting. Currently I7 has no way to change this setting, so we can
hardwire them in `Definitions.i6t`.

(Adjusting `DICT_WORD_SIZE` means new computed values for `DICT_ENTRY_BYTES`,
`#dict_par1`, and `#dict_par2`.)

### Kits

Glulx.i6t: Change the `buffer` array (also `buffer2` and `buffer3`) to 32-bit (`-->`) arrays.

Glulx.i6t: Update the `VM_ReadKeyboard()` function to call `glk_request_line_event_uni()` instead of `glk_request_line_event()`.

Other changes to Glulx.i6t: `VM_CopyBuffer()`, `VM_PrintToBuffer()`, `VM_Tokenise()`, `LTI_Insert()`, `GGWordCompare()`.

Note that outside of Glulx.i6t, these changes will rely on `#ifdef TARGET_ZCODE/TARGET_GLULX` to do the correct array access (`buffer->x` vs `gbuffer-->x`). This will get messy.

Language.i6t: `LanguageContraction()`

Parser.i6t: `PrintSnippet()`, `SpliceSnippet()`, `Keyboard()`, `Parser__parse()`, `NounDomain()`, `SetPlayersCommand()`, `TEXT_TY_ROGPRI()`, `TryNumber()`.

Tokens.i6t: `DECIMAL_TOKEN()`, `FLOAT_TOKEN()`.

Time.i6t: `TIME_TOKEN()`.

Actions.i6t: `DA_Topic()`.

Tests.i6t: `TestKeyboardPrimitive()`.

Mathematics.i6t: `FloatParse()`.

Printing.i6t: `CPrintOrRun()`.

### Compiler

To completely support Unicode input in Inform, `inform7` will have to be updated
to correctly handle `Understand` declarations with Unicode text:

```
Understand "βράχος" as the rock.
Understand "παίρνω [things]" as taking.
```

Currently this erroneously understands each Unicode character as a grammar token,
and generates errors of the form:

> The grammar token 'unicode 946' in the sentence 'Understand "[unicode 946][unicode 961][unicode [...] ode 967][unicode 959][unicode 962]" as the rock' looked to me as if it might be a unicode character, but this isn't something allowed in parsing grammar.

The simplest way to handle this is to write the characters directly out to the `auto.inf` source file (in UTF-8 encoding), and then pass the `-cU` flag to the I6 compiler.

```
Verb 'παίρνω' * multi -> Take;
```

I6 Unicode escapes would also work (without needing the `-cU` flag):

```
Verb '@{3C0}@{3B1}@{3AF}@{3C1}@{3BD}@{3C9}' * multi -> Take;
```

## Rejected alternatives

The messy `#ifdef TARGET_ZCODE/TARGET_GLULX` array access could in theory be tidied by adding a new I6 operator or assembly macro. Perhaps `:->`, acting like `->` in Z-code, `-->` in Glulx. But this seems like vastly more trouble than it's worth.

Some `#ifdef`s could be avoided by defining a `DICT_CHAR_SIZE` constant, which is 1 in Z-code, 4 in Glulx. The `#ifdef`s do the job, though.
