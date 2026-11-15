# frontend — Implementation TODO

> **Package:** `quazi_frontend`  
> **Source:** `src/frontend/src/`  
> **Global roadmap:** [`../../TODO.md`](../../TODO.md) — **P0/P1/P2/P3 mean exactly the same milestones here**.

## Responsibility and boundaries

Produce tokens and spanned AST from Quazi syntax. Syntax only: no name lookup, type/ownership decisions, QZI or target code generation.

**Integration boundary:** source → frontend → modules / semantic; parser passes exact AST spans and recoverable syntax diagnostics.

**Bootstrap reference files** (paths are relative to the separate Rust `quazistrap` repository; these files are *not* copied into self-host):

- `src/lexer/mod.rs`
- `src/lexer/token.rs`
- `src/parser/mod.rs`
- `src/parser/ast.rs`
- `src/parser/items.rs`
- `src/parser/common.rs`
- `src/parser/format.rs`

## Tasks by global phase

**Scope invariant:** implement existing bootstrap behavior only through P2; P3 items are proposals, not feature commitments. All checkboxes start unchecked because this archive contains no self-host implementation.

### P0 — QZI foundation

Quazi-written `.qz` → valid bootstrap-compatible `.qzi`; **no Quazi native codegen is required**.

- [ ] Establish token variants, literal payloads, keywords, punctuation and full source spans (`src/tokens.qz`).
- [ ] Lex whitespace, comments, identifiers, decimal integers, floats, boolean and string literals with offsets (`src/lexer.qz`).
- [ ] Implement quoted escape decoding and raw backtick strings with clear malformed-input errors.
- [ ] Implement `fn`, `import`, `pub`, `const`, `var`, `ret`, `if`, `else`, `for`, `match`, `break`, `continue`, `struct` in tokenization.
- [ ] Implement a lexer that advances on bad input and never loops forever; preserve parser-synthesized `&&`/`||` behavior if matching bootstrap tokens.
- [ ] Define spanned AST types for modules, declarations, statements, expressions, types, literals and patterns (`src/ast.qz`).
- [ ] Parse function declarations/signatures/blocks and import declarations (`src/declarations.qz`, `src/parser.qz`).
- [ ] Parse `const`/`var`, return, expression statements, nested blocks and `if` statements (`src/statements.qz`).
- [ ] Parse `for condition`, `for {}`, C-style `for`, range iteration and array iteration with break/continue placement.
- [ ] Implement expression precedence, parentheses, unary/binary operators, assignment and short-circuit logical syntax (`src/expression/precedence.qz`).
- [ ] Parse names, literals, parenthesized expressions, array/struct constructors (`src/expression/primary.qz`).
- [ ] Parse calls, arguments, indexes, field access, methods and postfix operators in one path (`src/expression/postfix.qz`).
- [ ] Parse `match` arms and patterns needed by P0 programs (`src/expression/patterns.qz`).
- [ ] Parse essential primitive, reference, pointer, array and named type forms without doing type resolution.
- [ ] Report expected-token/identifier/type errors at the right source location; reject truncated input deterministically.
- [ ] Export exactly one parser entry returning AST plus diagnostics (`src/lib.qz`).
- [ ] Differential-test AST shape and syntax acceptance against Rust lexer/parser for the P0 supported subset.

### P1 — native self-host

Minimum own native x86-64 Linux path and self-hosted compiler rebuild; **Rust is not used in the normal Quazi compilation chain**.

- [ ] Parse full self-host source signatures: generic `[T]` types/functions, `trait`, `impl`, methods, enums, aliases and generic instances.
- [ ] Parse explicit `self: &T`, `self: &T!` and consuming receivers, including method and trait-impl signatures.
- [ ] Parse `&`, `&!`, `*`, dereference, borrow expressions, ownership-relevant assignments and place syntax.
- [ ] Parse closures `|args| expression`, function type `fn(...) Return` and nested calls used by compiler sources.
- [ ] Parse guarded match arms and contextual constructors for `Option`/`Result` and enum variants.
- [ ] Parse `?`, tail return expressions and `if ... else ...` expression forms as supported by bootstrap.
- [ ] Parse named call arguments, variadic syntax where required, type casts and full type arguments.
- [ ] Parse attributes actually required by self-host, including `@cfg` and relevant intrinsic/inline markers.
- [ ] Test every syntax construct exercised by all Quazi compiler packages and standard/runtime imports.

