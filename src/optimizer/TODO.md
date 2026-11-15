# optimizer — Implementation TODO

> **Package:** `quazi_optimizer`  
> **Source:** `src/optimizer/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Operate on validated semantic annotations/chunks using correctness-preserving analyses and QZI transformations. Optional before P2.

**Integration boundary:** typed/verified chunks → optimized verified chunks; pipeline opts in once baseline compiler and self-host are correct.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/semantic/optimize.rs`
- `src/bytecode/constprop.rs`
- `src/bytecode/regalloc.rs`
- `src/bytecode/codegen.rs`
- `src/bytecode/power.rs`

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

- [ ] Specify optimizer pass interface, preconditions, preserved invariants and deterministic sequencing (`src/pipeline.qz`, `src/lib.qz`).
- [ ] Build chunk CFG/basic blocks, reachable paths, predecessors/successors and dominance information as needed (`src/control_flow.qz`).
- [ ] Implement use/def sets, liveness, call/escape side-effect classification and memory alias conservatism (`src/analysis.qz`).
- [ ] Implement local and cross-block constant folding/propagation with join/meet lattice (`src/constants.qz`, `src/propagation.qz`).
- [ ] Preserve signed/unsigned numeric semantics, overflow, float NaN and integer power behavior during folding.
- [ ] Kill constant facts for address-taken slots and unknown aliasing writes; do not fold through borrowed mutable memory.
- [ ] Eliminate unreachable blocks, unused expressions and dead virtual-register definitions only when side-effect free (`src/dead_code.qz`).
- [ ] Implement inlining of safe candidate function chunks with correct call args, return handling and branch/jump retargeting (`src/inlining.qz`).
- [ ] Perform register compaction/linear scan with pinned address-taken block constraints and stable multi-slot ABI order.
- [ ] Add loop simplification and safe strength reduction (power of two etc.) with proof of equivalence (`src/loops.qz`).
- [ ] Implement conservative escape/retention analysis where ownership semantics permit (`src/escape.qz`).
- [ ] Revalidate instruction/register/branch/constant metadata after each destructive pass and before QZI encoding.
- [ ] Match observable results of unoptimized program, Rust bootstrap and Quazi optimized output for a broad regression corpus.
- [ ] Test aliases, loops, dead backedges, address-taken arrays, multi-slot results, foreign calls and panic paths.
- [ ] Benchmark optimizer cost and generated binary/runtime performance without adding debug/release optimization profiles.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Consider interprocedural specialization, profile-guided transformations and advanced dataflow only if benchmarks justify them.
- [ ] Add architecture-aware post-QZI passes behind backend contracts rather than inserting target semantics into portable QZI.
- [ ] Keep optimization proofs/fallbacks explicit and protect reproducibility of portable artifacts.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/analysis.qz` | Build conservative dataflow, use/def and liveness facts for optimization. |
| `src/constants.qz` | Compile-time constants. |
| `src/control_flow.qz` | Control-flow representation and checks. |
| `src/dead_code.qz` | Eliminate unreachable/dead code. |
| `src/escape.qz` | Escape and allocation analysis. |
| `src/inlining.qz` | Inlining subject to cost limits. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/loops.qz` | Loop transformations when justified. |
| `src/pipeline.qz` | Compose optimization passes, not full compiler compilation stages. |
| `src/propagation.qz` | Value and constant propagation. |

## Repository boundary

- Operate on target-neutral QZI/chunks or an explicitly documented machine-independent form. Register allocation and target-specific peepholes belong to `targets/arch/*`.

## Completion / verification gates

- [ ] P0 and P1 may use zero optimizer passes; they cannot be blocked by optimization completeness.
- [ ] Every pass preserves the semantic ownership, trap, ABI and QZI verification invariants established before it.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
