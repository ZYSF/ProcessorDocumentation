# Portability Between 32- And 64-bit Models

## Design

The second-generation architecture (relative to the original concept) is designed specifically so that "system" programs can be seamlessly portable between 32-bit and 64-bit processor implementations.

The design features explained here do not cover _everything_ - they're designed specifically for system and embedded/hardware-level programs.

That means that differences in things like pointer arithmetic and `size_t`-like values can mostly be abstracted away _at the binary/instruction-decoding level_ so that low-level code shouldn't generally need to be recompiled to target different variants. It _doesn't_ help you abstract away any advanced maths though (for example to implement arbitrary C-like types, besides pointers). This keeps the system simple and focused.

## Advantages

* Could potentially lessen development/deployment/testing costs substantially for porting some pieces of software
* Makes it easier to work with simple 32-bit-only hardware architectures from 64-bit mode (doesn't assume the hardware bus, or processor, is designed for optimised 64-bit reads/writes)
* Makes it easier to work with shared-memory structures on multiprocessor systems (e.g. a 32-bit processor could easily share some 64-bit-safe pointers to it's structures)
* Fits in well with the other detectable features, so it should be very easy to write portable programs in practice (at least once the features mature!)
* For 32-bit-only builds, the instructions can also be used for to optimise initialising simple "long long" values from "int" values
* For possible mixed-mode designs in the future, makes switching between 32-/64-bit mode significantly simpler!

## Disadvantages

* Can be wasteful of memory, generally requires reserving 64-bit memory spaces for pointers and other values using the mechanism
* Not optimal for performance (on higher-end 64-bit implementations it'd be faster to read/write whole 64-bit values at a time, and on lower-end implementations it'd be faster to remove or discard the higher 32-bits entirely)
* Not optimal for code density (requires a sequence of two instructions to read/write a "pointer-sized" value

In other words, for performance it's still ideal to optimise every program for the exact processor configuration it'll be running on.

## How It Works

The initial read/write architecture was very simple, aside from allowing a separate, simple I/O bus it basically just allowed 32-bit reads & writes. Eventually I started adding more features, but (for convenience and general efficiency) most memory operations are still assumed to be 32-bit.

Since the initial design was internally 64-bit but was based on the assumption of a 32-bit-oriented external bus, it made it remarkably easy to produce 32-bit "emulations" of some of the 64-bit operations.

To write an unsigned pointer-sized value:

    write32 r2, r3, 0    ; Write the low 32-bits of register #2 to the address of register #3 (plus zero)
    write32hz r2, r3, 4  ; Write the high 32-bits of register #2 to the address of register #3 plus 4 (conceptually this would be "zero-extended" if it doesn't fill)

Or for a signed pointer-sized value (e.g. maybe a +/- offset from a memory pointer)

    write32 r2, r3, 0    ; Write the low 32-bits of register #2 to the address of register #3 (plus zero)
    write32hx r2, r3, 4  ; Write the high 32-bits of register #2 to the address of register #3 plus 4 (conceptually this would be "sign-extended if it doesn't fill)

To read, there is no such signed/unsigned distinction. On a 32-bit platform, the second instruction will either be ignored completely, or the processor will simply read the value from memory and ignore it:

    read32 r2, r3, 0     ; Read our value back to r2 (from address at r3+0)
    read32h r2, r3, 4    ; Conceptually read our higher-half (from address at r3+4), assigning it to bits [63:32] of register r2 (or ignoring it in 32-bit mode)

## Detecting Word Size

While some other features can be determined from the control registers, detecting word size currently has to be done by checking mathematical features. An easy way would be causing a 32-bit overflow (e.g. 0xFFFFFFFF + 1) and checking if the unsigned result is > 0xFFFFFFFF. Or you could use shifting etc. to see if you can set/check a higher bit.

In the future there might be some faster or more convenient way, i.e. to quickly branch to your optimised functions depending on whether you're on a 32-bit or 64-bit implementation.

## Required Extensions

Really the "control register" operations were designed poorly, meant to mimic a kind of internal memory bus that never really worked like memory. In practice this meant that 32-bit and 64-bit code required separate instructions to read/write the same/very-similar control registers (to line up with the data/IO encodings, i.e. reading a potentially 64-bit register would be treated a bit like a 64-bit memory read, separate to reading the same register on a 32-bit implementation).

Since the special `read32h` and associated instructions above already broke the perfect alignment of bits in the read/write encodings, it seemed natural to sort of dispose with the old CTRL encodings entirely, in favour of instructions which just read the control register.

So future versions will probably use a consistent `ctrlin`/`ctrlout` encoding for both 32-/64-bit versions.
