# PyramidOS

> **Kernel Version:** v0.8.1 (Hardened docs pass)
> **Architecture:** x86 (32-bit Protected Mode)  
> **Boot Standard:** Legacy BIOS (Custom Bootloader)
> **Project Stage:** Experimental kernel / research OS, not a production desktop OS

PyramidOS is an experimental Arabic-first monolithic kernel written from scratch in C and Assembly. It features a custom multi-stage BIOS bootloader, protected-mode kernel bring-up, memory-management foundations, hardware drivers, a kernel shell, and early storage/VFS work.

The project should currently be understood as a serious systems-programming and OS-research project, **not** as a production-ready Windows/Linux replacement.

---

## 🚀 Current Status

The system boots into a **Protected Mode Shell** with memory management, hardware interrupts, and timekeeping capabilities.

| Component | Status | Description |
|-----------|--------|-------------|
| **Bootloader (Stage 1/2)** | ✅ Working | MBR, A20 Enable, E820 Map, Kernel Header Parsing, PM Switch. |
| **Kernel Entry** | ✅ Working | Stack setup, GDT, IDT (Exception Handling), ISR Stubs. |
| **Memory (PMM/VMM)** | ⚠️ Working, Needs Hardening | Bitmap Allocator, Paging Enabled (Identity Mapped). |
| **PIC Driver** | ✅ Working | 8259 PIC Remapped to vectors 32-47. |
| **Keyboard Driver** | ✅ Working | Scancode Set 1 translation, Shift/Caps state, Circular Input Buffer. |
| **System Timer (PIT)** | ✅ Working | 8253 PIT configured at 100Hz for system ticks and sleep. |
| **Real-Time Clock (RTC)** | ✅ Working | CMOS register parsing for Wall Clock Time (Y/M/D H:M:S). |
| **KShell** | ✅ Working | Interactive command interpreter with history and backspace support. |
| **Terminal (VGA Text Mode)** | ✅ Working, Needs Scrolling | Text Mode (80x25) with hardware cursor support. |
| **CPU Idle / Power Management** | ✅ Working | Uses STI+HLT (`cpu_idle()`) to avoid busy-waiting when idle. |
| **Kernel Heap** | ✅ Working | Doubly-linked list allocator with `kmalloc`/`kfree` and coalescing. |
| **VMM** | ⚠️ Working, Early Design | Paging enabled; Heap mapped to `0xD0000000`. |
| **Storage (ATA/PIO)** | 🚧 In Progress | LBA28 PIO reads (Read-Only) + IDENTIFY-based presence detection, stricter status checks. |
| **Block Layer (Registry)** | ✅ Working | Generic `BlockDevice` registry; ATA registered only when a real device is present (`disk0`, optional `disk1`). |
| **DevFS (/dev)** | ✅ Working | Virtual device filesystem exposing `/dev/disk0`, `/dev/disk1`, `/dev/null`, `/dev/zero`. |
| **Partition Discovery (MBR)** | ✅ Working | Parses MBR and registers `disk0p1..disk0p4` block devices (read-only). |
| **VFS (Foundation)** | 🚧 Foundation | Static mount table + FD table; `/` is `nullfs`, `/dev` is `devfs`. |
| **PyFS (Read-Only Bring-up)** | 🚧 In Progress | Probes `disk0p1` and mounts at `/py` if superblock is valid; exposes `/py/superblock` for verification. |

---

## ⚠️ Engineering Review Notes

A documentation review on 2026-06-03 identified several high-priority correctness risks that should be resolved before adding GUI, networking, multitasking expansion, or Baa/Takween migration work.

| Finding | Priority | Action |
|---|---:|---|
| Kernel load comment says 1 MiB, but `0x10000` is 64 KiB | P0 | Correct docs/comments or move kernel to `0x100000` consistently. |
| PMM bitmap is fixed at `0x20000` | P0 | Prove no overlap or place dynamically after `kernel_end`. |
| Stage 2/kernel disk layout lacks hard size guards | P0 | Add Makefile image-layout checks. |
| E820 usable ranges should be freed conservatively | P0 | Free only fully usable pages: start up, end down. |
| BootInfo field writes should match C layout exactly | P0 | Write `mmap_count` as a dword and zero the structure before use. |
| Shell and terminal need maintainability polish | P1 | Add command table and terminal scrolling. |

The review findings are now merged into the master roadmap so planning has one source of truth:

- [`docs/ROADMAP.md`](docs/ROADMAP.md) — master roadmap, risk register, release gates, and v0.9 scope.
- Archived review notes are kept under `docs/archive/2026-06-03-review/` for history only.

---

## 🛠️ Building and Running

### Prerequisites

* **System:** Linux, WSL2, or MacOS.
* **Toolchain:** `gcc`, `ld`, `make`, `nasm`.
* **Emulator:** `qemu-system-i386`.

### Quick Start

