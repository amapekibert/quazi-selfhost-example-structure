# semantic — Implementation TODO

> **Package:** `quazi_semantic`  
> **Source:** `src/semantic/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Validate resolved AST, infer/check types, enforce ownership and produce the typed/annotated model and cleanup decisions consumed by chunks. No code emission.

**Integration boundary:** frontend AST + module bindings → typed semantic report + ownership/drop plan → chunks. All checks must occur before emitting QZI.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/semantic/mod.rs`
- `src/semantic/declare.rs`
- `src/semantic/typecheck.rs`
- `src/semantic/types.rs`
- `src/semantic/type_context.rs`
- `src/semantic/declaration_types.rs`
- `src/semantic/arithmetic.rs`
- `src/semantic/initialization.rs`
- `src/semantic/control_flow.rs`
- `src/semantic/borrow.rs`
- `src/semantic/places.rs`
- `src/semantic/ownership.rs`
- `src/semantic/retention.rs`
- `src/semantic/unused.rs`
- `src/type_resolution.rs`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Define semantic type identity and primitives: `void`, `!`, booleans, signed/unsigned integer widths, floats, strings, pointers and arrays (`src/types/types.qz`).
- [ ] Introduce declaration tables for functions, locals, constants, type names and module-qualified symbols (`src/symbols.qz`).
- [ ] Implement lexical scopes, shadowing rules, parameter bindings and declaration/use lookup (`src/scopes.qz`).
- [ ] Resolve qualified imports and source aliases through `modules` without reimplementing module loading (`src/resolution.qz`).
- [ ] Define a typed expression/item representation with origin spans, resolved symbols and explicit result types (`src/analyzer.qz`).
- [ ] Check function signatures, parameter/return annotations, entry signature and return coverage (`src/types/check/items.qz`).
- [ ] Infer literal and local-binding types with expected-type contexts and deterministic failures (`src/types/inference.qz`).
- [ ] Implement assignment, argument and return compatibility for the P0 primitive/array/reference subset (`src/types/check/compatibility.qz`).
- [ ] Check names, literals, calls, local initializers, blocks, array/struct literals and expressions (`src/types/check/expressions.qz`).
- [ ] Check direct call arity and argument types, function expression calls and basic method resolution (`src/types/check/calls.qz`).
- [ ] Type arithmetic/comparison/logical operators, unary negation, booleans and explicit numeric conversions (`src/types/check/operators.qz`).
- [ ] Enforce `const` immutable vs `var` writable, initialized-before-read and legal assignments (`src/validation.qz`).
- [ ] Check conditional and loop conditions, returns, `break`/`continue` placement, reachability for sound QZI (`src/control_flow.qz`).
- [ ] Resolve and type basic constant values without letting invalid compile-time values leak into codegen (`src/constants.qz`).
- [ ] Handle minimal `Option[T]`, `Result[T,E]` and matching only if P0 test corpus actually needs them; record their ABI consistently.
- [ ] Establish typed call/branch/place contracts and poison/error states so lowering cannot run on invalid semantic reports.
- [ ] Test successful and rejected programs for scopes, return types, initializedness, arithmetic width, invalid calls and control-flow.
- [ ] Compare diagnostics and runtime behavior against bootstrap on the selected P0 QZI-subset corpus.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Implement named/generic types, generic functions, concrete substitution, specializations and monomorphization identities (`src/types/generics.qz`).
- [ ] Check struct/enums, aliases, field/method access, constructors and pattern bindings needed by self-host sources.
- [ ] Implement trait declarations/implementations, trait bounds, method lookup, receiver adaptation and `dyn Trait` dispatch contracts (`src/types/traits.qz`).
- [ ] Type closures, function values, captured variables and their ownership/layout rules for the forms actually used by the compiler.
- [ ] Add ownership classification for scalar Copy, owned aggregates, references, function values and heap-backed runtime types (`src/ownership/ownership.qz`).
- [ ] Track moves, partial/whole value invalidation, reinitialization and branch/loop merges (`src/ownership/moves.qz`).
- [ ] Establish `Place`/projection identity for fields, array indexes, dereference and assignment targets (`src/ownership/ownership.qz`).
- [ ] Implement shared and exclusive loan creation, lexical expiry, overlap and conflict checks (`src/ownership/borrow/loans.qz`, `conflicts.qz`).
- [ ] Reject invalid reference provenance, reference escape, lifetime violations and use-after-move (`src/ownership/borrow/provenance.qz`, `src/ownership/lifetimes.qz`).
- [ ] Implement receiver semantics: legacy borrowing, explicit `&T`, `&T!` and consuming `self: T` with correct call-site effects.
- [ ] Analyze ownership changes across `if`/`else if`/`match`, `for` iterable moves, C-style initialization and `break`/`continue` transitions.
- [ ] Plan exact cleanup on normal scope exit, `ret`, early failure and temporary ownership transfers (`src/ownership/cleanup.qz`).
- [ ] Compute retained owners and call-return ownership semantics for correct String, Array, Box, Option and Result use (`src/ownership/retention.qz`).
- [ ] Validate concrete generic parameter/result register-block layouts and reject unsupported multi-slot uses before emission.
- [ ] Check `?` against enclosing callable `Option`/`Result` return type and preserve error payload layout.
- [ ] Validate compiler source and its transitive runtime/stdlib calls without false-positive loans or silent ownership loss.
- [ ] Test trait objects, generic recursion, enum pattern branches, closures, owners in loops and borrowed method receivers against bootstrap.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Match full bootstrap numeric coercion/cast policy: signedness, float widths, saturating casts and special f32/raw-pointer rules.
- [ ] Implement complete bitwise/shift/power operator requirements, checked integer exponentiation and floating power semantics.
- [ ] Finish raw pointer/reference conversion, C type compatibility, `str` vs `&str` representation equivalence and distinct dyn trait identities.
- [ ] Reach C ABI diagnostics: variadics, `@repr(C)` layout, packed structs, unions, bitfields, flexible tails, callbacks and foreign globals.
- [ ] Implement full `@cfg` validation, `@derive` semantics, named arguments, `@format`, `@ignore` and declaration visibility.
- [ ] Enforce compiler's `any` restriction: only final erased `@format ...args: any`, no runtime dynamic value type.
- [ ] Finish closure expected-type inference, capture restrictions, owned fn value propagation and nested return contexts.
- [ ] Match detailed borrow checker place/loan graph behavior for index projections, temporaries, branches, re-own assignment and conditional exits.
- [ ] Implement concrete call-site ownership/effect graphs, symbolic generic substitution and conservative opaque boundary rejection.
- [ ] Implement retention/cleanup correctness for nested owned formatting, generic aggregates, dynamic receivers and QZI dependency boundaries.
- [ ] Complete source-level ownership ABI compatibility gates, proof boundaries and fallback rejection for legacy unverified dependencies.
- [ ] Report unused bindings/functions/imports, unreachable code, exhaustive matches and unreachable arm warnings.
- [ ] Harden semantic error recovery, cancellation, specialization budget limits, recursion guards and adversarial deeply nested programs.
- [ ] Differential-test all bootstrap semantic regressions: `*_tests.rs` cases for arithmetic, ownership, places, retention, control flow and initialization.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Improve currently limited capture/borrow/effect analysis only with soundness criteria and independently testable interfaces.
- [ ] Consider more expressive generic/trait bounds and ownership contracts after the bootstrap compatibility gate.
- [ ] Any new semantic feature must define memory behavior, target ABI implications and QZI compatibility before acceptance.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/analyzer.qz` | Top-level semantic phase coordination. |
| `src/constants.qz` | Compile-time constants. |
| `src/control_flow.qz` | Control-flow representation and checks. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/ownership/borrow/conflicts.qz` | Enforce loan conflicts for reads, writes, moves and overlapping places. |
| `src/ownership/borrow/loans.qz` | Create, track and expire shared/exclusive loans over lexical regions. |
| `src/ownership/borrow/provenance.qz` | Track origins of borrowed values and prevent invalid reference escape. |
| `src/ownership/borrowing.qz` | Borrow rules. |
| `src/ownership/cleanup.qz` | Deterministic cleanup on all exits. |
| `src/ownership/lifetimes.qz` | Lifetime checks. |
| `src/ownership/moves.qz` | Move tracking. |
| `src/ownership/ownership.qz` | Ownership semantics and state. |
| `src/ownership/retention.qz` | Retention and resource ownership. |
| `src/resolution.qz` | Name and path resolution. |
| `src/scopes.qz` | Lexical and semantic scopes. |
| `src/symbols.qz` | Symbol identifiers and references. |
| `src/types/check/calls.qz` | Checking function/method calls, arguments, overload resolution and generic call instantiation. |
| `src/types/check/compatibility.qz` | Assignment/return type compatibility and implicit or explicit coercion checks. |
| `src/types/check/expressions.qz` | Type-checking expression forms, including contextual expected types; delegate specialized cases. |
| `src/types/check/items.qz` | Type-checking item declarations, function bodies, parameters and declared signatures. |
| `src/types/check/operators.qz` | Unary/binary operator type rules, numeric compatibility and return types. |
| `src/types/checking.qz` | Type checking. |
| `src/types/constraints.qz` | Type constraints. |
| `src/types/generics.qz` | Generic instantiation and monomorphization rules. |
| `src/types/inference.qz` | Type inference. |
| `src/types/traits.qz` | Traits and contracts. |
| `src/types/types.qz` | Type definitions and identity. |
| `src/validation.qz` | Semantic invariants and diagnostics. |

## Repository boundary

- `semantic/` determines source-language meaning, safety, layout constraints and target-neutral call ABI descriptions. Concrete calling-convention register/stack assignments belong to `targets/abi/*`.
- `runtime/` is a separate library: check its declared signatures without copying `Option`, `Result`, `String`, `Array`, `Box` implementations into the compiler.

## Completion / verification gates

- [ ] Self-host compiler code typechecks and ownership-checks entirely in Quazi with no Rust semantic calls by P1.
- [ ] No rejected borrow/move state is converted into superficially valid QZI; no backend may infer ownership from missing semantic facts.
- [ ] Every emitted cleanup action is derived from a validated semantic plan, never guessed by the backend.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
