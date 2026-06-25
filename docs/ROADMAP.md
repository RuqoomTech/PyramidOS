# PyramidOS Roadmap

> **Single source of truth for planning PyramidOS.**
>
> This roadmap merges the project positioning, technical review findings, boot
> and memory-layout notes, v0.9 stabilization plan, and smoke-test expectations
> into one place. The old review docs are archived under
> `docs/archive/2026-06-03-review/` for history only.

## 0. Project Positioning

PyramidOS is an experimental Arabic-first x86 operating-system project. Its
current value is as a serious systems-programming platform and as a distinctive
Arabic-first kernel project, not as a production-ready Windows/Linux replacement.

### What PyramidOS is now

- A custom legacy BIOS boot path with Stage 1 and Stage 2.
- A 32-bit protected-mode monolithic kernel.
- Early memory management: PMM, VMM, heap, identity paging, high virtual heap.
- Hardware bring-up: IDT/ISR/IRQ, PIC, PIT, RTC, PS/2 keyboard, VGA terminal.
- KShell for diagnostics and manual testing.
- Early storage: ATA PIO read-only path, block-device registry, MBR partition
  discovery, DevFS, VFS foundation, and PyFS superblock probing.
- Arabic-first identity work in the bootloader and long-term text/UI roadmap.

### What PyramidOS is not yet

- Not a daily-driver OS.
- Not a desktop OS yet.
- Not ready for GUI, networking, package management, or app ecosystem work.
- Not ready for Baa/Takween migration beyond planning and tiny mixed-build
  experiments.

### Product truth

The strongest identity is:

> **Arabic-first low-level operating-system experience from bootloader to shell
> to future desktop.**

The main engineering rule is:

> **Do not add impressive features before boring correctness gates pass.**

The long-term research stance is:

> **PyramidOS is not anti-Unix. It is post-Unix aware.**

That means Unix-like scaffolding is acceptable during bring-up, but POSIX,
byte-stream-only IPC, hierarchical paths, ambient global authority, and
shell-as-system-interface should not become destiny by accident.

AI is allowed as a force multiplier for research, documentation, prototype
generation, test harnesses, and review. AI-generated low-level code must be
treated as untrusted until it is manually reviewed and tested.

---

## 1. Roadmap Layers

| Layer | File | Purpose | Status |
|---|---|---|---|
| Master roadmap | `docs/ROADMAP.md` | Planning source of truth, release gates, risk register | Active |
| Strategic roadmap | `docs/ROADMAP_L1_STRATEGY.md` | Long-range product/kernel milestones | Active, high-level |
| Architecture roadmap | `docs/ROADMAP_L2_ARCH.md` | Subsystem architecture and design direction | Active, design-level |
| Tactical roadmap | `docs/ROADMAP_L3_TACTICAL.md` | Current sprint implementation checklist | Active, short-term |
| Future concepts | `docs/ROADMAP_L4_FUTURE.md` | Moonshot backlog, not scheduled | Reference only |
| Zenith concepts | `docs/ROADMAP_L5_ZENITH.md` | Very long-term innovation ideas | Reference only |
| Arabic-first roadmap | `docs/ROADMAP_ARABIC_FIRST.md` | Arabic/RTL/UTF-8/text/UX track | Active, staged |
| Baa/Takween plan | `docs/BAA_TAKWEEN_OS_INTEGRATION_PLAN.md` | Future compiler/build-system integration | Deferred until gates pass |
| Research track | `docs/ROADMAP_RESEARCH.md` | Post-Unix, AI-assisted, capability/object/persistence experiments | Research only |

**Rule:** Detailed implementation tasks belong in Layer 3. High-level ambition
belongs in Layers 1, 4, and 5. Cross-cutting policies and gates belong here.

---

## 2. Current Capability Snapshot

