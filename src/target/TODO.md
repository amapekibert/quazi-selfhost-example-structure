# target — TODO

> **Responsibility:** Target identity, capability checks, profile resolution, installed-component lookup and loading for the compiler; no CPU, ABI, OS, object-format or linker implementation.
> **Phase contract:** P0 = Quazi-written source → portable bootstrap-compatible QZI; Rust bootstrap supplies native output. P1 = Rust-free native self-rebuild using Quazi-written installed x86-64/Linux components. P2 = Rust-bootstrap feature parity, broader existing targets, optimization and reliability. P3 = optional extensions beyond bootstrap parity.
> [ ] = not yet verified for the *self-host ecosystem*. Existing bootstrap/runtime code is a reference, not evidence that the new cross-repository integration is complete.

## Boundaries

**This package selects and loads implementations only.** Source lives at `../targets/`, a separate repository; release artifacts are independently usable `.qzi` libraries.
**Profiles are composition descriptors**, not embedded codegen source; see `targets/profiles/`.

## Existing implementation references

Paths below refer to the separate Rust bootstrap or the existing Quazi runtime. These files are **references**, not code to duplicate inside unrelated packages.

- `src/backend/target.rs`
- `src/backend/mod.rs`
- `src/project.rs`
- `src/loader.rs`
- `src/abi.rs`
- `docs/PROJECTS.md`

## Work by global phase

### P0 — QZI-producing compiler

- [ ] Represent canonical architecture/OS/ABI identity and source-visible `@cfg` values without requiring a native plugin (`src/components.qz`, `src/compatibility.qz`).
- [ ] Support explicit target and host-target defaults for source analysis; ensure QZI-only compilation never probes or loads the native assembler/linker.
- [ ] Separate source platform configuration (`@cfg`, target-size assumptions) from executable emission settings.
- [ ] Diagnose unsupported target triples and conflicting cfg flags deterministically.
- [ ] Test that target-neutral code compiles to compatible portable QZI without installed native target packages.

### P1 — Native backend and independent self-host

- [ ] Define a small target component API and typed capability descriptors (`src/components.qz`) covering arch, OS, ABI, object writer and linker.
- [ ] Resolve `x86_64-linux-static` profile into the five *external* packages in `targets/` (`src/resolver.qz`).
- [ ] Discover locally installed target component .qzi libraries, check their compatible version/capability contracts, and fail when absent (`src/loader.qz`, `src/registry.qz`).
- [ ] Validate `x86_64` + Linux + SysV AMD64 + ELF64 + static ELF linker compatibility (`src/compatibility.qz`).
- [ ] Expose selected target data layout, native pointer width, symbol policy, OS runtime and linker options to `quazi/src/backend` without copying any of the implementations.
- [ ] Prevent target package loading from depending on a network connection or external manager during self-rebuild.
- [ ] Test incorrect/missing/incompatible component combinations and an end-to-end self-build with preinstalled target packages.

### P2 — Bootstrap parity, optimization and reliability

- [ ] Resolve existing bootstrap counterparts `x86_64-windows`, `x86_64-netbsd`, `x86_64-none` where matching external components are available.
- [ ] Support the supplied `x86_64-linux-musl` profile and associated libc/runtime constraints; preserve explicit libc opt-in policy.
- [ ] Match Rust-bootstrap handling of target-specific linker settings, output naming, mangling/crash switches, host vs cross-target selection.
- [ ] Make target ABI/object/linker capability mismatch errors explicit and actionable (`src/compatibility.qz`).
- [ ] Validate source configuration and resolved native format for each shipped declarative target profile; use differential tests against bootstrap where supported.

### P3 — Optional expansion

- [ ] Design a versioned independently downloadable component registry/manifest format (`src/manifest.qz`) after the local P1 contract is stable.
- [ ] Integrate optional `qzup` installation/version-selection interfaces without placing package-manager implementation in `quazi`.
- [ ] Allow new architectures, operating systems, ABI and linkers to be composed declaratively without modifying parser or semantic code.

## Acceptance checks

- [ ] P0 `.qzi` generation passes with zero installed target packages.
- [ ] P1 target resolver loads preinstalled external component libraries, not bundled x86_64 code inside `quazi`.
- [ ] Every target profile resolves to a compatible set or fails with a named missing/invalid component.
