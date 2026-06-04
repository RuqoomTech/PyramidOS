# PyramidOS v0.9 Stabilization Plan

## Milestone Name

**PyramidOS v0.9 — Storage + VFS Reality Check**

## Purpose

v0.9 is a stabilization milestone. Its job is to turn the current storage and
VFS foundation into something testable from the shell while removing the most
obvious boot/memory-layout risks.

This milestone intentionally avoids new high-level features. No GUI, networking,
Baa/Takween migration, multitasking expansion, audio, or AI/local-intelligence
work should enter v0.9 unless the core correctness work is already complete.

## Scope Summary

| Area | Work | Priority |
|---|---|---:|
| Boot/image layout | Enforce disk/image size and overlap checks | P0 |
| Memory layout | Fix/comment kernel load address and reserve real kernel range | P0 |
| PMM | Conservative E820 page rounding and reserved-region hardening | P0 |
| BootInfo | Fill all fields consistently and zero structure before use | P0 |
| Terminal | Implement scroll-up instead of clear-on-bottom | P1 |
| Shell | Replace command pile with command table | P1 |
| VFS/PyFS | Implement directory/file reads, not only superblock probe | P1 |
| KShell tools | Add VFS-backed `ls` and `cat` | P1 |
| QA | Add repeatable QEMU smoke-test checklist/script | P2 |
| Repository hygiene | Add clean archive/export process | P2 |

## Task Breakdown

### T1 — Add build-time image layout guards

**Goal:** invalid boot images must fail during `make`, not during QEMU boot.

Acceptance criteria:

```text
stage1.bin is exactly 512 bytes
stage2.bin <= STAGE2_SECTORS * 512 bytes
Stage 2 disk range does not overlap kernel disk range
kernel.img fits in pyramidos.img
build output prints stage2/kernel sizes clearly
```

Suggested implementation:

- add `KERNEL_START_SECTOR ?= 60` in Makefile;
- use shell/Python checks after Stage 2 and kernel image creation;
- make `$(DISK_IMG)` depend on the check.

### T2 — Correct and harden the boot memory layout

**Goal:** PMM must reserve the actual kernel and metadata ranges.

Acceptance criteria:

```text
linker.ld exposes kernel_start and kernel_end
PMM reserves kernel_start..kernel_end
comments no longer call 0x10000 "1MB"
PMM bitmap cannot overlap kernel image silently
```

### T3 — Fix BootInfo field filling

**Goal:** BootInfo layout in assembly and C must match exactly.

Acceptance criteria:

```text
Stage 2 zeroes BootInfo before writing fields
mmap_count is written as a dword
reserved byte/field is zeroed
kernel_load_addr representation is documented
```

### T4 — Make PMM E820 handling conservative

**Goal:** only fully usable frames are freed.

Acceptance criteria:

```text
pmm_mark_region_free aligns start up and end down
pmm_mark_region_used aligns start down and end up
zero-length/underflow edge cases are handled
pmm diagnostics still pass
```

### T5 — Add terminal scrolling

**Goal:** diagnostics should not disappear when output reaches the bottom row.

Acceptance criteria:

```text
printing past row 24 scrolls content upward
cursor remains visible
clear still resets the screen
long command output remains readable
```

### T6 — Convert shell commands to a command table

**Goal:** adding storage commands and Arabic aliases should be simple.

Acceptance criteria:

```text
commands are registered in a table
help is generated from command metadata
existing commands still work
aliases can be added without duplicating command bodies
```

### T7 — Implement PyFS real reads

**Goal:** PyFS becomes a usable read-only filesystem, not only a probe.

Acceptance criteria:

```text
PyFS validates the superblock
PyFS can list the root directory
PyFS can open/read at least one regular file
invalid images fail cleanly
all reads go through the block layer
```

### T8 — Add VFS-backed `ls` and `cat`

**Goal:** storage should be testable through generic VFS paths.

Acceptance criteria:

```text
ls /dev works through VFS
ls /py works through VFS when PyFS is mounted
cat /py/<file> works for a small text file
commands report clear errors for missing paths and unsupported operations
```

### T9 — Add smoke-test documentation/script

**Goal:** every storage/VFS change has repeatable verification.

Acceptance criteria:

```text
docs/SMOKE_TESTS.md exists
manual QEMU test sequence is documented
optional scripts/qemu_smoke.sh exists if automation is practical
expected KShell outputs are listed
```

### T10 — Clean release/archive process

**Goal:** shared archives should not contain `.git/`, `.vs/`, build outputs, or
other private/editor state.

Acceptance criteria:

```text
make archive or scripts/export_source.sh creates a clean zip/tarball
archive excludes .git, .vs, build, IDE files, and temporary files
docs explain how to create a handoff archive
```

## Definition of Done

v0.9 is complete when this sequence passes:

```bash
make clean && make
make clean && make debug
make clean && make STRICT=1
make run
```

Manual KShell checks:

```text
help
diagnose
mem
blkinfo
mounts
pyfs_sb
ls /dev
ls /py
cat /py/<small-test-file>
diskread 0
reboot
```

The milestone is not complete if any of these are true:

- kernel load comments still confuse `0x10000` and `1 MiB`;
- PMM bitmap overlap is possible without a loud panic/build failure;
- Stage 2 can exceed its reserved disk sectors silently;
- PyFS can only expose `/py/superblock` and cannot read real files;
- shell storage commands bypass VFS.

## Out of Scope

Explicitly not part of v0.9:

- GUI / framebuffer desktop;
- networking;
- audio;
- process scheduler/userland expansion;
- package manager;
- Baa/Takween integration;
- full Arabic shaping/bidi console;
- local AI or fuzzy command execution.