| Component | Status | Honest description |
|---|---:|---|
| Bootloader | ✅ Working | Custom legacy BIOS Stage 1/2 loads kernel and switches to protected mode. |
| Arabic boot UX | ✅ Early success | Mode 13h Arabic splash/menu/fatal screen exists; keep fallback reliable. |
| Kernel entry | ✅ Working | Stack setup and C handoff exist. |
| IDT/IRQ path | ✅ Working | Exceptions and hardware IRQs are wired. |
| PMM | ⚠️ Working, risky | Bitmap allocator exists, but fixed metadata placement must be hardened. |
| VMM | ⚠️ Early | Paging is enabled; higher-half/user isolation is not complete. |
| Heap | ✅ Working | Kernel heap exists with coalescing and checks. |
| Terminal | ⚠️ Usable, limited | VGA text mode works but needs scrolling before verbose diagnostics grow. |
| KShell | ⚠️ Usable, needs structure | Good bring-up shell; should move to a command table. |
| ATA/PIO | 🚧 Early read-only | Useful for testing; not production storage. |
| Block layer | ✅ Foundation | Generic block registry exists. |
| MBR partitions | 🚧 Early | Partition block devices register from MBR. |
| DevFS | ✅ Foundation | `/dev` exposes virtual devices. |
| VFS | 🚧 Foundation | Mount/FD layer exists but needs real path traversal and user-facing commands. |
| PyFS | 🚧 Probe only | Superblock probe exists; directory/file reads are the next reality test. |
| Tests | ⚠️ Manual | Selftests exist; QEMU smoke-test automation is still needed. |

---

## 3. Risk Register

These risks block new feature expansion until resolved.

| Risk | Priority | Roadmap gate | Decision / required action |
|---|---:|---|---|
| Kernel load address is `0x10000`, not 1 MiB | P0 | v0.9 | Keep temporarily and document honestly, or move all assumptions to `0x100000`. |
| Fixed PMM bitmap at `0x20000` can overlap a growing kernel | P0 | v0.9 | Use linker symbols and place/reserve bitmap safely. |
| Disk image layout has weak build-time guards | P0 | v0.9 | Make invalid image layouts fail at build time. |
| BootInfo assembly writes must match C struct exactly | P0 | v0.9 | Zero BootInfo and write `mmap_count` as a 32-bit field. |
| E820 page rounding may free partial boundary pages | P0 | v0.9 | Free usable pages conservatively; reserve used ranges expansively. |
| Shell command chain will become fragile | P1 | v0.9 | Convert to command table and generate help from metadata. |
| Terminal clears instead of scrolling | P1 | v0.9 | Add scroll-up before verbose storage commands. |
| Version strings are inconsistent | P2 | v0.9 | Define one `PYRAMIDOS_VERSION`. |
| Release archives include `.git/` and `.vs/` | P2 | v0.9 | Add clean export/archive target and stop shipping editor/private data. |
| Vision docs can outrun engineering reality | Ongoing | Every release | Every release must have gates and explicit deferrals. |
| Post-Unix ideas can derail v0.9 | Ongoing | v0.9 | Research may be documented, but implementation waits until foundation gates pass. |
| AI-generated kernel code can look correct while being unsafe | Ongoing | Every release | Treat AI output as untrusted; require manual review, tests, and documented invariants. |

---

## 4. Release Gates

A milestone is not complete because the feature boots once. A milestone is
complete only when the gate passes.

### Gate A — Build and image correctness

- Stage 1 must be exactly 512 bytes.
- Stage 2 must fit within `STAGE2_SECTORS * 512`.
- Stage 2 disk range must not overlap the kernel disk range.
- Kernel image must fit in the disk image.
- Build output must print Stage 2 size, kernel size, and disk sector ranges.
- `make` must fail before QEMU if the disk image would be invalid.

### Gate B — Boot memory correctness

- Linker exports `kernel_start` and `kernel_end`.
- PMM reserves the exact loaded kernel range.
- PMM reserves BootInfo, E820 map, stack, page tables, and allocator metadata.
- PMM bitmap cannot silently overlap the kernel image.
- Low-memory reserved ranges are documented in this roadmap and in code comments.

