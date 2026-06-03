# PyramidOS Technical Review — 2026-06-03

## Executive Summary

PyramidOS is a serious early-stage i386 hobby kernel with a strong identity: an
Arabic-first operating-system experience from the bootloader upward. The project
already has more substance than a typical "hello kernel": it includes a custom
BIOS boot path, protected-mode entry, interrupt handling, basic memory managers,
input/time drivers, a shell, early storage probing, a VFS foundation, DevFS, MBR
partition discovery, and early PyFS probing.

The project is **not yet close to being a practical general-purpose OS**. Its
current value is as a systems-programming platform and as a distinctive
Arabic-first kernel project. The next phase should focus on correctness and
boring reliability, not on GUI, networking, AI, or advanced desktop features.

## Verdict

| Dimension | Assessment | Notes |
|---|---:|---|
| Kernel bring-up seriousness | 8/10 | Real boot, interrupts, PMM/VMM, drivers, shell, and VFS foundations exist. |
| Arabic-first identity | 9/10 | This is the project's strongest differentiator. Keep it central. |
| Product readiness | 4/10 | Useful as an educational/kernel research project, not as an end-user OS yet. |
| Documentation quality | 7/10 | Better than average, but needs stronger risk tracking and less overclaiming. |
| Engineering risk | High | Memory layout and boot/runtime invariants need hard guards. |

## What Is Strong

### 1. The project has a clear identity

The Arabic-first bootloader, Arabic strings/forms tooling, and long-term console
/ GUI localization roadmap make PyramidOS more interesting than a generic toy
kernel. This should remain the core project narrative.

### 2. The codebase is organized sensibly

The current layout is understandable:

```text
boot/src/legacy/       BIOS boot stages and boot assets
kernel/arch/i386/      architecture-specific entry, IDT, linker, CPU helpers
kernel/core/           PMM, VMM, heap, shell, debug, self-tests
kernel/drivers/        terminal, keyboard, PIT, RTC, ATA, block, MBR
kernel/fs/             VFS, nullfs, devfs, pyfs
kernel/lib/            freestanding helpers
```

This organization is good enough to keep scaling through the next storage and
VFS milestones.

### 3. The project has real subsystems

Current implemented or partially implemented subsystems include:

- custom legacy BIOS Stage 1 / Stage 2 bootloader;
- A20 enable and protected-mode switch;
- boot information handoff with E820 memory map;
- IDT / ISR / IRQ path;
- PIC, PIT, RTC, keyboard, terminal;
- PMM bitmap allocator;
- early VMM/paging;
- kernel heap;
- shell diagnostics;
- ATA PIO read path;
- block device registry;
- MBR partition discovery;
- DevFS;
- VFS mount and file descriptor foundation;
- PyFS superblock probe.

That is a real foundation.

## Main Findings

### Finding 1 — The memory layout is the highest-risk area

The documentation says the kernel is loaded at 1 MiB, but the linker script and
bootloader currently use `0x00010000`, which is **64 KiB**, not 1 MiB.

Current layout observed from the project:

| Region | Current address / behavior | Risk |
|---|---:|---|
| BootInfo | `0x5000` | Reasonable early fixed address, but should be documented as reserved. |
| Kernel load | `0x10000` | Comment mismatch: this is 64 KiB, not 1 MiB. |
| PMM bitmap | `0x20000` | Fixed placement can collide with a growing kernel. |
| Kernel image write LBA | sector 60 | Stage 2 size and disk layout need build-time guards. |

The kernel currently has only about 64 KiB between `0x10000` and `0x20000` before
it risks colliding with the fixed PMM bitmap. Even if the current binary fits,
this is fragile and will break as the kernel grows.

**Recommendation:** make `kernel_start` / `kernel_end` linker symbols
authoritative, pass reserved regions explicitly, and stop relying on a fixed PMM
bitmap address unless the address is proven safe at boot.

### Finding 2 — Build-time guards are missing

The Makefile has useful structure, but it does not yet enforce several critical
boot-image invariants.

Required guards:

| Guard | Why it matters |
|---|---|
| Stage 1 is exactly 512 bytes | MBR boot sectors must fit exactly one sector. |
| Stage 2 size <= `STAGE2_SECTORS * 512` | Prevents Stage 2 from overwriting reserved/kernel image space. |
| Kernel image does not exceed disk image capacity | Prevents silent truncation or unusable images. |
| Stage 2 end sector < kernel start sector | Prevents bootloader/kernel overlap on disk. |
| Kernel physical range does not overlap PMM bitmap | Prevents memory corruption after PMM init. |
| BootInfo/E820 pages are marked reserved | Prevents allocator from reusing boot handoff data too early. |

**Recommendation:** add a `make check-image-layout` target and make the disk
image depend on it.

### Finding 3 — E820 handling needs stricter page rounding

