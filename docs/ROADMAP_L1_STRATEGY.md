# Layer 1: Strategic Roadmap (PyramidOS)

**Vision:** A sovereign Arabic-first operating-system project engineered from
scratch. The near-term kernel is a simple monolithic C/Assembly system for
learning, control, and momentum; the long-term product should be post-Unix aware,
capability-conscious, and willing to rethink storage, authority, applications,
and Arabic-first interaction instead of becoming a toy Linux.

> **Legend:**
> ✅ = Completed | 🚧 = In Progress | 📅 = Planned | 🔮 = Long Term Vision

## Strategic Rule

PyramidOS may use Unix-like pieces as scaffolding, but it must not accept POSIX,
byte streams, hierarchical files, ambient authority, or ELF-like executable
semantics as destiny. Radical ideas belong in the research track until the
foundation gates in `ROADMAP.md` pass.

---

## 0. 🇸🇦 Arabic-First Initiative (End-to-End)

**Goal:** Ship Arabic-first UX from power-on (bootloader) to desktop. English remains available as an optional debug fallback.

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Arabic Boot Experience (Stage 2)** | 📅 | Arabic splash/menu/messages in graphics mode (Mode 13h now, VBE later); must stay within Stage 2 size constraints and keep a reliable fallback path. |
| **Unicode Strategy (UTF-8 Everywhere)** | 📅 | Standardize on UTF-8 for kernel/userland/VFS; add safe UTF-8 primitives (decode/iterate/width). |
| **Framebuffer Terminal (Arabic Text Stack)** | 📅 | Primary console becomes framebuffer-rendered text with Arabic shaping + RTL; VGA text mode becomes compat/debug only. |
| **Arabic Shaping (v1)** | 📅 | Contextual forms + Lam-Alef; diacritics support staged. |
| **Bidi (v1)** | 📅 | Start with Arabic-only RTL lines, then support mixed Arabic+Latin+numbers runs with sane cursor behavior. |
| **Arabic Keyboard Layout** | 📅 | Scancode -> Unicode Arabic mapping, layout toggle, digit mode (Arabic-Indic vs Western). |
| **Arabic KShell & Base Tools** | 📅 | Commands, help, and system messages Arabic-first; keep English aliases for development. |
| **UTF-8 Filenames in VFS/PyFS** | 📅 | End-to-end UTF-8 path handling, directory entries, and a policy for normalization and sorting. |
| **GUI RTL + Arabic Text** | 📅 | GUI toolkit supports RTL layout mirroring and shaped/bidi text rendering everywhere. |
| **Baa + Takween Integration (Arabic-first DX)** | 📅 | Build PyramidOS with Baa/Takween and ship Arabic-first developer tooling and project schemas. |

**Detailed plan:** `docs/ROADMAP_ARABIC_FIRST.md`  
**Scheduling rule:** Arabic-first remains the identity, but advanced shaping/bidi/framebuffer work starts only after the v0.9 storage/memory gates in `docs/ROADMAP.md` pass.

---

## 0.5. 🧪 Post-Unix Research Initiative

**Goal:** keep the project imaginative without damaging engineering focus.

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Research Track Document** | ✅ | `ROADMAP_RESEARCH.md` captures post-Unix, AI-assisted, capability/object/persistence experiments. |
| **AI-Assisted OSDev Policy** | 📅 | Define where AI is allowed: docs, comparison, prototypes, tests, audits; generated kernel code remains untrusted. |
| **Capability Authority Model** | 🔮 | Explore explicit authority handles/capabilities instead of ambient global access. |
| **Persistent Object Store** | 🔮 | Explore structured persistent objects beside/above PyFS, not as a v0.9 replacement. |
| **Structured IPC / Component Model** | 🔮 | Typed messages and manifests instead of byte-stream-only glue. |
| **System Model Notebook (R0)** | 📅 | After v0.9, define program, object, authority, storage, identity, and message before implementation. |

**Scheduling rule:** research can influence design, but implementation waits
until current lower-level gates pass.
## 1. 🛤️ Milestone 1: The Bootloader (PyramidBL)

**Goal:** Reliably load the kernel payload into memory and transition the CPU to a usable state.

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Legacy BIOS (Stage 1)** | ✅ | MBR, 512-byte limit, CHS/LBA disk reading. |
| **Legacy BIOS (Stage 2)** | ✅ | A20 enable, E820 memory map, Kernel Header parsing. |
| **Protected Mode Setup** | ✅ | GDT setup, 32-bit transition, jump to Kernel Entry. |
| **Bootloader UX (Tier 1: Text Mode Polish)** | ✅ | Stage 2 branded screen, consistent status lines, progress bar, boot menu; Stage 1 remains minimal for stability. |
| **Bootloader UX (Tier 2: Mode 13h Splash)** | ✅ | BIOS Mode 13h (320x200x256) splash + progress bar; hard fallback to text mode. |
| **Bootloader UX (Tier 3: VBE Splash / Installer UI)** | 🔮 | VBE high-res LFB splash and future “Blue Crystal” installer UI; robust fallback path required. |
| **UEFI Support** | 📅 | Modern UEFI bootloader (EDK2) loading from ESP. |
| **Multiboot Compliance** | 🔮 | Standard header for compatibility with GRUB/QEMU. |

