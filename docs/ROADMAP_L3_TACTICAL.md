# Layer 3: Tactical Roadmap (Current Sprint)

**Focus:** v0.9 stabilization — boot/memory correctness, storage/VFS reality,
and repeatable smoke testing.

**Current Kernel:** v0.8.1 documentation/hardening baseline  
**Target:** v0.9 — Storage + VFS Reality Check

> The full risk register, release gates, and v0.9 scope now live in
> [`ROADMAP.md`](ROADMAP.md). This Layer 3 file is the short execution checklist.

---

## Sprint Rule

Do not start GUI, networking, audio, package management, full Arabic shaping,
Baa/Takween migration, scheduler expansion, or local AI work during this sprint.

The goal is boring reliability: build guards, boot/memory correctness, terminal
scrolling, shell maintainability, PyFS real reads, and VFS-backed commands.

---

## 1. P0 — Boot and Image Layout Hardening

- [ ] Add explicit `KERNEL_START_SECTOR ?= 60` in the build configuration.
- [ ] Fail the build if `stage1.bin` is not exactly 512 bytes.
- [ ] Fail the build if `stage2.bin` exceeds `STAGE2_SECTORS * 512`.
- [ ] Fail the build if Stage 2 disk range overlaps the kernel disk range.
- [ ] Fail the build if `kernel.img` cannot fit in the disk image.
- [ ] Print Stage 2 size, kernel image size, and sector ranges during image creation.

**Acceptance:** an invalid image layout must fail during `make`, not during QEMU
boot.

---

## 2. P0 — Boot Memory Layout Correction

- [ ] Decide whether the near-term kernel load address remains `0x10000` or moves to `0x100000`.
- [ ] Add linker symbols for `kernel_start` and `kernel_end`.
- [ ] Reserve the actual linker-provided kernel range in PMM.
- [ ] Ensure the PMM bitmap cannot overlap the kernel image silently.
- [ ] Reserve BootInfo, E820 map, stack, page tables, and allocator metadata.

**Acceptance:** the allocator never frees kernel, boot handoff, bitmap, stack, or
page-table pages.

---

## 3. P0 — BootInfo and E820 Correctness

- [ ] Zero the BootInfo structure before Stage 2 fills it.
- [ ] Write `mmap_count` as a 32-bit field to match `BootInfo` in C.
- [ ] Document `kernel_load_addr` representation clearly in code comments.
- [ ] Make E820 free-range handling conservative:
  - [ ] freeing usable ranges: align start up, align end down;
  - [ ] reserving used ranges: align start down, align end up.
- [ ] Add diagnostics for malformed/empty E820 maps.

**Acceptance:** C and assembly agree exactly on the BootInfo layout, and PMM
never frees partially usable boundary pages.

---

## 4. P1 — Terminal and Shell Maintainability

- [ ] Implement terminal scroll-up instead of clear-on-bottom.
- [ ] Convert shell command dispatch to a command table.
- [ ] Generate `help` output from command metadata.
- [ ] Prepare the command table for English + Arabic aliases.

**Acceptance:** long diagnostic output remains visible, and adding commands no
longer requires a fragile conditional chain.

---

## 5. P1 — PyFS Real Read-Only Bring-up

- [ ] Validate PyFS superblock with clear errors.
- [ ] Implement root directory listing.
- [ ] Implement open/read for at least one regular file.
- [ ] Ensure all disk access goes through the block layer.
- [ ] Keep PyFS read-only until read-path correctness is proven.

**Acceptance:** PyFS is no longer only a superblock probe; it can expose real
files through VFS.

---

## 6. P1 — VFS-Backed KShell Storage Commands

- [ ] Add `ls <path>` through VFS.
- [ ] Add `cat <path>` through VFS for small files.
- [ ] Verify `ls /dev` works through DevFS.
- [ ] Verify `ls /py` works when PyFS is mounted.
- [ ] Verify invalid paths return readable errors.

**Acceptance:** shell storage commands test the generic VFS layer instead of
calling filesystem internals directly.

---

## 7. P2 — Smoke Tests and Release Hygiene

- [ ] Keep the smoke-test command list in `ROADMAP.md` updated with expected results.
- [ ] Add optional QEMU smoke-test automation if practical.
- [ ] Add a clean source export/archive process.
- [ ] Exclude `.git/`, `.vs/`, `build/`, generated images, and editor files from handoff archives.
- [ ] Unify visible version strings (`v0.8.1` vs `v0.8`) before release tagging.

**Acceptance:** a reviewer can build, boot, test, and inspect a clean archive
without private/editor artifacts.

---

## Completed v0.8.x Foundation Work

- [x] Panic/debug foundation with register dump path.
- [x] PMM next-fit allocator optimization.
- [x] `cpu_idle()` via STI+HLT for idle waiting.
- [x] Runtime diagnostics through `diagnose`.
- [x] VGA terminal extraction into `kernel/drivers/terminal.c/.h`.
- [x] ATA LBA28 read path and `diskread` command.
- [x] IDENTIFY-based disk detection.
- [x] Block device registry.
- [x] DevFS mounted at `/dev`.
- [x] MBR partition registration.
- [x] PyFS superblock probe and `/py/superblock` verification.
