# (IE-0011) Additional data structures

* Proposal: IE-0011
* Reference link: [#11](https://github.com/ganelson/inform-evolution/pull/11)
* Authors: Dannii Willis
* Language feature name: None
* Status: Draft
* Related proposals: None
* Implementation: None

## Summary

Add some new data structure kinds into `BasicInformKit`.

## Motivation

Inform has some data structures already, notably the List and the Table, but there are some others that would be very useful to authors for various purposes. These Data Structures were first prototyped in the extension [Data Structures by Dannii Willis](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Data%20Structures.i7x).

It is proposed that these kinds be brought into the core of Inform for these reasons:

1. They are of general usefulness, and installing an extension/kit like this can be complicated. (Though hopefully much less so once [IE-0001](https://github.com/ganelson/inform-evolution/pull/1) is implemented.)
2. At least in 6M62, it was not possible to specify how a new kind was to have its `constant-compilation-method:special` be implemented. This meant that most of these new kinds needed to be constructed with long blocks, when they could possibly be more efficient as short-block-only kinds.
3. A function like `ANY_TY_Print_Kind_Name` can be constructed by the compiler rather than manually written.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] Major changes to inbuild.
- [x] Change to inform7.
- [?] No change to inter.
- [ ] No change to the Inter specification.
- [x] Change to runtime kits.
- [?] No changes to the Standard Rules and Basic Inform.
- [x] Minor changes to documentation.
- [ ] No change to the GUI apps, when downloading or installing extensions.

## Impact on existing projects

None.

## Proposal

The following new kinds be added to Inform in the `BasicInformKit`:

### Anys

An any stores a value and its kind; the kind cannot be determined at compile time, but can be read at run time. These are useful for when you want to store multiple kinds of values in one list or map, or for when you don't know what kind some data might be.

```
When play begins:
  let apple be "Royal Gala" as an any;
  if kind of apple is a text:
    say "[apple] is a text[line break]";
  if apple is a text let apple name be the value:
    say "Apple variety: [apple name][line break]";
  let year be apple as a number or 2022;
```

### Couples/Tuples

A couple is a 2-tuple, grouping two values of any kind. Couples are useful for when you need to return two values of different kinds from a phrase.

```
To decide what couple of person and number is the person evaluation:
  decide on yourself and 1234 as a couple;

When play begins:
  let result be the person evaluation;
  say "Person: [first value of result][line break]Evaluation: [second value of result][line break]";
```

I couldn't work out how to implement a generic N-tuple type, but maybe it could be if incorporate into the core. Also, there does exist a hidden `TUPLE_ENTRY_TY` which I don't know anything about.

### Maps

Maps store key-value pairs. Each map has a set kind for its keys and another set kind for its values, but if you need to store heterogenous keys or values you can make a map using anys.

```
When play begins:
  let data be a map of text to any;
  set key "player" of data to yourself;
  set key "score" of data to 0;
  set key "action" of data to the jumping action;
  if get key "score" of data is some let score be the value:
    say "Starting score: [score][line break]";
  let temperature be get key "temp" of data or 23 as an any;
```

### Nulls

Nulls do not represent any value, but the positive affirmation of a lack of a value. They could be used, for example, when deserialising JSON.

Nulls could possibly be unified with the internal `NIL_TY` kind.

### Options

An optional value, either nothing, or a value of a specific kind.

```
When play begins:
  let O1 be "Hello" as an option;
  let O2 be a text none option;
  if O1 is some let message be the value:
    say "Message: [message][line break]";
  let second message be value of O2 or "Goodbye";
```

### Results

A result contains either a wrapped value or an error message text.

```
When play begins:
  let R1 be 1234 as a result;
  if R1 is okay let score be the value:
    say "Score: [score][line break]";
  let R2 be a number error result with message "Oops!";
  if R2 is okay let score be the value:
    say "Score: [score][line break]";
  otherwise if R2 is an error let  message be the error message:
    say "Error! [message][line break]";
```

## Questions

1. For each of these there is the question of what the best representation is: a short and long block? Short block only? I also never looked at hashing and how that would work for these new kinds. (I'm not sure when hashing is actually used by Inform.)

2. How should Maps be implemented for efficient searching? Particularly string searching.

3. Would it be feasible to have general purpose [sum types](https://en.wikipedia.org/wiki/Algebraic_data_type) added to Inform, in which case Options and Results would not need any special handling, but could just be kinds supported by default?

4. Should the unchecked phrases actually be included, or should only the safe phrases be supported? (If someone needed unsafe phrases they could perhaps be provided by a separate extension.)
