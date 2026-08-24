# Baa + Nazm + Takween Plan for Building PyramidOS

## Objective
Enable PyramidOS to be built with **Baa (language/compiler)**, **Nazm
(assembler/object writer)**, and **Takween (build system)** for real bare-metal
OS workflows (bootloader, freestanding kernel, disk image, QEMU run), not just
user-space apps.

This is a future integration track. It must not distract from the v0.9
boot/memory/storage gates.

## Current Gap Summary
- PyramidOS is currently **32-bit i386 freestanding** (`i686-elf-*`, custom linker, raw disk image).
- Baa currently focuses on **x86_64 Windows/Linux user-space targets**.
- Nazm 0.4 currently owns an **x86-64** Arabic assembly and ELF64/COFF path; it
  is Baa's admitted hosted production assembler but does not provide i386/ELF32
  objects.
- Takween now has cross-platform structured process execution, typed manifests,
  locks, and target discovery for hosted Baa projects. It does not yet own a
  freestanding OS profile, raw-image pipeline, or PyramidOS toolchain contract.
- ArbSh and Qalam are hosted developer-experience tools; neither is part of the
  kernel image or a prerequisite for building it.

## Design Rules

- Baa/Takween must support PyramidOS as a freestanding kernel target, not force
  hosted Windows/Linux assumptions into the build.
- The first success is a mixed build, not a full rewrite.
- The target architecture must converge across Baa, Nazm, Takween, and
  PyramidOS. An i386 Baa-only prototype may use the external assembler, but it
  must not be presented as the final sovereign toolchain.
- Baa should help PyramidOS move toward a sovereign application/runtime model
  later, but early kernel correctness remains C/Assembly-led.
- AI may assist with compiler tests and build scripts, but generated ABI/runtime
  code must be reviewed as unsafe until proven.

## Required Work in Baa
1. Add target: `i386-elf-baremetal`.
2. Add freestanding mode flags:
   - `--freestanding`
   - `--no-startup` (no `main`/CRT assumptions)
   - `--no-rt` (no hosted runtime/stdlib linkage)
3. Support custom entry symbol (example: `k_main` / `_start`).
4. Emit i386-compatible objects (`ELF32`) and assembly (`-m32` semantics).
5. Implement i386 ABI details (cdecl, stack alignment, callee/caller-saved rules).
6. Preserve low-level control needed by kernels (inline asm stability, volatile semantics, no hidden host dependencies).
7. Add regression suite for freestanding/kernel cases (no libc calls, no forbidden runtime refs).

## Required Work in Takween
1. Extend the existing structured argv/cwd/environment process layer from
   hosted Baa projects to explicit freestanding toolchains.
2. Add OS profile type in config (example: `النوع: نواة`).
3. Add pipeline stages:
   - assemble bootloader
   - compile kernel objects
   - link with linker script
   - build raw image
   - run in QEMU
4. Add structured capability discovery for `i686-elf-gcc`, `i686-elf-ld`,
   `nasm`, and `qemu-system-i386`; do not compose shell command strings.
5. Extend deterministic build graphs and lock evidence to custom linker scripts,
   boot objects, raw images, and the selected emulator/toolchain identities.

## Required Architecture Decision in Nazm

After PyramidOS v0.9 exits, choose one path before implementation expands:

1. **Coherent i386 path:** add and verify i386 instruction/ABI coverage plus
   ELF32 serialization in Nazm, then admit the same target through Baa and
   Takween; or
2. **Staged x86-64 OS migration:** keep the current i386 kernel build native,
   plan PyramidOS x86-64 bring-up, and reuse the admitted Baa/Nazm x86-64 path.

Decision criteria must include boot complexity, duplicate backend lifetime,
ELF32 relocation work, migration rollback, QEMU coverage, and the intended
long-term PyramidOS ABI. Do not silently combine a permanent i386 Baa backend
with an x86-64-only Nazm roadmap.

## Required Work in PyramidOS
1. Define Baa-safe kernel coding subset (no hosted assumptions).
2. Start with a mixed build (C + Baa), then migrate module-by-module.
3. Suggested migration order:
   - `kernel/lib` helpers
   - pure logic in `kernel/core`
   - selected `kernel/fs` pieces
   - drivers last (ATA/interrupt-sensitive paths)
4. Keep bootloader + linker script authoritative during early migration.

## Milestones (Execution Order)
1. **M0: Architecture Decision + Spec Freeze**
   Choose coherent i386 support or staged x86-64 migration, then define ABI,
   entrypoint, sections, object/relocation ownership, and forbidden runtime behavior.
2. **M1: Baa Freestanding Prototype**  
   Compile one minimal kernel function to `ELF32 .o` and link into current PyramidOS.
3. **M2: Takween OS Pipeline MVP**  
   Build + image + run (`qemu-system-i386`) in one command.
4. **M3: Mixed-Kernel Bring-up**  
   Boot PyramidOS with at least one production module compiled from Baa.
5. **M4: Expansion + QA Gates**  
   Determinism, cross-host reproducibility, panic/diagnostic parity checks.

## Definition of Done (First Real Success)
- An Arabic Takween kernel-profile build produces `build/pyramidos.img`.
- The corresponding run command boots to KShell in QEMU.
- At least one non-trivial kernel module is built from Baa and passes `diagnose`.
- No CRT/libc dependency appears in final kernel link.
- The assembler/object path is named explicitly: external i386 bridge or
  admitted Nazm target; no hidden host fallback is accepted.

## Immediate Next Actions
1. Keep implementation behind `ROADMAP.md` gates until PyramidOS v0.9 exits.
2. Record evidence for the i386-versus-x86-64 architecture decision.
3. Finalize the selected target spec (registers, calling convention, object format, relocation owner).
4. Add Baa freestanding flags and negative tests.
5. Extend Takween's typed config schema with an OS profile.
6. Implement a pilot mixed build (one pure Baa module linked with the existing kernel).
