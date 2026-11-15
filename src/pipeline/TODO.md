# pipeline — Implementation TODO

> **Package:** `quazi_pipeline`  
> **Source:** `src/pipeline/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Orchestrate compilation phases and session state; never implement lexing, semantic checks, QZI codec or native instruction selection itself.

**Integration boundary:** cli/build → pipeline → source/modules/frontend/semantic/chunks/qzi; from P1 optionally target/backend/linker.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/main.rs`
- `src/analysis.rs`
- `src/loader.rs`
- `src/cancel.rs`
- `src/incremental.rs`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Expose compiler request/options/result types that distinguish QZI emit, check-only and failed compilation (`src/options.qz`, `src/result.qz`).
- [ ] Define one compilation session containing source IDs, imported files, diagnostics and target-neutral settings (`src/session.qz`).
- [ ] Create ordered P0 phases: read sources → resolve modules → lex/parse → semantic → chunks → QZI verify/encode (`src/pipeline.qz`).
- [ ] Call each package through a narrow API; do not import private parser/borrow/codegen implementation files directly.
- [ ] Stop on fatal errors, preserve all collected nonfatal diagnostics and return a structured failure without partial output.
- [ ] Keep QZI output independent from installed native targets, linker, host assembler and Rust's codegen at the boundary.
- [ ] Provide a service entrypoint callable by `cli` and future build runners (`src/compiler.qz`, `src/lib.qz`).
- [ ] Handle multiple source entry files and deterministic module order when requested.
- [ ] Report stage progress and timings via callbacks/data without coupling core logic to terminal colors or stdout.
- [ ] Run sample `.qz → .qzi → Rust bootstrap native` end-to-end tests and compare stdout/exit status.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Add compilation targets for QZI → native object → link → executable using Quazi's backend.
- [ ] Support `check` vs build, executable/library compilation and entry-point selection for self-build.
- [ ] Coordinate import/injection of the **separate** `runtime/` (prelude) and `std/` package sources, plus semantic specialization and portable QZI composition.
- [ ] Coordinate `quazi_backend` and `quazi_target` interfaces with separately installed `targets/` components, passing runtime dependency/configuration settings without implementing ABI/startup code.
- [ ] Implement explicit compiler bootstrap stages with no Rust calls inside the Quazi compilation pipeline.
- [ ] Provide reproducible options and output errors to build/CLI for rebuilding the compiler itself.
- [ ] Detect self-host compiler dependency graph and prevent cycles/invalid startup sequences.
- [ ] Verify the newly built compiler rebuilds itself again and compares key output behavior.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Add cached/exact/partial compilation sessions by delegating state storage to build package.
- [ ] Integrate optimization pass scheduling after correctness checks, with invalidation rules and no user debug/release mode split.
- [ ] Restore bootstrap timing/progress reporting, cancellation checkpoints and deterministic behavior on failures.
- [ ] Support artifact variants, target overrides, native source inputs, libraries, tests and QZI-only rebuilds.
- [ ] Implement consistent failure/cleanup logic for temporary object, project cache and output files.
- [ ] Compare cold/warm compilation speed and behavior against bootstrap; benchmark compiler memory and stage costs.
- [ ] Add internal debugging/disassembly/trace hooks without polluting public compiler APIs.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Expose stable compile-session APIs for LSP, alternate frontend adapters and external tools only when needed.
- [ ] Support plug-in target/backend workflows or distributed compilation only through explicitly versioned interfaces.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/compiler.qz` | Compiler service API. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/options.qz` | Explicit configuration without build optimization profiles. |
| `src/pipeline.qz` | Composition of necessary compilation stages. |
| `src/result.qz` | Result and error types for compiler execution. |
| `src/session.qz` | Per-invocation compiler state. |

## Repository boundary

- `pipeline/` orders compilation stages; `runtime/` supplies the prelude and `targets/` supplies native implementations.
- `lsp/` is a separate repository: expose reusable analysis APIs but do not implement the language-server protocol here.

## Completion / verification gates

- [ ] P0 never invokes native codegen in Quazi: it emits compatible QZI only.
- [ ] P1 clean Quazi-to-Quazi rebuild does not invoke Rust in normal compilation chain.
- [ ] The pipeline contains orchestration/state, not a duplicate copy of parser or semantic algorithms.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
