# PyramidOS Project Positioning

## Honest Position

PyramidOS is currently best described as:

> An Arabic-first educational/research operating-system kernel for x86 legacy
> BIOS environments, with a custom bootloader, early kernel subsystems, and a
> long-term vision for a sovereign Arabic-first OS experience.

It should **not** currently be described as:

- a Windows/Linux replacement;
- a daily-driver desktop OS;
- secure enough for real user data;
- production-ready;
- hardware-portable;
- POSIX-compatible;
- ready for app developers.

## Why This Positioning Matters

Overstating the project makes the roadmap look unrealistic. Positioning it as a
serious Arabic-first kernel project makes it impressive and believable.

## Strong Public Description

Use this when presenting the project:

> PyramidOS is an experimental Arabic-first operating-system kernel built from
> scratch in C and Assembly. It currently targets 32-bit x86 legacy BIOS systems
> and includes a custom bootloader, protected-mode kernel bring-up, interrupt
> handling, memory management, a kernel shell, hardware timer/keyboard/RTC
> drivers, early ATA storage support, and a VFS foundation. The long-term goal is
> to explore what a sovereign Arabic-first computing environment could look like,
> starting from the lowest layers of the system.

## Short Description

> Experimental Arabic-first x86 kernel and bootloader project.

## Current Capability Statement

PyramidOS can currently:

- boot through a custom legacy BIOS path;
- enter 32-bit protected mode;
- initialize core CPU interrupt infrastructure;
- use basic memory management;
- accept keyboard input in a kernel shell;
- display output in VGA text mode;
- read time from RTC;
- perform early read-only ATA/block-device probing;
- expose a basic VFS/DevFS foundation;
- probe a PyFS superblock.

PyramidOS cannot yet:

- run user programs;
- provide process isolation;
- write to disk safely;
- recover from most driver errors;
- display a real Arabic console with shaping and bidi at the kernel level;
- provide a GUI;
- connect to networks;
- protect user data;
- run on modern UEFI-only machines without compatibility support.

## Recommended Roadmap Narrative

Keep the roadmap in this order:

```text
1. Boot and memory correctness
2. Storage and VFS reality
3. Shell and diagnostics polish
4. Userland/scheduler groundwork
5. Arabic-correct console rendering
6. Filesystem writing and installer experiments
7. GUI/desktop experiments
8. Baa/Takween integration
9. Networking and higher-level platform work
```

Arabic-first identity should stay visible at every layer, but correctness must
come before presentation.