The PMM correctly starts by marking all memory used, then frees E820 usable
regions. However, usable-memory ranges should be rounded conservatively.

Recommended rule:

```text
When freeing usable memory:     align start up,   align end down.
When reserving used memory:     align start down, align end up.
```

That prevents partially usable boundary pages from being incorrectly freed.

### Finding 4 — BootInfo should be filled consistently

`BootInfo.mmap_count` is a `uint32_t`, but Stage 2 currently writes only a word
at `BootInfo + 16`.

Recommended fix:

```asm
xor eax, eax
mov ax, [e820_entry_count]
mov dword [es:di+16], eax
```

Also consider zeroing the whole BootInfo structure before filling fields, so
future fields start in a known state.

### Finding 5 — The shell is becoming a command pile

The shell works for bring-up, but command dispatch should soon move from a long
conditional chain to a command table.

Recommended shape:

```c
typedef void (*shell_command_fn)(int argc, char **argv);

typedef struct {
    const char *name;
    const char *summary;
    shell_command_fn run;
} ShellCommand;
```

This will make it easier to add `ls`, `cat`, Arabic aliases, help text, and
developer-only commands without making `shell.c` fragile.

### Finding 6 — The terminal needs scrolling

The VGA terminal clears when reaching the bottom. That is acceptable during early
bring-up but painful for diagnostics. Implement simple scroll-up behavior before
adding more verbose VFS/storage commands.

### Finding 7 — Repository hygiene needs cleanup

The uploaded project archive includes `.git/` and `.vs/`. For source releases,
review zips, or handoff archives, these should not be included.

Required hygiene:

- keep `.gitignore` entries for IDE/build outputs;
- remove tracked `.vs/slnx.sqlite` from the repository;
- do not distribute `.git/` inside review/handoff zips;
- add an archive command that exports only source, docs, and tooling.

### Finding 8 — Version labels should be unified

Current documentation and runtime messages disagree between `v0.8` and `v0.8.1`.

Recommended single source of truth:

```c
#define PYRAMIDOS_VERSION "v0.8.1"
```

Use it in boot banners, shell output, docs, and release notes.

## Recommended Next Milestone

The next milestone should be:

```text
PyramidOS v0.9 — Storage + VFS Reality Check
```

Do not add GUI, networking, audio, package management, Baa/Takween integration,
or AI features before this milestone is complete.

### v0.9 Goals

| Priority | Goal | Why |
|---:|---|---|
| P0 | Fix/document boot memory layout | Prevents silent corruption. |
| P0 | Add image-layout guards | Prevents invalid images from building. |
| P0 | Make PMM reserved ranges authoritative | Prevents allocator from touching kernel/bitmap/boot data. |
| P1 | PyFS directory/file reads | Turns PyFS from probe into usable storage. |
| P1 | VFS-backed `ls` and `cat` | Makes storage testable from the shell. |
| P1 | Terminal scrolling | Makes diagnostics usable. |
| P2 | Shell command table | Keeps shell maintainable. |
| P2 | QEMU smoke-test checklist/script | Reduces regression risk. |
| P2 | Clean release archive process | Improves project handoff quality. |

## v0.9 Definition of Done

PyramidOS v0.9 should not be considered complete until all checks below pass:

```text
make clean && make
make clean && make debug
make clean && make STRICT=1
make run
```

KShell verification:

```text
diagnose
blkinfo
mounts
pyfs_sb
ls /dev
ls /py
cat /py/<small-test-file>
diskread 0
```

Required results:

- kernel boots to shell;
- no panic during `diagnose`;
- `blkinfo` shows detected real block devices/partitions correctly;
- `/dev` is mounted and readable;
- `/py` is mounted only when a valid PyFS superblock exists;
- `ls` and `cat` use VFS, not direct filesystem shortcuts;
- terminal output scrolls instead of clearing unexpectedly;
- build fails loudly if image layout invariants are violated.

## What Not To Do Next

Avoid these until v0.9 is complete:

- GUI or desktop shell;
- networking;
- audio;
- multitasking/userland expansion;
- Baa/Takween migration;
- AI/local-intelligence features;
- advanced Arabic shaping in the kernel console.

These are attractive, but they will hide the most urgent correctness problems.

## External Engineering References

These references support the project direction and the review recommendations:

- OSDev GCC Cross-Compiler: https://wiki.osdev.org/GCC_Cross-Compiler
- OSDev Why do I need a Cross Compiler?: https://wiki.osdev.org/Why_do_I_need_a_Cross_Compiler%3F
- OSDev Detecting Memory (x86): https://wiki.osdev.org/Detecting_Memory_%28x86%29
- OSDev Higher Half Kernel: https://wiki.osdev.org/Higher_Half_Kernel
- OSDev Bare Bones: https://wiki.osdev.org/Bare_Bones
