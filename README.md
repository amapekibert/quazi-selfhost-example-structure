# Quazi

**The Quazi compiler, written in Quazi.**

Quazilang Selfhost is the next implementation of the Quazi compiler, developed in Quazi itself. The project follows an incremental path: first produce QZI using a Quazi-written compiler, then replace the Rust bootstrap's native backend with a native backend written in Quazi.

## Development Status

**Plan completion: 0.0%.**  
**Remaining work: 100.0%.**

**Current phase:** P0 — QZI Compiler Foundation  
**Implementation status:** Initial source structure; implementation pending.

| P0 task | Completed | Progress |
| :--- | ---: | ---: |
| Workspace, compiler entry point & build workflow | 0/1 | 0.0% |
| Source handling, lexer, parser & AST | 0/1 | 0.0% |
| Name resolution, type checking, semantic validation & diagnostics | 0/1 | 0.0% |
| Lowering & QZI generation | 0/1 | 0.0% |
| QZI compatibility with the Rust bootstrap | 0/1 | 0.0% |
| End-to-end compilation & behavior verification | 0/1 | 0.0% |
| **P0 total** | **0/6** | **0.0%** |

The table tracks the **6 P0 tasks** in [`TODO.md`](TODO.md); the overall plan completion above covers all **24 roadmap tasks** (P0–P3). These percentages describe completion of the *selfhost roadmap*, not the functionality of the Quazi language or the status of the existing Rust bootstrap.

## Compilation Strategy

### P0 — Quazi frontend, Rust backend

```text
source.qz
    │
    ▼
Quazi Selfhost
  frontend → semantic analysis → lowering → QZI emission
    │
    ▼
program.qzi
    │
    ▼
Quazistrap (Rust bootstrap)
  QZI → native code → linking
    │
    ▼
executable
```

P0 focuses exclusively on producing compatible `.qzi` artifacts. Native code generation and linking remain the responsibility of the existing Rust bootstrap.

### P1 — Independent self-hosting

```text
source.qz
    │
    ▼
Quazi Selfhost
  frontend → semantic analysis → lowering → QZI
    │
    ▼
Quazi native backend
  QZI → native code → linking
    │
    ▼
executable
```

P1 introduces the native backend in Quazi and establishes a compiler that can rebuild itself without relying on the Rust backend. Full bootstrap feature parity is reserved for P2.

## Roadmap

Development is organized into four priorities:

- **P0 — Foundation:** A working Quazi-written `.qz → .qzi` compiler, with native binaries produced through Quazistrap.
- **P1 — Self-hosting:** Essential compiler functionality, native code generation, linking, and a verified self-rebuild.
- **P2 — Parity & optimization:** Complete the remaining bootstrap capabilities and improve performance, diagnostics, and reliability.
- **P3 — Expansion:** Extend existing limitations and introduce major new language, compiler, and toolchain capabilities.

For the complete task list and milestone criteria, see **[TODO.md](TODO.md)**.

## License

Licensed under the **GNU Affero General Public License v3.0**. See [LICENSE](LICENSE).
