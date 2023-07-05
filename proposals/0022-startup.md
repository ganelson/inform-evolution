# (IE-0022) Tidying the startup sequence

* Proposal: [IE-0022](0022-startup.md)
* Discussion PR link: [#22](https://github.com/ganelson/inform-evolution/pull/22)
* Authors: Dannii Willis
* Language feature name: None
* Status: Implemented but unreleased
* Related proposals: None

## Summary

To tidy up the startup sequence for better regularity and flexibility.

## Motivation

Inform has a fairly involved startup sequence: when the virtual machine starts running a storyfile, Inform includes code to set up many things before it's ready to run the authors' own code. But the startup sequence is fairly convoluted, and inconsistent between regular and Basic Inform (as well as inaccurately documented).

First, here's regular (non-Basic) Inform:

- `Main` (WorldModelKit - OrderOfPlay)
    - Startup rules (Variables and Rulebooks)
        - `INITIALISE_MEMORY_R` (OrderOfPlay)
            - `VM_PreInitialise` (Architecture16Kit/Architecture32Kit - Startup)
                - Glk checks
        - `VIRTUAL_MACHINE_STARTUP_R` (OrderOfPlay)
            - Starting the virtual machine activity (Basic Inform - Miscellaneous Definitions)
                - `FINAL_CODE_STARTUP_R`
            - `VM_Initialise` (Architecture16Kit/Architecture32Kit - Startup)
                - Various things

And here's Basic Inform:

- `Main` (BasicInformExtrasKit - Miscellany)
    - `VM_Initialise` (Architecture16Kit/Architecture32Kit - Startup)
        - Various things
    - `INITIALISE_MEMORY_R` (Miscellany)
        - `VM_PreInitialise` (Architecture16Kit/Architecture32Kit - Startup)
            - Glk checks

There are several issues here:

- It's hard to predict in what order things are run: `VM_Initialise`'s docs say it's almost the first routine called, which is blatantly false in non-Basic Inform, and technically incorrect in Basic Inform because it actually *is* the first routine called.
- `VM_PreInitialise` is run before `VM_Initialise` in regular Inform but after in Basic Inform!
- Basic Inform, despite defining the Startup Rules and Starting the Virtual Machine activity, doesn't actually run them.
- One function, `INITIALISE_MEMORY_R`, has somewhat different contents in regular vs Basic Inform, which could easily confuse someone trying to see what it does.

Furthermore, some of the startup sequence is accessible to Inform as rules (so that extensions or authors could modify where necessary), but a lot of it is implemented in functions that can only be modified through function replacement. In particular, `VM_Initialise` in Glulx handles setting up all of the Glk objects, making it hard to change just one part of it. Breaking up `VM_Initialise` was the major motivation for the older extension [Alternative Startup Rules](https://github.com/i7/extensions/blob/9.3/Dannii%20Willis/Alternative%20Startup%20Rules.i7x).

## Components affected

- [ ] No change to the natural-language syntax.
- [ ] No change to inbuild.
- [ ] No changes to inform7.
- [ ] No change to inter.
- [ ] No change to the Inter specification.
- [x] Minor changes to runtime kits.
- [x] Minor changes to the Standard Rules and Basic Inform.
- [x] Minor change to documentation.
- [ ] No change to the GUI apps.

## Impact on existing projects

If someone had manually been tinkering with the startup rules or the starting the virtual machine activity there's a chance their changes might conflict with what this proposal would change.

## Proposal

A new function `Startup` will be added, which will do two things: run an Architecture16Kit/Architecture32Kit function `VM_Check_Functionality` which would test the VM meets the minimum requirements for Inform (in Glulx it would test a few Glulx/Glk gestalts; in Z-Code it would probably do nothing.) Then it would run the Startup Rules. And that's it - everything else in the startup sequence would be moved to Inform-accessible rules.

- `Main` (WorldModelKit/BasicInformExtrasKit)
    - `Startup` (BasicInformKit - Startup)
        - `VM_Check_Functionality` (Architecture16Kit/Architecture32Kit - Startup)
        - Startup rules
            - Starting the virtual machine activity

The startup sequence would be reorganised according to these principles:

1. Differences between regular Inform and Basic Inform should be minimised. But when regular Inform needs to do something that Basic Inform doesn't, there are three options:

    - Do it in a new rule
    - Replace a rule from Basic Inform with a new rule
    - Wrap the rule's function, with a `replace(keeping)(from BasicInformKit)` directive, so that the original function can still be called

    Each of these are appropriate choices for different situations.

2. Giant functions should be avoided, instead each function or rule should have one thing that it implements. (In the sense that Glk windows and streams need separate rules, but not that each global variable needs to be initialised in its own rule, related globals can be initialised together.)

3. The starting the virtual machine activity will be better utilised for the startup sequence. Low level rules currently found in the startup rules will be moved to the activity, so that the startup rules can be mostly left for the author's own code.

    The three rulebooks of the activity also help us organise the startup sequence:

    - The before rules are for setting up the essential systems of Inform, but cannot use or access any of the IO systems.
    - The for rules are where the Glk IO systems are set up.
    - The after rules are a good place for other miscellaneous IO-dependent startup rules. They're also a good place for extensions to place their code (though Glk extensions will likely need to use all three rulebooks.)

    Using the three activity rulebooks also means that the startup rules won't need as much manual ordering as there currently is, because they'll have a natural ordering.