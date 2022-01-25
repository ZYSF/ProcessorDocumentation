# ProcessorDocumentation

Documentation for my latest processor designs.

## Introduction & History

The design originates here: https://github.com/ZYSF/gen1cpu which was my "first-generation" processor design, but the documentation in this repository is for my "second-generation" or current design (as of 2022).

Major changes to the implementation/s (which I haven't uploaded fully, at least at this stage) include gradually moving towards an algorithmic/autogenerated design which is already capable of producing 32-bit as well as 64-bit implementations. This means some necessary breaking changes were needed, so I decided to branch the documentation off separately while finalising how to continue developing/releasing future implementations. Other improvements have been made to ease testing.

The contents of this repository is developed from scratch (based on my previous/original designs), and made freely available for whatever purposes (under the terms of "unlicense").

### Breaking Changes/Compatiblity Notes

Firstly there were/are a lot of bugs, so much of the logic shouldn't be expected to work without specific testing of each feature/in each configuration!

Architecturally, though, there is one major breaking change: The `READ32H` and `WRITE32H`/`WRITE32HX` opcodes are moved to better encodings, and are now more accurately-specified for 32-bit mode. This allows for more thorough implementation of the rest of the planned read/write instructions, and also serves to allow easier implementation of 32-/64-bit-neutral programs.

Other new instructions either fit into previously-unused encodings or only apply to 32-bit mode (which was not fully documented or implemented in the original "gen1cpu").

Besides the new/modifiedinstructions various bugfixes (some of which have been back-ported to the original "simple" codebase), the major version has been bumped up to 2!

Creating a new "major version" within the same architecture allows for the first experiments in backwards-compatibility; Theoretically, code can detect/adapt whether they're running on a "gen1" or "gen2" processor, and (theoretically) code running on a 64-bit "gen2" processor could use existing capabilities to specially patch out the conflicting instructions to create a compatibility mode for (64-bit only) gen1 instructions.

The current version of the processor generator can even backport some fixes and other non-conflicting updates to the "gen1" implementation!

## Index of Documents

* [Instruction Set](InstructionSet.md) documents the semantics and encoding of each of the standard instructions.
* [Control Registers](ControlRegisters.md) documents the meanings, encodings and indices of the control registers.
* [Modes & Exceptions](ModesAndExceptions.md) documents the user-mode/system-mode switching and the meanings of the exception codes.
* [Startup & Reset State](StartupAndResetState.md) documents the startup/reset sequence and what state to expect the core to be in at initialisation.
* [Addressing Modes](AddressingModes.md) should help to clarify the role of the MMU and the ways in which instructions, memory locations, I/O and control registers are addressed

NOTE: Updated "RISC Emulation" circuitry as in the first generation still exists in the second-generation design but this has been de-prioritised in favour of other features (at least for now), and has been mostly isolated away from the core ISA design (except for the relevant mode-setting & instruction overriding information).
