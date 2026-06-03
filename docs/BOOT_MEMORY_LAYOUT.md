# Boot and Memory Layout Notes

This document records the current PyramidOS boot/memory layout and the required
hardening work before the kernel grows further.

## Current Boot Flow

```text
BIOS
  -> Stage 1 MBR boot sector
  -> Stage 2 legacy BIOS loader
  -> E820 memory detection
  -> kernel image load
  -> protected-mode switch
  -> kernel entry
  -> C kernel initialization
```

## Current Fixed Addresses

| Item | Address / value | Owner | Notes |
|---|---:|---|---|
| BootInfo | `0x5000` | Stage 2 -> kernel | Must remain reserved until kernel copies/parses it. |
| E820 map | Stage 2-defined low memory address | Stage 2 -> kernel | Must remain reserved until PMM has consumed it. |
| Kernel physical load | `0x10000` | linker + Stage 2 | This is 64 KiB, not 1 MiB. Existing comments must be corrected. |
| PMM bitmap | `0x20000` | kernel PMM | Fixed address; can collide with a growing kernel. |
| Stage 2 disk sector count | `STAGE2_SECTORS ?= 32` | Makefile / Stage 1 | Must be enforced against actual `stage2.bin` size. |
| Kernel disk start sector | sector `60` | Makefile / Stage 2 | Must remain beyond Stage 2 and within disk capacity. |

## Critical Documentation Correction

The linker script comment currently describes `0x10000` as 1 MiB. That is
incorrect.

```text
0x10000  = 65,536 bytes = 64 KiB
0x100000 = 1,048,576 bytes = 1 MiB
```

Any future design should explicitly choose one of these:

1. keep loading at `0x10000` temporarily and document it honestly; or
2. move the kernel to `0x100000` and update Stage 2, linker script, PMM, and VMM
   assumptions together.

## Reserved Regions Required by PMM

During PMM initialization, the allocator should reserve at least:

```text
[0x00000000, 0x00001000)        null page / BIOS low-memory safety
BootInfo page(s)                boot handoff structure
E820 map page(s)                BIOS memory map
Stage 2 occupied memory         if still needed after handoff
kernel_start..kernel_end        loaded kernel image
PMM bitmap range                physical allocator metadata
initial page tables             VMM metadata
initial kernel stack            entry/bootstrap stack
```

## Required Linker Symbols

Add linker-provided symbols and consume them from C:

```ld
kernel_start = .;
/* sections */
kernel_end = .;
```

Then expose them in C:

```c
extern uint8_t kernel_start[];
extern uint8_t kernel_end[];
```

PMM should reserve `kernel_start..kernel_end`, not a hardcoded guess.

## Page-Rounding Policy

Use conservative page rounding.

| Operation | Start | End | Reason |
|---|---|---|---|
| Free E820 usable region | align up | align down | Avoid freeing partial boundary pages. |
| Reserve used region | align down | align up | Ensure every touched page is protected. |

## Build-Time Layout Guards

The image build should fail if any invariant is false:

```text
stage1.bin size == 512
stage2.bin size <= STAGE2_SECTORS * 512
1 + STAGE2_SECTORS < KERNEL_START_SECTOR
kernel.img fits inside disk image
kernel physical range does not overlap PMM bitmap
kernel_end <= PMM_BITMAP_BASE or bitmap is moved dynamically
```

## Near-Term Recommendation

For v0.9, do the minimal safe step:

1. correct comments/docs that say `0x10000` is 1 MiB;
2. add linker symbols for `kernel_start` and `kernel_end`;
3. reserve the actual kernel range in PMM;
4. add image-layout build guards;
5. either move the PMM bitmap dynamically after `kernel_end` or move the kernel
   load address to 1 MiB and update all boot assumptions in one commit.
