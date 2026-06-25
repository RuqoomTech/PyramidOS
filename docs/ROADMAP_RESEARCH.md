# PyramidOS Research Track — Post-Unix, Arabic-First, AI-Assisted

> **Status:** Research backlog, not an implementation sprint.
>
> This file captures the long-term direction inspired by the “AI can revive
> OSDev” discussion. It exists so radical ideas have a home without hijacking
> the current v0.9 stabilization work.

## 0. Position

PyramidOS should not be an anti-Unix reaction project. It should be
**post-Unix aware**:

- use proven Unix-like ideas where they are useful for bootstrapping;
- avoid treating POSIX, byte streams, hierarchical files, ambient authority,
  and shell pipelines as the final shape of computing;
- make Arabic-first UX and sovereign system design first-class constraints;
- use AI as leverage for research, review, documentation, and prototypes, not
  as an excuse to skip engineering discipline.

## 1. Non-Negotiable Rule

Research ideas may influence architecture, but they must not block the
foundation gates.

```text
v0.9 must still finish boot/memory/storage/VFS correctness.
v0.10+ may prepare abstractions that leave room for post-Unix designs.
Moonshot research must stay behind gates until the kernel can prove basics.
```

## 2. Why This Track Exists

Most hobby kernels follow the same path:

```text
bootloader -> PMM -> VMM -> scheduler -> VFS -> ELF -> syscalls -> POSIX-ish shell
```

That path is useful for learning, but it usually creates a toy Linux. PyramidOS
can use pieces of that path as scaffolding without accepting the whole worldview.

The research track asks:

> What would an Arabic-first operating system look like if it did not inherit
> the mental furniture of the 1970s as destiny?

## 3. Historical Systems Worth Studying

These systems are not templates to copy. They are proof that serious alternatives
to mainstream Unix/Windows categories have existed.

| System / line | What to study | Relevance to PyramidOS |
|---|---|---|
| Plan 9 | Namespaces, distributed resources, 9P | Good example of rethinking system composition without abandoning simplicity. |
| Oberon | Whole-system coherence, language + OS + UI | Shows the power of small integrated environments. |
| KeyKOS / EROS / CapROS | Object capabilities, persistence | Directly relevant to authority, security, and persistent objects. |
| Singularity | Language-based isolation, contracts, manifests | Useful for thinking about safe components and non-POSIX app models. |
| seL4 | Formal verification discipline | A reminder that kernel correctness needs proof-like thinking, not vibes. |
| Smalltalk/Lisp machines | Live objects and inspectable systems | Relevant to the “living environment” idea. |

## 4. Research Themes

### R1 — Capability-Based Authority

Replace ambient global access with explicit authority tokens/capabilities.

Early experiments:

- capability object model document;
- capability table sketch for future processes;
- compare file-descriptor style handles vs true capabilities;
- define how `/dev`, PyFS, and future IPC would expose authority.

Do not implement in v0.9.

### R2 — Persistent Object Store

Explore storage where structured objects are first-class, not merely bytes in
paths.

Early experiments:

- object identity format;
- persistent object reference format;
- relation to PyFS and PyDB;
- recovery model after crash;
- export/import story for interoperability.

Do not replace PyFS now. PyFS still needs a boring read-only v0.9 reality check.

### R3 — Structured IPC / Component Model

Avoid making text serialization the universal interface.

Early experiments:

- typed message envelope;
- zero-copy buffer passing policy;
- schema/versioning rules;
- capability-gated communication;
- Arabic-first diagnostic representation for messages.

### R4 — Live Inspectable System

Make debugging, editing, and system observation a first-class experience.

Early experiments:

- kernel object registry design;
- `inspect` command concept;
- `dmesg` + structured log records;
- exposing system state without corrupting it.

### R5 — Arabic-First Interaction Model

Arabic should not be a translation layer on top of English assumptions.

Early experiments:

- Arabic command grammar;
- RTL shell prompt and cursor behavior;
- Arabic-first errors with English debug fallback;
- glossary for kernel/system terms.

### R6 — Non-POSIX Application Model

PXF should not become ELF with a different magic number.

Early experiments:

- record-based executable semantics;
- manifest-driven permissions;
- no implicit global filesystem access;
- structured imports/capabilities rather than blind dynamic linking.

### R7 — AI-Assisted OSDev Workflow

AI is allowed as a collaborator, not an authority.

Good uses:

- summarize hardware manuals and papers;
- generate throwaway prototypes;
- create QEMU harnesses and negative tests;
- audit invariants and documentation;
- compare architecture options.

Forbidden/unsafe uses without human review:

- merging generated paging/interrupt/disk-write code blindly;
- trusting AI for ABI, calling convention, or memory-order details;
- accepting security-boundary designs without tests and manual reasoning.

## 5. Research Gate Model

A research idea can move toward implementation only if it has:

1. a short design note;
2. a failure model;
3. test or emulator strategy;
4. interaction with current roadmap gates;
5. a rollback/deferral plan;
6. a reason it belongs in PyramidOS rather than in a prototype only.

## 6. Near-Term Allowed Work

Before v0.9 exits, allowed research work is documentation-only:

- collect papers and notes;
- write design sketches;
- define vocabulary;
- compare alternatives;
- update roadmap constraints.

No kernel implementation should be diverted from v0.9 for these ideas.

## 7. First Serious Research Milestone

After v0.9:

```text
R0: System Model Notebook
```

Deliverables:

- one document defining PyramidOS “program”, “object”, “authority”, “storage”,
  “message”, and “identity”;
- one small simulator in user-space if useful;
- no kernel rewrite.

The goal is to protect imagination without damaging engineering focus.
