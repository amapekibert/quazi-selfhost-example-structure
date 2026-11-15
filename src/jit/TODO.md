# jit — Implementation TODO

> **Package:** `quazi_jit`  
> **Source:** `src/jit/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Optional execution of already-compiled native images/QZI in process memory. It does not create chunks from Quazi source; JIT work is P3 per the global roadmap.

**Integration boundary:** P3 optional: QZI/verified native image + backend/linker → memory mapping/relocation/execute; no dependency from P0/P1 self-host on JIT.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/jit.rs`
- `src/backend/builtin_linker.rs`
- `src/backend/x86_64/mod.rs`
- `docs/LIBRARIES.md`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

**No required work for this package in this phase.** Keep this package out of the phase critical path.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

**No required work for this package in this phase.** Keep this package out of the phase critical path.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

**No required work for this package in this phase.** Keep this package out of the phase critical path.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Define an opt-in in-memory execution API and strict host/target compatibility checks (`src/jit.qz`, `src/lib.qz`).
- [ ] Reuse a linked image from `targets/linker/elf_static` and load supported Linux x86-64 ELF program headers (`src/loader.qz`); do not implement an ELF linker or x86-64 codegen here.
- [ ] Validate load segments, sizes/offsets, alignment, overlap, entry address and relocation completeness before executable mapping.
- [ ] Map code/data pages with correct permissions and W^X transitions; reject failed or unsafe mappings (`src/memory.qz`).
- [ ] Resolve linked internal and external symbol references and imported runtime dependencies explicitly (`src/symbols.qz`).
- [ ] Reconstruct process stack ABI (`argc`, `argv`, `envp`, auxv) and pass runtime arguments correctly (`src/execution.qz`).
- [ ] Set up startup register/frame conventions and entry transfer, as bootstrap's `jit.rs` does for Linux x86-64.
- [ ] Handle process termination, signal dispositions, guard pages and cleanup without corrupting the running compiler.
- [ ] Test executable page protection and bounds checks; never execute unverified arbitrary QZI/module bytes.
- [ ] Expose optional CLI `run --jit` with meaningful target restrictions and error handling.
- [ ] If implementing true runtime incremental JIT rather than image-in-memory execution, design separate compilation/patching semantics first.
- [ ] Benchmark startup and memory costs against ordinary native execution; keep native AOT path independently usable.
- [ ] Include clean tests for args, stdin/out, syscalls, panic, invalid ELF image, permissions and unsupported host OS.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/execution.qz` | In-memory invocation. |
| `src/jit.qz` | JIT entrypoint and orchestration. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/loader.qz` | Load components for current host. |
| `src/memory.qz` | Executable-memory management. |
| `src/symbols.qz` | Symbol identifiers and references. |

## Repository boundary

- Current bootstrap `src/jit.rs` is a Linux x86-64 **in-memory ELF executor**, not an optimizing tiered JIT compiler.
- JIT is optional P3, does not block P0–P2, and consumes images emitted by external target packages.

## Completion / verification gates

- [ ] P0/P1/P2 completion must not depend on JIT; the roadmap lists it as an optional P3 capability.
- [ ] Do not confuse bootstrap's current in-memory ELF loader with a tiered/reoptimizing runtime JIT.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
