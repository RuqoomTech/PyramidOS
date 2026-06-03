# Documentation Changelog — 2026-06-03

## Added

- `docs/TECHNICAL_REVIEW_2026-06-03.md`  
  Honest technical review covering strengths, risks, and next milestone.

- `docs/BOOT_MEMORY_LAYOUT.md`  
  Current boot/memory map, fixed-address risks, reserved ranges, linker-symbol
  recommendation, page-rounding policy, and build-time layout guards.

- `docs/V0_9_STABILIZATION_PLAN.md`  
  Concrete v0.9 plan focused on boot/memory correctness, PyFS/VFS reality,
  terminal scrolling, shell maintainability, and smoke tests.

- `docs/SMOKE_TESTS.md`  
  Manual build, boot, core runtime, storage/VFS, and negative test checklist.

- `docs/PROJECT_POSITIONING.md`  
  Honest description of what PyramidOS is today and what it should not yet claim
  to be.

## Updated

- `README.md`
  - Reframed PyramidOS as an experimental Arabic-first kernel/research OS.
  - Added project-stage warning: not a production desktop OS.
  - Replaced several overconfident “Stable” labels with “Working”, “Foundation”,
    or “Needs Hardening”.
  - Added engineering review notes and links to the new docs.
  - Corrected the architecture overview to state that `0x10000` is currently
    64 KiB.
  - Updated roadmap snapshot toward v0.9 stabilization.

- `docs/SETUP.md`
  - Clarified that native `gcc -m32` fallback is only an early convenience.
  - Promoted `i686-elf` cross-compiler as the recommended serious development
    path.
  - Added toolchain verification with `i686-elf-gcc -dumpmachine`.
  - Added required reading before kernel growth.

- `docs/ROADMAP_L3_TACTICAL.md`
  - Replaced the old completed v0.8.1 sprint with the new v0.9 tactical roadmap.
  - Added P0/P1/P2 tasks and acceptance criteria.
  - Explicitly deferred GUI/networking/audio/Baa/Takween/AI work.

- `AGENTS.md`
  - Added current engineering priorities.
  - Pointed agents to smoke tests and boot-memory docs.
  - Clarified that archives must exclude `.git/`, `.vs/`, `build/`, and generated
    images.

- `CONTRIBUTING.md`
  - Added documentation-update requirements for risky kernel areas.
  - Added `kernel/fs/` to project layout rules.
  - Added positioning guidance to avoid production-ready claims.

- `kernel/arch/i386/linker.ld`
  - Corrected the misleading comment that called `0x10000` 1 MiB.
  - No behavior was changed.

## Not Changed

- No kernel logic was modified.
- No Makefile build behavior was changed.
- No storage/VFS implementation was changed.
- No bootloader behavior was changed.

This pass is documentation-only except for correcting a source-code comment in
`linker.ld`.
