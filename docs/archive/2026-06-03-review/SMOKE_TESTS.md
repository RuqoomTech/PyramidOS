# PyramidOS Smoke Tests

This document defines the minimum checks before declaring a PyramidOS change
healthy. These are not a replacement for future automated tests; they are the
current practical verification path for a bootable freestanding kernel.

## Build Checks

Run all supported build profiles:

```bash
make clean && make
make clean && make debug
make clean && make STRICT=1
```

Expected result:

```text
Build Complete: build/pyramidos.img
```

If any profile fails, the change is not ready.

## Boot Check

Run:

```bash
make run
```

Expected result:

```text
PyramidOS reaches KShell without panic.
Keyboard input works.
The cursor is visible.
```

## Core Runtime Checks

At KShell, run:

```text
help
mem
time
uptime
diagnose
```

Expected result:

```text
help lists commands
mem reports total/free memory
RTC/time command does not panic
uptime increases over time
diagnose completes without panic
```

## Storage / VFS Checks

Run:

```text
blkinfo
mounts
pyfs_sb
diskread 0
```

Expected result:

```text
blkinfo lists detected block devices only
mounts shows / and /dev, plus /py only when PyFS is valid
pyfs_sb reports a valid superblock only on valid PyFS media
diskread 0 returns a hex dump or a clear read error, not undefined behavior
```

When v0.9 adds VFS-backed file reading, also run:

```text
ls /dev
ls /py
cat /py/<small-test-file>
```

Expected result:

```text
ls /dev lists devfs entries
ls /py lists PyFS root directory entries when mounted
cat reads a small file through VFS
missing paths report clean errors
```

## Negative Checks

These checks prevent false confidence:

| Scenario | Expected behavior |
|---|---|
| No ATA disk present | No fake `disk0`; storage commands fail cleanly. |
| Invalid MBR | No bogus partitions; diagnostic message is clear. |
| Invalid PyFS superblock | `/py` is not mounted, or PyFS reports a clean invalid-superblock error. |
| Oversized Stage 2 | Build fails before creating a misleading image. |
| Kernel overlaps PMM bitmap | Build or boot fails loudly with a clear error. |

## Regression Notes Template

Use this template when documenting a failed smoke test:

```text
Build command:
QEMU command:
Last visible boot message:
KShell command that failed:
Expected result:
Actual result:
Recent changed files:
Screenshot/log:
```
