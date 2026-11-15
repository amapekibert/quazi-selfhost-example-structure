# cli — Implementation TODO

> **Package:** `quazi_cli`  
> **Source:** `src/cli/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Own user-facing `qz` command parsing, dispatch, output and exit codes. Package name is `quazi_cli`; native command stays `qz`.

**Integration boundary:** qz CLI parses/options → build/pipeline/target and reports diagnostics; it must not implement compiler algorithms.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/cli.rs`
- `src/main.rs`
- `src/progress.rs`
- `src/test_runner.rs`
- `src/docs.rs`
- `src/header.rs`
- `src/apidoc.rs`
- `src/lsp/mod.rs`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Define `qz` binary entry and argument parser (`src/main.qz`, `src/cli.qz`); emit usage and version.
- [ ] Implement `qz build file.qz` to write QZI by default, with explicit output path (`src/commands/build.qz`).
- [ ] Implement `qz check file.qz` to run parse/type/semantic phases without generating artifacts (`src/commands/check.qz`).
- [ ] Support single-file and minimal project build invocation without requiring an installed native backend.
- [ ] Print actionable compilation failures with file/line diagnostics and meaningful nonzero exit status (`src/output.qz`).
- [ ] Keep human-readable, deterministic progress output optional, with quiet/plain output for tests and terminals.
- [ ] Provide clear error for unsupported flags/commands instead of silently invoking Rust-native operations.
- [ ] Test help/version, invalid options, missing file, valid QZI build, malformed source, check-only and output naming.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Extend `qz build` to explicit native executable/object output using Quazi's own backend, preserving QZI default.
- [ ] Implement `qz run` to build and run a native executable in the P1 self-host environment (`src/commands/run.qz`).
- [ ] Support project builds, self-host package selection, target Linux x86-64 and native output names.
- [ ] Provide `qz clean` for generated project output (`src/commands/clean.qz`).
- [ ] Preserve normal self-host rebuild flow from command invocation through linker without spawning Rust backend.
- [ ] Ensure error/exit propagation through child native programs, compiler errors and failed linking.
- [ ] Test CLI against P1 self-host executable running its own build command, including clean/rebuild.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Match bootstrap `build`/`run`/`check`/`test` behavior including filters, libs/bins, bytecode, obj, native and output flags.
- [ ] Add `fetch`, `deps`, `add`, `remove`, `new`, `init` project and package commands through build APIs.
- [ ] Add optional `fmt`, `doc`, `header`, `lsp` command dispatch only by forwarding to external dedicated tools (including separate `lsp/` repository); no LSP server implementation in `quazi`.
- [ ] Implement debugger/disassembly and `info` inspection (`src/commands/info.qz`) where supported by bootstrap.
- [ ] Support `--target`, `--linker`, `--linker-script`, `--static-lib`, `--shared-lib`, `-L`, `-l`, `--strip` and `--no-incremental` consistently.
- [ ] Match bootstrap flags `--silent`, `--no-color`, `--no-unicode`, `--no-progress`, `-o`, `-q`, `--bin`, `--lib` and help text.
- [ ] Implement native process args, stdout/stderr behavior, pass-through errors and expected exit status on every command.
- [ ] Provide tests for multiple-artifact projects, invalid option pairs, Windows/NetBSD target names and cross-compile options.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Expose JIT/in-memory QZI execution (`qz run --jit`) only when the JIT module is prioritized and verified.
- [ ] Consider new CLI subcommands only to expose implemented features; keep CLI stable and avoid redundant flags.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/cli.qz` | Parse CLI options and route the supported qz commands. |
| `src/commands/build.qz` | Build command, default emission: QZI. |
| `src/commands/check.qz` | Analyze and validate without native code generation. |
| `src/commands/clean.qz` | Remove generated project artifacts. |
| `src/commands/info.qz` | Query compiler, package and target metadata. |
| `src/commands/run.qz` | Run QZI using runtime or native path. |
| `src/main.qz` | Entry point / CLI dispatch. |
| `src/output.qz` | Human-readable compiler output. |

## Repository boundary

- `cli/` owns `qz` commands and flags, not target implementation, runtime methods, or an LSP server. Default QZI output must work without installed native target packages.

## Completion / verification gates

- [ ] Default `qz build` in self-host emits QZI, not ELF/PE; native output is an explicit path/mode in P1.
- [ ] CLI should never need to know which package implements QZI instruction encoding or borrow conflicts.
- [ ] The emitted executable is named `qz`, even though its internal package is `quazi_cli`.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
