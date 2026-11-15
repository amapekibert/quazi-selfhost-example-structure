# source — Implementation TODO

> **Package:** `quazi_source`  
> **Source:** `src/source/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Own source-file identity, immutable text, spans and position mapping. Never parse syntax or resolve imports here.

**Integration boundary:** source → frontend / diagnostics / modules; consumers use the same file IDs and half-open span convention.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/loader.rs`
- `src/lexer/token.rs`
- `src/parser/ast.rs`
- `src/analysis.rs`
- `src/lsp/document.rs`
- `src/lsp/span.rs`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Define stable file IDs, source buffers, byte offsets and half-open spans; document UTF-8 offset conventions (`src/source.qz`, `src/spans.qz`).
- [ ] Open a `.qz` path, read its bytes, retain original contents and canonical display path without destructive normalization (`src/files.qz`).
- [ ] Supply source slices to the lexer and expose the complete file text without repeated copies (`src/source.qz`, `src/files.qz`).
- [ ] Resolve `line:column` from byte spans with predictable treatment of LF, CRLF and EOF (`src/locations.qz`).
- [ ] Preserve token, AST and diagnostics provenance across several input files using shared source IDs.
- [ ] Detect unreadable/missing files, invalid spans and invalid UTF-8 consistently and report failure rather than panic.
- [ ] Offer the narrow public interface required by lexer, parser and reporter (`src/lib.qz`).
- [ ] Test empty input, one-line file, trailing newline, CRLF, Unicode offsets, multi-file spans, EOF errors and missing files.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Support project/module-level source files with stable identities through import expansion and library/prelude injection.
- [ ] Keep span origins intact across generic instantiation, library imports and expanded/derived source constructs required for self-hosting.
- [ ] Ensure source handles remain valid for the entire compile session without retaining unnecessary duplicate buffers.
- [ ] Test diagnostic paths from compiler source files distributed across packages and imported modules.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Match bootstrap loader span and source-mapping behavior for nested modules and generated source fragments.
- [ ] Support source overlays/unsaved buffers and consistent incremental invalidation, as the bootstrap loader uses for LSP.
- [ ] Improve memory use for large projects, repeated sessions and cancellation without changing public source IDs.
- [ ] Differential-test byte/line/column ranges against Rust diagnostics, including unusual Unicode and malformed input.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Consider source maps or virtual files for future macros/alternative syntaxes only when a concrete new frontend needs them.
- [ ] Provide extensible source-provider APIs for non-filesystem inputs without coupling the compiler to an editor service.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/files.qz` | File loading and tracking. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/locations.qz` | Line/column mapping. |
| `src/source.qz` | Source abstraction and IDs. |
| `src/spans.qz` | Byte spans in source. |

## Repository boundary

- Source management is compiler infrastructure; external `lsp/` may reuse it through an API. Do not implement LSP transport or language-server features in `quazi`.

## Completion / verification gates

- [ ] Every diagnostic label round-trips to the exact source slice, including imported files.
- [ ] Out-of-range offsets never crash or point into another source file.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
