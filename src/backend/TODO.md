# backend — TODO

> **Responsibility:** Quazi compiler-side backend orchestration and target-neutral contracts. Concrete CPU codegen, ABI lowering, OS start/runtime, object writing and linking belong exclusively to the separate `targets/` repository.
> **Phase contract:** P0 = Quazi-written source → portable bootstrap-compatible QZI; Rust bootstrap supplies native output. P1 = Rust-free native self-rebuild using Quazi-written installed x86-64/Linux components. P2 = Rust-bootstrap feature parity, broader existing targets, optimization and reliability. P3 = optional extensions beyond bootstrap parity.
> [ ] = not yet verified for the *self-host ecosystem*. Existing bootstrap/runtime code is a reference, not evidence that the new cross-repository integration is complete.

## Boundaries

**Owner:** `quazi/src/backend` = adapters, interfaces, dispatch, result types.
**External dependency:** `targets/arch`, `targets/abi`, `targets/os`, `targets/object`, `targets/linker` = actual native implementation.
**Runtime distinction:** `runtime/` = language types/methods; target OS/startup components = platform hooks and intrinsics.

## Existing implementation references

Paths below refer to the separate Rust bootstrap or the existing Quazi runtime. These files are **references**, not code to duplicate inside unrelated packages.

- `src/backend/mod.rs`
- `src/backend/target.rs`
- `src/backend/linker.rs`
- `src/bytecode/chunk.rs`
- `src/abi.rs`

## Work by global phase

### P0 — QZI-producing compiler

**No required implementation in this phase.**

### P1 — Native backend and independent self-host

- [ ] Define a stable target-neutral `Backend` contract: verified in-memory chunks/QZI, selected target components, output kind, diagnostics and generated artifacts (`src/backend.qz`, `src/lib.qz`).
- [ ] Choose exactly one consumption model for native plugins (decoded/verified chunks or equivalent QZI IR); keep instruction schemas defined by `quazi/src/qzi` / `quazi/src/chunks`, not redefined in target packages.
- [ ] Provide the codegen dispatch boundary (`src/codegen.qz`) that calls the installed `targets/arch/x86_64` component without containing x86-64 instruction definitions, machine registers or encoders.
- [ ] Define target-independent value-location and register requirement descriptors (`src/registers.qz`) sufficient for the architecture plugin to allocate actual registers; do not write an x86-64 allocator here.
- [ ] Define object artifact and section interface types (`src/objects.qz`) consumed by `targets/object/elf64`; no ELF header/section binary encoding here.
- [ ] Define relocation request and resolution contracts (`src/relocations.qz`) without hardcoded R_X86_64_* relocation encodings.
- [ ] Define linker invocation/results (`src/linking.qz`) and invoke `targets/linker/elf_static`; do not merge ELF sections in `quazi`.
- [ ] Wire target selection and native output request through `src/target.qz` to `quazi/src/target` resolver; no duplicate target manifest registry.
- [ ] Pass correct calls/returns/data-layout/runtime-hook requirements through selected ABI and OS components, never implement SysV argument registers or Linux syscalls in the backend host.
- [ ] Reject unsupported opcodes, target capability combinations, unresolved imports and plugin-version mismatches before producing an executable.
- [ ] Run QZI → `targets` component → ELF object → linked executable without the Rust native backend; feed this path into a complete compiler self-rebuild.

### P2 — Bootstrap parity, optimization and reliability

- [ ] Extend dispatch to existing bootstrap-supported Windows/NetBSD/freestanding target capabilities as their packages become parity-ready.
- [ ] Support external linker selection and archive/shared/static artifact requests by passing structured options to `targets/linker/*` rather than parsing native formats internally.
- [ ] Support compiler-known foreign ABI symbols, callbacks and exported interfaces via target-neutral metadata and adapters provided by selected target components.
- [ ] Ensure deterministic native artifact/error reporting across different codegen, ABI, writer and linker package versions.
- [ ] Compare native output behavior, object metadata, and linker diagnostics with Rust bootstrap on a per-target corpus.
- [ ] Benchmark backend orchestration overhead independently of encoder/writer/linker implementations.

### P3 — Optional expansion

- [ ] Allow independently installable alternative architecture backends through the same stable contracts when the core target loader is ready.
- [ ] Consider pluggable codegen optimization levels or profile-guided plugin hooks only if required by a real use case (no artificial debug/release split).

## Acceptance checks

- [ ] No concrete x86-64 encoding, ABI algorithms, ELF/PE/COFF serialization or native linker implementation remains in this repository.
- [ ] P1 Quazi compiler rebuild uses real native implementations from `targets/` without Rust compiling its native output.
- [ ] Plugins report capability mismatches as errors; portable chunks/QZI are not corrupted by target adaptation.
