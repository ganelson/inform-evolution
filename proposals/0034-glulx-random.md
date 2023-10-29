# (IE-0034) A new Glulx random algorithm

* Proposal: [IE-0034](0034-glulx-random.md)
* Discussion PR link: [#34](https://github.com/ganelson/inform-evolution/pull/34)
* Authors: Dannii Willis
* Status: Accepted
* Implementation: Not yet implemented

## Summary

Existing Glulx interpreters can have flawed random implementations, so move the
implementation of Inform's `random` function inside Architecture32Kit so that
we can ensure it is reliable.

## Motivation

In September 2023 it was revealed in [a forum discussion](https://intfiction.org/t/lack-of-randomness-sometimes-when-compiling-for-glulx/64533)
that some Glulx interpreters have very flawed random number generators. In
particular, when requesting a random number between 1 and N, if N is a power of
2, then there will only be N distinct sequences. Even worse, the initial numbers
of those sequences are often the same, the example code showing that when
requesting a random number between 1 and 4, the first number will always be a 3.
This could have dire consequences: if a murder mystery game picked randomly
between 4 suspects, in these interpreters it would always pick the third suspect.

The affected interpreters include Glulxe in Windows, and Git in all OSes. These
interpreters have since been patched to use much more reliable RNGs, but it can
take a long time for downstream interpreters to include these updates; many
interpreters are only updated infrequently, and some are no longer maintained,
including all Android Glulx interpreters.

Inform must provide high quality randomness out-of-the-box without authors needing
to dig into interpreter quirks. But as we can't rely on the interpreters being
patched in a timely manner, we need to instead move the implementation of Inform's
RNG inside Inform kit code (Architecture32Kit) to insulate authors from the flaws
of the interpreters. Many other programming languages also implement their RNGs
inside their library code rather than relying on the OS-provided RNG, so Inform
will be in good company doing this.

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [x] Minor changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor changes to runtime kits.
- [ ] No changes to the Standard Rules and Basic Inform.
- [ ] No change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

Authors will receive better quality random numbers without any change to their
code. However if anyone is unintentionally relying on the quirky results of
calling `random` with a negative value in Glulx, they will need to change what
they are doing.

## Details

The Inform 6 function `random` is a "system" function. This means that by default
it is not a true function, but instead just a shortcut for the VM-specific random
opcode. System functions can however be replaced by the author, so that is what
we will do to fix this problem, replacing it with a function in Architecture32Kit.

Considerations:

1. Ideally we want an RNG that produces high quality random numbers, has decently
    high performance, and does not use excessive amounts of memory. The [Xorshift](https://en.wikipedia.org/wiki/Xorshift)
    family of RNGs fits these criteria. Glulxe and Git switched to the
    [xoshiro128** algorithm](https://prng.di.unimi.it/), so we might as well
    use it too.

2. This RNG can be seeded with any non-zero value, and it is common to seed it
    with a timestamp. The Inform test suite however requires predictable randomness,
    which it currently accomplishes through Glulxe's `--rngseed` argument. So
    rather than seeding Inform's new RNG with a timestamp, we should instead seed
    it with `@random`. Even though the interpreter's RNG may be flawed, one call
    with a high enough range is probably much safer than calling it with a range
    of only 4 or 8. It has not yet been determined whether a full 32-bit `@random`
    call is safe in affected interpreters, or instead whether it should make a
    ranged call using a high prime number, or if any priming is necessary.

3. We need to replace the Inform 6 `random` function rather than introducing a
    new random function so that all randomness, even in extensions, uses the new
    algorithm.
    
    But the I6 `random` function does more than just returning a random number.
    As [documented in the DM4](https://www.inform-fiction.org/manual/html/s1.html#p45)
    `random` can also be called with multiple arguments, for it to randomly choose
    between. This however [does not work in Inform 7](https://intfiction.org/t/lack-of-randomness-sometimes-when-compiling-for-glulx/64533/101).
    Inline invocations like this
    ```
    To decide which number is todays number: (- random(1,2,3,4,5,6,7,8,9) -).
    ```
    produce the error
    > inform7: this inv of !random should have 1 argument(s), but has 9
    
    But calls to `random` in whole-function inclusions like this
    ```
    Include (-
    [ TodaysNumber;
        return random(1,2,3,4,5,6,7,8,9);
    ];
    -).
    ```
    compile, however they are silently modified during the Inter transformation
    to only include one argument in the final build.

    Inform 7 should consistently reject the multiple-argument form of `random`
    at all times.

4. There is a further complication: the Glulx I6 implementation of `random`
    does not match the behaviour described in the DM4. Instead of negative or
    zero values seeding the interpreter's RNG, it is passed directly to [the Glulx
    `@random` opcode](https://www.eblong.com/zarf/glulx/Glulx-Spec.html#opcodes_rand),
    before adding 1. This results in the following inconsistencies:

    | Argument | DM4/Z-Machine | Glulx |
    |----------|---------------|-------|
    | Positive N | Return a random number in the range 1 to N | Return a random number in the range 1 to N |
    | Zero | Switch the interpreter RNG to random mode | Return a full 32-bit random number |
    | Negative N | Seed the interpreter RNG in predictable mode | Return a random number in the range (N + 2) to 1

    Nobody would intentionally call `random(-10)` expecting it to return a number
    between -8 and 1, so we don't need to be concerned with preserving the I6
    behaviour of `random` in Glulx; instead we should make it conform to the DM4.

5. People may find the behaviour of the Glulx `@random` opcode more useful than
    the I6 `random` function, so we should also provide a function that mimics
    the `@random` opcode, but using the new RNG.