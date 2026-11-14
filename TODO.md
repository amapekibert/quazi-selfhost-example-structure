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
- [ ] Implement the essential lowering pipeline and QZI generation for a usable language subset.
- [ ] Emit QZI compatible with the existing Rust bootstrap's QZI-to-native compilation path.
- [ ] Compile and run representative `.qz` programs through the **Quazi frontend → QZI → Rust backend** pipeline, comparing behavior with the bootstrap.

**Milestone:** A Quazi-written compiler can produce working QZI programs; the Rust bootstrap supplies native code generation and linking. The Quazi native backend is not a P0 requirement.

## P1 — Native backend & initial self-hosting

- [ ] Implement the additional language constructs and semantic rules needed to compile the self-hosted compiler itself.
- [ ] Implement the required ownership, borrowing, lifetime, and cleanup behavior for that subset.
- [ ] Implement a **native x86-64 backend code generator in Quazi**, consuming the same QZI representation produced in P0.
- [ ] Implement the initial object/executable emission, ABI, symbol resolution, and linking path needed for self-hosting.
- [ ] Complete the necessary module, project, dependency, and compiler-driver workflows for building the compiler itself.
- [ ] Validate the bootstrap chain: Rust builds the first Quazi compiler; the Quazi compiler rebuilds itself using **its own QZI and native backend**, and the rebuilt compiler passes the required regression tests.

**Milestone:** Quazi compiles itself into a native executable without invoking the Rust backend in its own compilation pipeline. Full bootstrap feature parity and advanced optimization are not required yet.

## P2 — Rust-bootstrap parity, optimization & reliability

- [ ] Implement the remaining language features and semantics already supported by the Rust bootstrap.
- [ ] Complete outstanding QZI, backend, linker, build-system, and target functionality from the Rust bootstrap.
- [ ] Implement and refine IR and native-code optimization passes.
- [ ] Improve compilation speed, caching, incremental workflows, and resource usage.
- [ ] Strengthen diagnostics, error recovery, validation, and compiler tooling.
- [ ] Expand differential, regression, cross-platform, fuzzing, stress, and performance testing.

**Milestone:** The self-hosted compiler reaches practical feature parity with the Rust bootstrap, with improved performance, correctness, and reliability.

## P3 — Advanced capabilities & future expansion

- [ ] Substantially extend existing limited language capabilities and add justified new features.
- [ ] Develop more advanced optimization analyses and transformation pipelines.
- [ ] Add further backends, architectures, targets, and toolchain integrations as independent extensions.
- [ ] Expand QZI capabilities, reusable compiler interfaces, and advanced compilation workflows.
- [ ] Develop optional execution features, including in-memory QZI execution or JIT, if prioritized.
- [ ] Extend developer tooling, language services, diagnostics, and ecosystem integration.

**Milestone:** Quazi evolves beyond the Rust bootstrap while retaining a modular, maintainable compiler codebase.

---

**Scope rule:** P0 outputs QZI and relies on the existing Rust native backend. P1 implements Quazi's own native backend and achieves initial independent self-hosting. P2 completes and optimizes bootstrap functionality. P3 extends beyond it. P2/P3 features must not block P0/P1 unless they are genuine prerequisites.
