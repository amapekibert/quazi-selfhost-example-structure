# diagnostics — Implementation TODO

> **Package:** `quazi_diagnostics`  
> **Source:** `src/diagnostics/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Collect, classify and render compiler diagnostics. Errors are data; emitting them must not mutate compiler semantics.

**Integration boundary:** source gives ranges; frontend/semantic/modules/qzi/backend produce diagnostics; cli selects presentation.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/analysis.rs`
- `src/progress.rs`
- `src/semantic/unused.rs`
- `src/parser/mod.rs`
- `src/semantic/typecheck.rs`
- `src/semantic/borrow.rs`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Define severity, stable code/category, message, primary span, related labels and optional hint (`src/diagnostic.qz`).
- [ ] Represent multiple source labels with ranges and plain descriptions without guessing positions (`src/labels.qz`).
- [ ] Distinguish lex/parse/resolve/type/IO/QZI errors and return structured diagnostics to `pipeline`.
- [ ] Render `path:line:column`, source excerpt and a clear underlined region (`src/reporter.qz`).
- [ ] Support a portable plain-text reporter; do not require ANSI terminal capabilities.
- [ ] Collect more than one independent source error when parser/semantic can safely recover.
- [ ] Sort by file/offset and deduplicate repeat diagnostics; preserve deterministic order.
- [ ] Test diagnostic snapshots for missing input, malformed source, undeclared symbols and type mismatch.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Add ownership/move/borrow conflict diagnostics with labels for initial loan, move, use and invalid escape.
- [ ] Present generic instantiation, trait dispatch, ABI, invalid method receiver and QZI compatibility errors in the correct source context.
- [ ] Propagate compilation failure across `pipeline` and `cli` with stable nonzero exit codes.
- [ ] Maintain usable error messages when self-host builds span several imported packages.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Match relevant bootstrap diagnostic classes and codes (parser `E..`, semantic `S..`, warnings `W..`) where compatible.
- [ ] Implement actionable suggestions (`src/suggestions.qz`) with exact source edits, never automatic unsafe rewrites.
- [ ] Implement bootstrap-style unused-variable/parameter/import warnings, unreachable-code warnings, `@ignore` scopes and suppressions.
- [ ] Improve multiline labels, Unicode column alignment, context frames, error recovery and no-color/no-unicode output modes.
- [ ] Add cancellation-safe diagnostic collection, snapshot stability and consistency across parallel/incremental runs.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Expose structured machine-readable diagnostics and optional fix-its for language-server integrations.
- [ ] Consider diagnostic localization or richer explanation links only when tooling requires them.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/diagnostic.qz` | Structured error/warning type. |
| `src/labels.qz` | Source highlights and labels. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/reporter.qz` | Diagnostic formatting. |
| `src/suggestions.qz` | Diagnostic fix suggestions. |

## Repository boundary

- Produce reusable compiler diagnostics here. The separate `lsp/` repository owns protocol translation and editor integration; never embed LSP transport in this package.

## Completion / verification gates

- [ ] An error at a dependency file must identify that dependency, not the merged importer file.
- [ ] Fatal errors cannot be disguised as warnings or permit invalid native/QZI output.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
