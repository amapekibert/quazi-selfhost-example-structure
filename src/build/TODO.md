# build — Implementation TODO

> **Package:** `quazi_build`  
> **Source:** `src/build/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Build projects and packages: read quazi.toml, resolve dependency artifacts, select outputs, manage cache and artifacts. Never implement parser, semantic rules or instruction encoding.

**Integration boundary:** CLI → build → pipeline; packages/source/QZI are inputs; target describes native outputs; QZC is cache, quazi.lock is dependency resolution.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/project.rs`
- `src/package.rs`
- `src/incremental.rs`
- `src/main.rs`
- `src/backend/archive.rs`
- `docs/PROJECTS.md`
- `docs/LIBRARIES.md`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Read and validate minimal `[package]` name/version, package root and output directories (`src/manifest.qz`).
- [ ] Select `.qz` file or a project entry for compilation to `.qzi`, with output-name override (`src/artifacts.qz`).
- [ ] Support one executable and one library entry where needed without requiring native linking in P0.
- [ ] Resolve local path dependencies for the P0 compiler/runtime source subset (`src/dependencies.qz`).
- [ ] Build minimal deterministic project dependency order; detect cycles without recursive hangs (`src/graph.qz`).
- [ ] Invoke `pipeline` for source → QZI with resolved entry sources and configuration; do not re-run compiler passes here.
- [ ] Write QZI output atomically, avoid truncated artifact on failure and propagate file I/O errors.
- [ ] Preserve build artifacts outside source files and prevent an output path escaping the selected project directory when inappropriate.
- [ ] Prove `.qzi` is accepted by Rust bootstrap backend; no C/assembler/linker inputs are required by QZI-only mode.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Support multiple compiler library packages with `[lib]` and `[[bin]]`, stable package identities and entry-point selection.
- [ ] Resolve compiler package imports/dependencies and their build order across the entire self-host tree.
- [ ] Distinguish source `.qz`, precompiled `.qzi` libraries and native object/executable outputs.
- [ ] Build the self-host executable by passing native artifact requests to `quazi_backend`, which resolves preinstalled `targets/` components; include the separate `runtime/` dependency without embedding its sources here.
- [ ] Implement basic package settings for `std`, `crash_handler` and `mangling` as required by compiler self-build.
- [ ] Support Quazi library QZI output with public interfaces, namespace roots and transitive dependency closure.
- [ ] Stage artifacts in a reproducible directory; never mix stale Rust-built compiler objects with self-host outputs.
- [ ] Validate cold clean build, repeated build and self-rebuild equivalence without Rust-native compilation calls.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Parse the complete bootstrap manifest: multiple bins/lib, `[dependencies]`, `[link]`, target overrides, `[cc]`, `[asm]`, package switches and out_dir.
- [ ] Resolve dependencies from filesystem, one source file, QZI, Git and archives with validated package names and versions.
- [ ] Implement `quazi.lock` reading/writing, commit/checksum recording, SHA256 verification and package cache isolation.
- [ ] Download/extract dependencies safely with traversal protections, reproducible locations and clear offline errors.
- [ ] Accept native C/ASM inputs, `.a`/`.so` and structured linker flags in build manifests; forward them to the chosen external `targets/linker/*` implementation, never parse/link them in `build`.
- [ ] Implement QZC exact-hit and partial chunk reuse with dependency fingerprints (`src/cache.qz`, `src/fingerprint.qz`).
- [ ] Include source, QZI ownership/interface and installed target-component identities in cache keys; reject stale artifacts rather than baking ABI logic into `build`.
- [ ] Restore bootstrap WPO/analysis even on partial QZC hits to prevent caching invalid semantic conclusions.
- [ ] Support `--no-incremental`, targeted invalidation, cache garbage collection, clean, artifacts and multi-target output layout.
- [ ] Handle project dependency add/remove/fetch, named artifact selection and source interface exports.
- [ ] Benchmark cold/warm builds, large projects and cache invalidation; match bootstrap outputs for build graphs and diagnostics.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Add workspace-level or reproducible remote artifact caching only if local build workflows justify their maintenance cost.
- [ ] Support external target package delivery through a separate registry contract, without folding qzup into build internals.
- [ ] Consider parallel package builds with deterministic linking, cancellation and safe shared cache coordination.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/artifacts.qz` | QZI, object and native output management. |
| `src/cache.qz` | Compilation artifacts cache. |
| `src/dependencies.qz` | Package dependency resolution. |
| `src/fingerprint.qz` | Stable cache identity. |
| `src/graph.qz` | Dependency ordering and cycles. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/manifest.qz` | Manifest format and validation. |

## Repository boundary

- `build/` owns package DAG, manifests, cache and artifact policy; native machine encoding, object writing and linking live in external `targets/` packages.
- `runtime/` and `std/` remain separate projects; `build/` only resolves/compiles their selected dependencies.
- P0 must never require installed target components; P1 requires at least the Linux static profile.

## Completion / verification gates

- [ ] A P0 build is QZI-only; it never fails for lack of a native linker.
- [ ] P1 can compile all self-host packages and link a self-rebuilt `qz` using Quazi compiler/backend alone.
- [ ] Lockfile is dependency resolution; QZC is disposable compilation cache and cannot grant semantic correctness.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