### Gate C — Storage through VFS

- All storage shell commands go through VFS, not filesystem internals.
- `ls /dev` works through DevFS.
- `ls /py` works when PyFS is mounted.
- `cat /py/<file>` can read at least one small regular file.
- Invalid paths return readable errors instead of crashes or silent failure.

### Gate D — Manual smoke test

After `make clean && make && make run`, the following commands should be tested:

```text
help
mem
time
uptime
blkinfo
mounts
ls /dev
ls /py
cat /py/<known-small-file>
pyfs_sb
diagnose
```

Expected result: no panic, readable output, and diagnostics explain missing ATA or
missing PyFS media clearly.

### Gate E — Clean handoff

- Release/archive zip excludes `.git/`, `.vs/`, `build/`, generated disk images,
  editor caches, and local machine artifacts.
- README status matches runtime version.
- Deferred features are not described as current functionality.

---

## 5. Research Track — Beyond the Unix-Shaped Rut

**Status:** active thinking, not active implementation.

This track gives the project a place to explore radical design ideas without
polluting the current sprint. The detailed research backlog lives in
[`ROADMAP_RESEARCH.md`](ROADMAP_RESEARCH.md).

### Research commitments

- PyramidOS may use Unix-like mechanisms as temporary scaffolding.
- PyramidOS should avoid becoming POSIX-compatible by accident.
- PXF should stay Pyramid-native and record/manifest oriented, not “ELF with a
  different magic number.”
- Long-term storage should explore structured objects and metadata-rich views,
  not only byte streams in folders.
- Authority should eventually move toward explicit handles/capabilities instead
  of ambient global permissions.
- AI may help compare designs and generate prototypes, but it does not validate
  correctness.

### Research gates

A research idea may move from Layer 4/5 into Layer 2/3 only when it has:

| Gate | Requirement |
|---|---|
| Design note | Clear model, scope, and non-goals. |
| Failure model | What can corrupt memory, storage, authority, or UX? |
| Test strategy | Emulator/unit/simulator path before kernel merge. |
| Roadmap fit | Does not block lower-level gates. |
| Rollback path | Can be removed without destabilizing the kernel. |

### First post-v0.9 research deliverable

After v0.9 exits, create:

```text
R0 — System Model Notebook
```

It should define PyramidOS terms for program, object, authority, storage,
message, identity, and UI surface before implementation starts.

## 6. v0.9 — Storage + VFS Reality Check

**Theme:** make the current kernel honest, testable, and safe enough for the next
real layer.

**Do not include:** GUI, networking, audio, package manager, userland expansion,
full Arabic shaping/bidi console, Baa/Takween migration, local AI, or scheduler
expansion.

### v0.9 Scope

| Track | Priority | Tasks | Acceptance |
|---|---:|---|---|
| Build/image guards | P0 | Add `KERNEL_START_SECTOR`, stage-size checks, overlap checks, kernel-image capacity checks | Bad layouts fail during `make`. |
| Boot memory layout | P0 | Linker symbols, reserve actual kernel range, prove/move PMM bitmap, document fixed ranges | PMM never frees kernel/bitmap/boot structures. |
| BootInfo + E820 | P0 | Zero BootInfo, dword `mmap_count`, conservative page rounding, malformed-map diagnostics | C and assembly agree; boundary pages are safe. |
| Terminal | P1 | Scroll-up instead of clear-on-bottom | Long diagnostics remain visible. |
| KShell | P1 | Command table, generated help, prepare Arabic aliases | Commands are easy to add safely. |
| PyFS read path | P1 | Validate superblock, list root dir, open/read one small file | PyFS becomes more than a probe. |
| VFS commands | P1 | `ls <path>`, `cat <path>`, readable errors | Storage is tested through VFS. |
| Hygiene/tests | P2 | Smoke-test checklist/script, clean archive/export, version unification | Project can be reviewed repeatably. |

