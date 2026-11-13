# Quazi self-host compiler — proposed source layout

Design scaffold accompanying the agreed architecture. **This is not a working compiler.**
All `.qz` files contain comments describing their intended responsibilities. Manifests
and component contracts are illustrative until the self-host design is finalized.

## Rules

- `src/` contains separately reusable compiler libraries (no `packages/` directory).
- `qz` is a thin CLI facade. The actual compiler is composed from libraries.
- **QZI is the primary output of `qz build`**, not a disposable intermediate form.
- `src/chunks/` builds in-memory chunks from checked semantics. `src/qzi/` serializes/reads them;
  the native backend can consume chunks without first writing a `.qzi` file.
- A single optimized compilation mode. No O0/O1/O2/O3 profiles.
- Downloadable targets are **declarative compositions** of QZI components for
  architecture, OS, ABI, object format and linker. Component source lives in `targets/`.
- `.qzi` files are generated artifacts: **none are fabricated in this scaffold**.
- The existing Quazi stdlib/runtime remain external; do not rewrite for its own sake.
- P0: reproduce **all bootstrap behavior** in clean Quazi code and architecture.
- P1: independent self-host without the temporary bridge; P2: optimize;
  P3+: new features and ecosystem extensions.
- `qzup`, `lsp`, `tests`, `docs` live in separate repositories.
- No `bootstrap/` directory in the self-host repository.
- `specs/` holds *internal technical contracts*, not user-facing documentation.

## Quick tour

- `src/`: CLI, frontend, semantics, `chunks/`, QZI codec, optimizer,
  native backend API, JIT, target loading, build and diagnostics.
- `targets/`: implementations and downloadable target profiles.
- `specs/`: internal format and interface contracts.
- `examples/`: sample target manifest, CLI and installed toolchain layouts.
- `SOURCE_TREE.txt`: exhaustive list of the scaffold files.

See `examples/architecture.txt` and `examples/installed-toolchain-layout.txt`.