1. **Clean and Build (Release default):**

    ```bash
    make clean && make
    ```

    *Generates `build/pyramidos.img` and `build/kernel.map`.*

2. **Optional: Debug Build:**

    ```bash
    make clean && make debug
    ```

3. **Optional: Strict Warnings (Werror):**

    ```bash
    make clean && make STRICT=1
    ```

4. **Run:**

    ```bash
    make run
    ```

## 🔒 Repository Notice

This repository is currently private / all-rights-reserved. See `NOTICE`.

## 🧹 Project Hygiene

- Formatting baseline: `.editorconfig`
- Contribution guidance: `CONTRIBUTING.md`
- Ownership / review routing: `.github/CODEOWNERS`

---

## 💻 Kernel Shell Commands

Once booted, the **KShell** accepts the following commands:

* `help`    : List available commands.
* `clear`   : Clear the screen and reset cursor.
* `mem`     : Display Physical Memory stats (Total/Free RAM).
* `time`    : Display current Date and Time (from RTC).
* `uptime`  : Show system running time (ticks/seconds).
* `sleep`   : Pause execution for 1 second (Busy-wait test).
* `reboot`  : Restart the system (via Keyboard Controller).
* `diskread`: Read and hex-dump a disk sector by LBA (e.g., `diskread 0`, `diskread 60`).
* `blkinfo` : List registered block devices (includes `disk0p1` after MBR scan).
* `mounts`  : List VFS mounts (expects `/dev` + optional `/py`).
* `pyfs_sb` : Read `/py/superblock` via VFS (PyFS probe verification).
* `diagnose`: Run kernel diagnostics (PMM/Heap/ATA).
* `crash`   : Force a kernel crash (for testing the panic/exception path).

---

## 📂 Project Structure

```text
/
├── Makefile                  # Master build orchestration (release/debug/strict)
├── NOTICE                    # Private / all-rights-reserved notice (no license granted)
├── docs/                     # Strategic, Architectural, and Tactical roadmaps
├── boot/
│   └── src/legacy/           # 16-bit Assembly Bootloader (MBR + Loader)
└── kernel/
    ├── arch/
    │   └── i386/             # Architecture-specific code (x86)
    │       ├── entry.asm     # Kernel entry (stack + handoff to C)
    │       ├── idt.c/h       # Interrupt Descriptor Table
    │       ├── idt_asm.asm   # ISR/IRQ stubs
    │       └── cpu.h         # Registers + CPU helpers (cli/sti/hlt/idle)
    ├── core/                 # Kernel Core Logic
    │   ├── main.c            # Entry point / init ordering
    │   ├── pmm.c/h           # Physical Memory Manager
    │   ├── vmm.c/h           # Virtual Memory Manager
    │   ├── heap.c/h          # Kernel Heap Allocator
    │   ├── debug.c/h         # Panic system
    │   ├── shell.c/h         # KShell logic
    │   └── selftest.c/h      # Diagnostics
    ├── drivers/              # Hardware drivers
    │   ├── pic.c/h           # 8259 PIC (remap + mask control)
    │   ├── keyboard.c/h      # PS/2 keyboard (buffered input)
    │   ├── timer.c/h         # PIT driver
    │   ├── rtc.c/h           # RTC/CMOS wall-clock time
    │   ├── ata.c/h           # ATA PIO (read-only)
    │   └── terminal.c/h      # VGA text-mode terminal
    └── lib/                  # Freestanding libc-like helpers
        └── string.c/h        # Memory/string ops + atoi
```

---

## 🧠 Architecture Overview

1. **Boot Sequence:** BIOS -> MBR (Stage 1) -> Loader (Stage 2) -> Protected Mode -> Kernel (`0x10000`, currently 64 KiB).
2. **Initialization:**
    * **PMM:** Reads E820 map, initializes Bitmap at `0x20000`.
    * **IDT:** Sets up 256 interrupt vectors (Exceptions + IRQs).
    * **PIC:** Remaps IRQs to avoid CPU conflicts.
    * **VMM:** Identity maps lower 4MB, enables Paging (CR0).
    * **HAL:** Initializes Timer (100Hz) and Keyboard.
3. **Runtime:** The kernel yields control to `shell_run()`, which blocks on buffered keyboard input while the CPU idles via `cpu_idle()` (STI+HLT).

---

## 🔮 Roadmap Snapshot

* **Current:** Storage consumption bring-up: DevFS + MBR partitions + PyFS read-only probe.
* **Immediate hardening:** Boot/memory layout documentation, build image guards, PMM reserved-region correctness, BootInfo consistency.
* **Next milestone:** `v0.9 — Storage + VFS Reality Check`: PyFS real directory/file reads + VFS-backed shell commands (`ls`, `cat`, etc).

*Start with [`docs/ROADMAP.md`](docs/ROADMAP.md). The layered roadmap files remain as supporting strategy/architecture/future-reference documents.*