---

## 2. 🧠 Milestone 2: Kernel Core Foundation

**Goal:** Establish control over the hardware resources (CPU, RAM, Interrupts).

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Kernel Entry** | ✅ | Stack setup, Environment cleanup, C Runtime handoff. |
| **Physical Memory (PMM)** | ⚠️ | Bitmap allocator and E820 parsing exist; v0.9 must harden reserved ranges and bitmap placement. |
| **Interrupts (IDT)** | ✅ | Exception handling (Page Faults, Div-by-zero) & Hardware IRQs. |
| **Virtual Memory (VMM)** | ⚠️ | Paging enabled with early mappings; higher-half/user isolation is planned, not complete. |
| **Hardware Interrupts** | ✅ | 8259 PIC Remapping, IRQ Masking/Unmasking. |
| **Kernel Heap** | ✅ | Dynamic memory (`kmalloc`/`kfree`) with coalescing and safety checks. |
| **Multitasking** | 📅 | Custom Process Control Blocks (PCB), Round-Robin Scheduler. |

---

## 3. ⌨️ Milestone 3: Interaction & Drivers (HAL)

**Goal:** Allow the user to interact with the system and persist data.

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Keyboard Driver** | ✅ | Scancode translation, Circular Buffer, Shift/Caps support. |
| **Text Shell (KShell)** | ✅ | Interactive CLI, Command parsing, History (Basic). |
| **Diagnostics (Selftest)** | ✅ | Boot-time + on-demand kernel diagnostics via KShell `diagnose` (PMM/Heap/ATA). |
| **RTC/CMOS** | ✅ | Real-Time Clock driver for system Date/Time. |
| **System Timer** | ✅ | PIT Driver (100Hz) for uptime and sleep (idle uses STI+HLT via `cpu_idle()`). |
| **Storage Drivers** | 🚧 | ATA/PIO driver (Read-Only): LBA28 PIO reads + IDENTIFY detection + safer polling/timeouts. |
| **Filesystem (VFS)** | 🚧 | VFS foundation: mount table + file descriptors + DevFS (`/dev`) exposure. |
| **Partition Discovery (MBR)** | 🚧 | Register `disk0p1..disk0p4` block devices for filesystem mounting. |
| **PyFS (Read-Only)** | 🚧 | Probe + mount PyFS on a partition device (e.g., `disk0p1`) before full RW+journaling. |
| **HAL Boundary Enforcement** | 📅 | Formalize stable HAL boundaries so Drivers don’t depend on Arch specifics (port I/O, IRQ wiring, paging internals). |

---

## 4. 📦 Milestone 4: Userland & Protected Execution

**Goal:** Securely execute separate programs in Ring 3 with full isolation.

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Higher-Half Kernel** | 📅 | **[CRITICAL]** Remap Kernel to `0xC0000000` to free lower memory for apps. |
| **Task State (TSS)** | 📅 | **[CRITICAL]** Hardware structure for Ring 3 -> Ring 0 stack switching. |
| **User Mode (Ring 3)** | 📅 | GDT User Segments, entering User Mode via `IRET`. |
| **System Calls** | 📅 | Custom `INT 0x80` or `SYSENTER` API interface. |
| **PXF Loader** | 📅 | **Pyramid Executable Format**. A custom binary format parser. |
| **Config Database** | 📅 | A custom hierarchical binary configuration store. |

---

## 5. 🖥️ Milestone 5: The Graphical User Interface

**Goal:** A unique desktop environment inspired by the "Classic" 95 feel.

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Video Driver** | 📅 | VESA BIOS Extensions (VBE) linear framebuffer. |
| **Graphics Engine** | 📅 | Custom 2D drawing primitives (Line, Rect, Blit). |
| **Window Manager** | 📅 | Custom Compositor, Z-Ordering, Message Passing. |
| **Widget Toolkit** | 📅 | Custom UI Controls (Buttons, Windows, Taskbar). |
| **Desktop Shell** | 📅 | Icons, Wallpaper, Start Menu (Pyramid Style). |

---

## 6. 🌐 Milestone 6: Advanced Features

**Goal:** Connectivity and Optimization.

| Feature | Status | Description |
| :--- | :---: | :--- |
| **Networking** | 🔮 | Network Card Drivers, Custom TCP/IP Stack. |
| **Audio** | 🔮 | AC97 or SoundBlaster drivers. |
| **Pyramid Component Model**| 🔮 | Custom IPC system for object embedding (Replacing OLE/COM). |
| **SMP** | 🔮 | Multi-core support (APIC). |
