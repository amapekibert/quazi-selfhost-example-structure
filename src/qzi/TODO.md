# qzi — Implementation TODO

> **Package:** `quazi_qzi`  
> **Source:** `src/qzi/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Own portable QZI format/schema, instruction model, binary encoder/decoder, verification, metadata, public interfaces and symbolic linking. No source syntax or target-specific code.

**Integration boundary:** chunks → QZI v14 bytes → Rust bootstrap backend (P0); decoder/verifier → Quazi backend (P1). QZI is portable and target independent.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/bytecode/chunk.rs`
- `src/bytecode/instruction.rs`
- `src/bytecode/opcode.rs`
- `src/bytecode/interface.rs`
- `src/bytecode/vtable.rs`
- `src/abi.rs`
- `src/bytecode/fixtures/qzi/README.md`
- `docs/LIBRARIES.md`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Define opcode IDs and all basic operand layouts from bootstrap, preserving the six-byte `[opcode, 4 operands, flags]` instruction (`src/instructions.qz`).
- [ ] Model QZI chunks: names, param count, register count, flags, constants and instruction vectors (`src/functions.qz`).
- [ ] Model required constant pool entries and width-checked references (`src/constants.qz`).
- [ ] Implement v14 QZI magic `\0QZI`, version `0x0E`, section directory, header and bounded section offsets (`src/header.qz`, `src/format.qz`).
- [ ] Serialize executable metadata: package name/version, kind and whether main takes args (`src/metadata.qz`).
- [ ] Serialize bytecode chunk stream with the current format, including params, regs, instructions and constant indexes (`src/encoder.qz`).
- [ ] Encode source-visible interface and call-relocation sections with valid empty/minimal values for the P0 executable subset.
- [ ] Emit mandatory v14 ownership artifact as explicit content-bound **Unverified**, including section digests; never invent a verified proof.
- [ ] Guarantee that encoded bit flags preserve signed/unsigned arithmetic, float operations, memory width and explicit `Lea` block metadata.
- [ ] Validate register numbers, branch destinations, const pool bounds, chunk indices, opcode flags and section boundaries (`src/verifier.qz`).
- [ ] Reject malformed/overflowing UTF-8 strings, missing sections, unknown opcodes and unsupported semantic claims before emitting files.
- [ ] Expose a single API turning `chunks` + metadata into a valid portable QZI file (`src/builder.qz`, `src/lib.qz`).
- [ ] Test exact v14 encoding via Rust `deserialize_qzi_module()` followed by Rust native compilation and actual execution.
- [ ] Golden-test deterministic serialization for representative functions, constants, branches, imports and owned/static values.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Implement strict current-v14 decoder for self-hosted native codegen, verifying metadata, sections, lengths and ownership envelope (`src/decoder.qz`).
- [ ] Reconstruct validated chunk/function/constant tables and symbolic names without reference to source files.
- [ ] Support executable and library module kinds as required to compile multi-package self-host sources.
- [ ] Implement symbolic call relocations and name resolution without conflicting definitions (`src/symbols.qz`, `src/imports.qz`).
- [ ] Preserve ordered dynamic-vtable layout descriptors and owner ABI generation in QZI-only native inputs.
- [ ] Export public source interfaces needed for self-hosted package imports, keeping implementation functions private (`src/exports.qz`).
- [ ] Reject unverified or legacy ownership ABI uses that would cross unsafe borrow boundaries; do not treat a digest as proof.
- [ ] Implement basic QZI linking of separately built packages and equivalent shared chunks needed by the compiler build.
- [ ] Verify same resulting native program using the bootstrap and Quazi backends from the same QZI input.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Match all current QZI v14 constant tags: Int, Float, Str, Bytes, FnAddr, ForeignSymbol, ForeignGlobal and VtableLayout.
- [ ] Complete portable ABI signature/foreign-global descriptions, exported C adapters, public API rendering and interface import validation.
- [ ] Implement all current flags and opcodes, including signed subword widths, float f32 conversions, power, CallCReg, syscalls, intrinsics and traps.
- [ ] Read historically compatible v2–v13 QZI fixtures, including version-specific chunk flag/layout differences, without accepting incompatible ownership assumptions.
- [ ] Reject legacy Pow, implicit `Lea`, stale reference/aggregate ABI, ambiguous v6 generics and legacy owned closure artifacts with actionable rebuild errors.
- [ ] Complete symbol namespace aliases, library root reachability, public interface pruning, symbolic relocation validation and duplicate chunk detection.
- [ ] Implement strict size/depth/budget validation against truncated, hostile and integer-overflowing input files.
- [ ] Preserve original ownership ABI generation and v14 unverified envelope across linking, aliases, serialization and library re-emission.
- [ ] Compare emitted bytes/decoded AST of QZI with Rust codec and fixtures across all supported format eras.
- [ ] Harden artifact format fuzzing; prove codec round-trip and deterministic reserialization for eligible current artifacts.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Specify a new QZI version only for justified new semantics; include migration, validation and backward-compatibility policy.
- [ ] Consider verified ownership/effect certificates and stable portable generic-template ABI only with formal trust boundaries.
- [ ] Expand reusable QZI linking/inspection APIs for third-party backends and advanced toolchains.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/builder.qz` | Assemble QZI sections from prevalidated chunks and metadata; do not generate language-level instructions. |
| `src/constants.qz` | Portable tagged QZI constant pool entries and encoding constraints. |
| `src/decoder.qz` | Deserialize QZI. |
| `src/encoder.qz` | Serialize QZI. |
| `src/exports.qz` | Exported symbols and metadata. |
| `src/format.qz` | QZI binary format contract. |
| `src/functions.qz` | Function table and bodies. |
| `src/header.qz` | QZI magic, format version and header. |
| `src/imports.qz` | QZI import symbols/interfaces, not source import resolution. |
| `src/instructions.qz` | Portable QZI instruction definitions. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/metadata.qz` | QZI module/package metadata and format-compatibility fields. |
| `src/symbols.qz` | Symbol identifiers and references. |
| `src/types.qz` | Type definitions and identity. |
| `src/verifier.qz` | Verify QZI format, control-flow and required safety invariants. |

## Repository boundary

- `qzi/` owns portable format, versioned codec, verifier and QZI library composition; never serializes ELF, COFF or PE.
- Loaded QZI is target-neutral input for `quazi/backend` and externally installed `targets/*`; ABI metadata is descriptive and must not turn QZI into a platform-native format.

## Completion / verification gates

- [ ] P0 QZI opens and compiles with the uploaded Rust bootstrap, not just with Quazi's own decoder.
- [ ] QZI owns byte layout and validation; chunks owns construction; backend owns native emission.
- [ ] Corruption in any mandatory section or content-bound digest fails closed.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
