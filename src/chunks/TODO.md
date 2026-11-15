# chunks — Implementation TODO

> **Package:** `quazi_chunks`  
> **Source:** `src/chunks/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Translate validated semantic structures into in-memory function chunks, virtual register instructions, constants and control-flow edges. No QZI file encoding or native instruction bytes.

**Integration boundary:** typed semantic report → Chunk[]; qzi serializes/validates it, backend consumes the same representation (or a verified decoded equivalent).

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/bytecode/codegen.rs`
- `src/bytecode/chunk.rs`
- `src/bytecode/instruction.rs`
- `src/bytecode/power.rs`
- `src/bytecode/vtable.rs`
- `src/bytecode/regalloc.rs`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Define a chunk-generation context with symbol/function-index tables and semantic-to-QZI type mapping (`src/builder.qz`).
- [ ] Allocate virtual registers, reserve `r0` for return values and track function-local parameter/register counts (`src/functions.qz`).
- [ ] Emit typed literal and constant-pool references without truncating integer/float/string payloads (`src/expressions.qz`).
- [ ] Lower local declarations, assignment, reads, writes and addressable places without losing initialization rules (`src/expression/places.qz`).
- [ ] Emit direct calls and argument passing (`CallArg`, `CallIdx`), normal returns and function entry/exit (`src/expression/calls.qz`).
- [ ] Lower unary, binary numeric, comparison, conversion and boolean operators with correct width/float/signedness flags (`src/expression/operators.qz`).
- [ ] Lower `if`/`else`, short-circuit evaluation and joins with patched branch targets (`src/control/branches.qz`).
- [ ] Lower `for condition`, `for {}`, C-style `for` and basic range/array iteration, including `break` and `continue` targets (`src/control/loops.qz`).
- [ ] Lower the P0 subset of `match` and branch result merges only when semantically type-checked (`src/control/match.qz`).
- [ ] Lower primitive arrays, safe element access, basic structs/field access and explicit bounds checks used by P0 examples (`src/expression/aggregates.qz`).
- [ ] Emit portable QZI opcode/operand layouts exactly as defined by `qzi`, not a parallel opcode vocabulary.
- [ ] Reject register, instruction, constant, offset and function index overflow before encoding; never silently wrap 8/16-bit operands.
- [ ] Keep `src/control_flow.qz` and `src/expressions.qz` as coordinators, with individual algorithm ownership in their children.
- [ ] Emit a public chunk builder API that does not need access to parser internals (`src/lib.qz`).
- [ ] Compare generated chunk disassembly and final QZI behavior with bootstrap for arithmetic, `for`, calls, branches, arrays and panic paths.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Lower trait and impl methods, generic monomorphizations and concrete call identities (`src/generics.qz`).
- [ ] Lower enum/Option/Result constructors, payload extraction, guarded `match`, `?` propagation and divergent exits.
- [ ] Lower function values and closures with correct environment allocation/lifetime semantics for self-hosted code.
- [ ] Support direct and indirect calls, method receiver adaptation and dynamic vtable calls with stable QZI descriptors.
- [ ] Emit `Lea` with explicit nonzero address-taken register block length and stable borrowed-register placements.
- [ ] Emit semantic-owned `Move`, `Drop` and `Dup` plans without duplicate destructor calls (`src/ownership.qz`).
- [ ] Implement predictable scope/early-return/branch cleanup from semantic plan (`src/ownership/cleanup.qz`).
- [ ] Generate per-type drop glue and ordering for Box/String/Array/struct/enum fields consumed from the separate `runtime/` API (`src/ownership/drop_glue.qz`).
- [ ] Preserve multi-slot values and portable call-layout metadata across recursive/specialized calls; platform-specific ABI assignment belongs to `targets/abi/*`.
- [ ] Emit target-neutral references to runtime intrinsics and `runtime/` String/Array APIs; startup and syscall definitions belong to `targets/os/*`.
- [ ] Preserve exact `str`/`&str` and borrowed/exclusive receiver representations and ownership behavior.
- [ ] Ensure function reachability retains all needed call dependencies, drop glue and trait-method targets.
- [ ] Test self-host corpus and QZI artifact compile through both Rust backend and new Quazi backend.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Finish all bootstrap QZI ops: bitwise/power, numeric cast rules, byte strings, string slicing, dense array access and checked indexing.
- [ ] Support `@api`, `@export`, `@syscall`, indirect C callbacks and foreign globals with target-neutral ABI metadata.
- [ ] Implement `@format` call-site expansion, intrinsic IDs, `size_of`/`align_of`, panic source location and non-std trap paths.
- [ ] Lower C structs/unions, named bitfields, packed fields, flexible array members and variable argument calling patterns.
- [ ] Match dynamic vtable layout constants, trait method symbol retention and reference ABI adaptation.
- [ ] Reach bytecode optimizations already done in codegen: safe inline expansion, dead-register elimination, jump fixups and register compaction as needed.
- [ ] Handle private vs exported/public library roots, symbol namespace rewriting and generic ABI gates when producing chunk sets.
- [ ] Reproduce bootstrap's traps for unsupported opcodes/intrinsics instead of dummy machine-code fallbacks.
- [ ] Differential-test all opcode classes and execution behavior for generated QZI, including current and historical fixtures where legal.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Expand chunk IR only when a new language feature or optimization demonstrates a real need; preserve QZI/backends contract.
- [ ] Consider new portable operations for advanced target features only with explicit versioning and verification rules.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/builder.qz` | Coordinate chunk creation from a validated semantic model; no parsing or type checking. |
| `src/control/branches.qz` | Lower if/else and short-circuit branch edges and join points into chunks. |
| `src/control/loops.qz` | Lower loops, break, continue, and loop control edges into chunks. |
| `src/control/match.qz` | Lower pattern-match dispatch, guarded arms and branch result merges into chunks. |
| `src/control_flow.qz` | Coordinate emission of branches, loops and join points into chunks. |
| `src/expression/aggregates.qz` | Lower struct/enum/array construction and aggregate field access into chunks. |
| `src/expression/calls.qz` | Lower direct/indirect, method, intrinsic and foreign calls into chunks. |
| `src/expression/operators.qz` | Lower arithmetic, comparison, bitwise, cast and logical operations into chunks. |
| `src/expression/places.qz` | Lower lvalue/place addressing, indexing, loads, stores and assignments into chunks. |
| `src/expressions.qz` | Coordinate translation of validated expressions into chunk instructions. |
| `src/functions.qz` | Build function chunks, parameters, bodies, calls and returns. |
| `src/generics.qz` | Emit chunks for semantic-approved concrete generic specializations; no type inference. |
| `src/lib.qz` | Public entrypoint for chunk generation from a validated semantic model. |
| `src/ownership/cleanup.qz` | Emit scope-exit, early-return and temporary cleanup from the semantic cleanup plan into chunks. |
| `src/ownership/drop_glue.qz` | Emit drop glue for owned values/aggregates in the order established by semantic analysis. |
| `src/ownership.qz` | Emit moves, drops and cleanup actions from validated semantic ownership plans. |

## Repository boundary

- `chunks/` generates compiler IR/chunks; `qzi/` owns portable encoding/decoding; `targets/*` converts verified chunks to native machine code or writes native images.
- No x86 instruction encoding, SysV/Win64 implementation, process entry stubs or native linker in this package.

## Completion / verification gates

- [ ] Chunk creation is deterministic for the same typed program; serialized QZI executes correctly under the Rust bootstrap in P0.
- [ ] No chunk-generation path duplicates semantic type/borrow checking or writes native code.
- [ ] Every generated branch target, function index and register operand can pass QZI verifier checks.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
