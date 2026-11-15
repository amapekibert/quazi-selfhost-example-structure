# modules — Implementation TODO

> **Package:** `quazi_modules`  
> **Source:** `src/modules/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Resolve source module names, imports, visibility and external package namespaces. Never own manifest fetching/build-cache policy.

**Integration boundary:** build provides resolved dependency roots; modules returns graph/source units to frontend/semantic and imports QZI interfaces.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/loader.rs`
- `src/project.rs`
- `src/package.rs`
- `src/bytecode/interface.rs`
- `docs/MODULES_AND_PACKAGES.md`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Define module identity, qualified names, visibility and resolved file path representation (`src/packages.qz`).
- [ ] Discover a single source entry and its direct/transitive local `.qz` imports (`src/resolver.qz`).
- [ ] Resolve `foo.qz`, `foo/mod.qz` and basic dotted imports relative to their importing module (`src/imports.qz`).
- [ ] Canonicalize paths enough to detect duplicate modules without losing human-readable import identities.
- [ ] Build an import graph with deterministic order and explicit cycle errors (`src/graph.qz`).
- [ ] Distinguish user source from injected runtime/prelude/library files for diagnostics and token accounting.
- [ ] Make a minimal compiler root/package namespace for P0 builtins and prelude names required by sample programs.
- [ ] Export module bindings and source units without concatenating identities or losing original file spans (`src/lib.qz`).
- [ ] Test nested local imports, missing module, duplicate import, cycles, relative path and private item error.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Resolve all self-host library packages as separate import roots with explicit dependency aliases.
- [ ] Support public/private items, `pub import` gateways, cross-package visibility and qualified references.
- [ ] Resolve/inject the separately versioned `runtime/` (prelude) package, based on package settings, without double declarations or import recursion.
- [ ] Resolve multiple entry points, std library imports, generic declarations and shared module dependencies in compiler build.
- [ ] Bind already-produced QZI dependency public interfaces for source consumers without needing their original source.
- [ ] Ensure symbol identity survives same-named types/functions in different packages and generic contexts.
- [ ] Test real self-host module graph, including `cli`, `pipeline`, `semantic`, `chunks`, `qzi` and native packages.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Reach bootstrap import variants: grouped items, wildcards, aliases, relative paths and alias-collision detection.
- [ ] Support project sources, local file dependencies, Git/archive/source/QZI dependencies and namespace checks exposed by build package.
- [ ] Materialize public QZI interface gateways and private implementation dependencies without leaking hidden API.
- [ ] Reproduce `@cfg` target-dependent import filtering, recursive dependencies, version identity and import-cycle diagnostics.
- [ ] Add overlays, cancellation checkpoints, dependency tree diagnostics and deterministic graph walks for tooling.
- [ ] Differential-test complex bootstrap loader cases, QZI libraries and multiple artifact imports.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Support richer compiled-module distribution or alternative source providers only with a specified module-identity model.
- [ ] Keep import resolution independent from target package delivery and compiler UI frontends.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/graph.qz` | Dependency ordering and cycles. |
| `src/imports.qz` | Import semantics. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/packages.qz` | Package references and namespace resolution. |
| `src/resolver.qz` | Resolve modules and dependencies. |

## Repository boundary

- `modules/` loads project and source module graphs; neither `runtime/` implementation nor `lsp/` services live here.
- Keep dependency imports of external `std/`, `runtime/` and target package QZI artifacts explicit and cycle-safe.

## Completion / verification gates

- [ ] The same qualified symbol resolves identically whether dependency source or QZI interface was loaded.
- [ ] Cycles/missing imports abort cleanly; duplicate paths do not load or analyze a module twice.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