### v0.9 Exit Criteria

PyramidOS can boot in QEMU, list `/dev`, mount/probe PyFS, list `/py`, read a
small file through VFS, run diagnostics, and ship as a clean source archive.

---

## 7. v0.10 — Kernel Structure and Userland Preparation

**Theme:** prepare for real processes without rushing into a desktop.

Suggested scope:

- System call design document and first syscall table.
- User/kernel address-space plan.
- TSS setup plan for Ring 3.
- File descriptor model hardened around VFS.
- Kernel log buffer (`dmesg`) so diagnostics are not tied to the screen.
- Init process design, even if the first `init` is tiny.
- Start separating `libk` from future userland libc assumptions.

Exit criteria: PyramidOS has a credible path to Ring 3 without rewriting VFS,
logging, or process abstractions again.

---

## 8. v0.11 — Minimal Ring 3 Experiment

**Theme:** prove the kernel can run one user process safely.

Suggested scope:

- GDT/TSS updates for Ring 3.
- Minimal syscall ABI.
- One tiny user program loaded from an initrd or PyFS.
- User process can write to console via syscall, not direct VGA memory.
- Kernel handles invalid syscall or bad pointer without dying.

Exit criteria: one user-mode process runs, exits or loops safely, and cannot
write directly to kernel memory.

---

## 9. v0.12 — Arabic Console Foundation

**Theme:** begin the Arabic-first kernel UX properly, after storage and basic
process direction are stable.

Suggested scope:

- UTF-8 decode/iterate primitives.
- Arabic keyboard layout mapping and layout toggle.
- Framebuffer console design spike.
- Minimal shaping rules for Arabic letters and Lam-Alef.
- Arabic KShell aliases through the command table.

Exit criteria: the console architecture is ready for Arabic-first text without
corrupting UTF-8 or relying permanently on VGA text mode.

---

## 10. Explicit Deferrals

The following are exciting, but not scheduled until the lower gates pass:

- GUI desktop / compositor.
- Networking stack.
- Audio stack.
- Package manager.
- Full PyFS write support and journaling.
- Baa/Takween mixed-kernel migration beyond tiny experiments.
- Local AI / fuzzy command execution.
- Capability-based security model implementation.
- Persistent object store replacing the normal filesystem.
- Structured IPC/component runtime beyond design notes.
- Mesh networking, hibernation snapshots, vector UI, disposable runtimes.

These belong in Layer 4/5 until the kernel has storage, VFS, basic userland,
and test discipline.

---

## 11. Decision Log

| Date | Decision | Reason |
|---|---|---|
| 2026-06-03 | Keep PyramidOS positioned as experimental/research OS for now | Honest positioning prevents roadmap fantasy. |
| 2026-06-03 | Merge review findings into the roadmap | The roadmap should be the source of truth. |
| 2026-06-03 | Make v0.9 a stabilization/storage milestone | Current risks are boot/memory/storage correctness, not missing GUI. |
| 2026-06-03 | Defer Baa/Takween integration until after v0.9 gates | Build/kernel assumptions must be stable before migration. |
| 2026-06-03 | Keep Arabic-first as core identity, but stage the hard text engine later | Arabic is the differentiator, but shaping/bidi needs framebuffer/UTF-8 groundwork. |
| 2026-06-25 | Add a post-Unix research track | PyramidOS should not become a toy Linux; radical ideas belong in a controlled research lane. |
| 2026-06-25 | Allow AI-assisted OSDev with strict review | AI helps explore design space, but generated kernel code is untrusted until reviewed and tested. |

---

## 12. Maintainer Rules

When changing kernel architecture, update this roadmap in the same commit.

Mandatory roadmap updates:

- boot address or disk sector layout changes;
- PMM/VMM reserved region changes;
- VFS/PyFS behavior changes;
- shell command additions/removals;
- release version changes;
- anything moved between active scope and deferred scope.

A task is not complete until code, tests/checklist, and roadmap status agree.
