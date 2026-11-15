# Quazi Self-Hosted Compiler — Global TODO

> High-level roadmap for rewriting the Quazi compiler in Quazi.
> Tracks milestones and major capabilities only; detailed implementation tasks belong in the source modules.

## Priority definitions

- **P0 — QZI-producing compiler:** Implement the minimum Quazi-written compiler pipeline needed to compile `.qz` into compatible `.qzi`. Use the existing Rust bootstrap to turn QZI into native binaries during this stage. **No self-hosted native codegen in P0.**
- **P1 — Native backend & self-hosting:** Implement the essential native code generation and linking path in Quazi, alongside the language functionality required to rebuild the compiler without Rust in the normal compilation chain. Full Rust-bootstrap feature parity is not yet required.
- **P2 — Bootstrap parity & optimization:** Finish capabilities already available in the Rust bootstrap, improve generated code and compiler performance, and strengthen correctness and tooling.
- **P3 — Major expansion:** Substantially extend limited capabilities and introduce new language, compiler, and toolchain features beyond the Rust bootstrap.

## P0 — QZI compiler foundation

- [ ] Establish the Quazi workspace, compiler entry point, and basic build workflow.
- [ ] Implement source handling, lexer, parser, and core AST representation.
- [ ] Implement initial name resolution, type checking, semantic validation, and diagnostics.
- [ ] Implement essential semantic-model → chunks construction and portable QZI serialization for a usable language subset.
- [ ] Emit QZI compatible with the existing Rust bootstrap's QZI-to-native compilation path.
- [ ] Compile and run representative `.qz` programs through the **Quazi frontend → QZI → Rust backend** pipeline, comparing behavior with the bootstrap.

**Milestone:** A Quazi-written compiler can produce working QZI programs; the Rust bootstrap supplies native code generation and linking. The Quazi native backend is not a P0 requirement.

## P1 — Native backend & initial self-hosting

- [ ] Implement the additional language constructs and semantic rules needed to compile the self-hosted compiler itself.
- [ ] Implement the required ownership, borrowing, lifetime, and cleanup behavior for that subset.
- [ ] Implement the **native x86-64 backend in Quazi via the separate `targets/` packages**, with `quazi/src/backend` providing only the compiler-side API and dispatch.
- [ ] Implement the initial ELF64 object writer, SysV AMD64 ABI, Linux OS/runtime hooks and static ELF linker **inside `targets/`**, and integrate them into the self-host compiler.
- [ ] Complete the necessary module, project, dependency, and compiler-driver workflows for building the compiler itself.
- [ ] Validate the bootstrap chain: Rust builds the first Quazi compiler; the Quazi compiler rebuilds itself using **its own QZI and native backend**, and the rebuilt compiler passes the required regression tests.

**Milestone:** Quazi compiles itself into a native executable without invoking the Rust backend in its own compilation pipeline. Full bootstrap feature parity and advanced optimization are not required yet.

## P2 — Rust-bootstrap parity, optimization & reliability

- [ ] Implement the remaining language features and semantics already supported by the Rust bootstrap, preserving compatibility with the separate `runtime/` (prelude) and `std/` packages.
- [ ] Complete outstanding portable QZI/compiler features in `quazi/` and native target/ABI/writer/linker parity in `targets/`, maintaining their interface boundary.
- [ ] Implement and refine IR and native-code optimization passes.
- [ ] Improve compilation speed, caching, incremental workflows, and resource usage.
- [ ] Strengthen diagnostics, error recovery, validation, and compiler tooling.
- [ ] Expand differential, regression, cross-platform, fuzzing, stress, and performance testing.

**Milestone:** The self-hosted compiler reaches practical feature parity with the Rust bootstrap, with improved performance, correctness, and reliability.

## P3 — Advanced capabilities & future expansion

- [ ] Substantially extend existing limited language capabilities and add justified new features.
- [ ] Develop more advanced optimization analyses and transformation pipelines.
- [ ] Add further backends, architectures and targets **in `targets/`** as independent extensions; keep integrations and toolchain services in their own repositories.
- [ ] Expand QZI capabilities, reusable compiler interfaces, and advanced compilation workflows.
- [ ] Develop optional execution features, including in-memory QZI execution or JIT, if prioritized.
- [ ] Extend compiler diagnostics/APIs and integrate with **separate** `lsp/`, `qzup/` and developer-tool repositories; do not embed their implementations.

**Milestone:** Quazi evolves beyond the Rust bootstrap while retaining a modular, maintainable compiler codebase.

---

**Scope rule:** P0 outputs QZI and relies on the existing Rust native backend. P1 implements Quazi's own native backend and achieves initial independent self-hosting. P2 completes and optimizes bootstrap functionality. P3 extends beyond it. P2/P3 features must not block P0/P1 unless they are genuine prerequisites.

## Repository ownership (all phases)

- **`quazi/`:** `cli`, `pipeline`, `frontend`, `semantic`, `chunks`, portable `qzi`, optimizer, compiler-facing `backend` API and `target` discovery.
- **`targets/`:** actual architecture code generators, operating-system hooks, ABI lowering, ELF/PE object writers, native linkers and declarative target profiles. Distributed as independently usable QZI target components.
- **`runtime/` (formerly prelude):** built-in language types, owned memory containers, trait definitions, panic/formatting, qoroutine support and associated runtime API; its implementation stays separate from the compiler.
- **`std/`, `lsp/`, `qzup/`:** separate repositories; the compiler links or calls their public interfaces, not their source implementations.

**Cross-repository P1 gate:** a compiler binary produced by the Rust bootstrap must build itself through `quazi` + installed Quazi-written `targets` components + separate `runtime`/`std` dependencies, without Rust native code generation or native linking in the normal self-host build. Full feature parity remains P2.