### P2 — bootstrap parity and optimization

Finish already-implemented Rust-bootstrap capabilities, correctness and performance; do not confuse this with new language design.

- [ ] Reach all implemented bootstrap literal spellings: byte strings `b`/`br`, octal/hex/Unicode escapes and raw strings.
- [ ] Complete arithmetic/bitwise/shift/power syntax and `+=`/`-=`/`*=`/`/=`/`%=` plus prefix/postfix increment/decrement.
- [ ] Complete slicing `start:end:step`, omitted boundaries, negative indices and range variants.
- [ ] Parse `@repr(C)` structs/unions, packed/aligned structs, bitfields, flexible array members and field attributes.
- [ ] Complete `@api`, `@export`, `@syscall`, `@intrinsic`, `@format`, `@derive`, `@ignore`, `@test`, `@panic_handler`, `@opaque`, `@contiguous_elements` syntax.
- [ ] Complete C callbacks, foreign globals, variadics, `dyn Trait`, unsafe constructs and advanced destructuring patterns.
- [ ] Reproduce bootstrap import variants: dotted, grouped, alias, wildcard and relative-file imports.
- [ ] Add parser error recovery/resynchronization and fuzz/proptest cases for nesting, malformed tokens and operators.
- [ ] Ensure formatting parser/AST helpers cover the syntax supported by bootstrap formatter and header generator without embedding formatting policy here.

### P3 — future expansion

Optional future capabilities beyond the parity baseline; nothing here blocks P0/P1/P2 unless separately approved as a prerequisite.

- [ ] Add new language syntax only after a design decision and tests; avoid committing parser AST to hypothetical future constructs.
- [ ] If alternative syntaxes are introduced, keep language-independent source/semantic interfaces and add a separate parser adapter.

## File ownership (current scaffold; no code yet)

| `.qz` file | Responsibility |
| :--- | :--- |
| `src/ast.qz` | Abstract syntax tree structures. |
| `src/attributes.qz` | Attributes and compile-time markers. |
| `src/declarations.qz` | Declarations and imports. |
| `src/expression/patterns.qz` | Patterns and match-arm grammar used by match expressions; no semantic exhaustiveness checks. |
| `src/expression/postfix.qz` | Postfix parsing for calls, indexing, field/method access and explicit type arguments. |
| `src/expression/precedence.qz` | Expression operator precedence and associativity parsing; binary/range/assignment operator chains. |
| `src/expression/primary.qz` | Primary expressions: literals, names, parenthesized expressions, constructors and closures. |
| `src/expressions.qz` | Expression syntax. |
| `src/lexer.qz` | Lexing source to tokens. |
| `src/lib.qz` | Public library entrypoint / explicit exports. |
| `src/parser.qz` | Parsing tokens into syntax structures. |
| `src/statements.qz` | Statement syntax. |
| `src/tokens.qz` | Token kinds and payloads. |

## Repository boundary

- Lexer, parser and AST stay language-specific here; no native codegen, runtime implementation or language-server protocol handling.

## Completion / verification gates

- [ ] Every supported token/AST node retains source ranges needed for diagnostics.
- [ ] No invalid syntax reaches semantic analysis as a silently fabricated valid node.
- [ ] Self-host compiler `.qz` files parse from source before the P1 compiler bootstrap gate.

**Evidence for marking tasks done:** add/identify the relevant Quazi unit or integration test, compiler output and, where semantics/ABI/format matter, a differential run against the Rust bootstrap. Do not mark a phase complete because the file exists.
