# AI Usage Policy

## 1. Purpose and Scope

This policy defines how artificial intelligence (AI) tools may be used when contributing to or maintaining this project. It applies to all contributors, maintainers, commits, and pull requests.

AI is an **assistance tool, not a substitute for engineering judgment**. Its primary role is to help developers analyze code, identify defects, evaluate possible improvements, and automate low-risk tasks. AI involvement must remain minimal, deliberate, and subject to human control—especially in the compiler and other architecture-critical components.

**The developer submitting a change is fully responsible for every line of that change, regardless of whether AI was involved.**

## 2. Permitted Uses

The following uses of AI are allowed, subject to the mandatory requirements in Section 4.

### 2.1. Analysis and Review

- Analyzing existing code to explain behavior, dependencies, invariants, and potential failure cases.
- Reviewing code for bugs, correctness issues, security risks, performance problems, and violations of established project conventions.
- Suggesting fixes, alternative approaches, and test cases for developer evaluation.
- Identifying edge cases, missing error handling, and gaps in test coverage.
- Helping investigate compiler errors, regressions, and unexpected behavior.

AI findings are **suggestions, not verified facts**. Developers must reproduce or otherwise substantiate reported problems before acting on them.

### 2.2. Low-Risk Automation and Maintenance

AI may assist with routine work that does not directly define or change compiler behavior, including:

- Writing or maintaining test runners, test infrastructure, CI helpers, and development scripts.
- Formatting documentation, improving wording, correcting typos, and organizing Markdown or other non-executable project files.
- Preparing repetitive configuration, templates, and other auxiliary tooling.
- Automating mechanical tasks with clearly defined inputs and expected outputs.

Such changes must still be reviewed, tested where applicable, and checked for unintended side effects. A file is **not** automatically low-risk merely because it is outside the compiler directory.

### 2.3. Targeted Code Changes and Optimization

AI may propose patches or optimizations, including for compiler code, **only as material for a developer to evaluate and verify**.

A proposed change may be accepted only when:

- The developer understands the full change and can explain why it is correct.
- It complies with the project's coding standards, design principles, and existing architecture.
- It preserves intended language semantics, compiler invariants, and observable behavior unless an intentional behavior change has been separately reviewed and approved.
- It is narrowly scoped and does not introduce unnecessary complexity, abstractions, or dependencies.
- Relevant tests pass, including regression tests for the issue being addressed.
- Any claimed performance or resource-usage benefit is demonstrated by appropriate measurements, not assumed from AI output.

A performance optimization without a demonstrated benefit or a justified engineering trade-off must not be accepted.

## 3. Prohibited Uses

The following practices are **strictly prohibited**:

1. **Unreviewed AI-generated code.** Committing, approving, or merging AI-produced or AI-modified code without a complete, independent human review.
2. **Delegating core engineering decisions.** Allowing AI to independently determine language semantics, compiler architecture, IR design, type-system rules, code generation strategy, or other fundamental technical decisions.
3. **Uncontrolled core modifications.** Allowing AI agents to autonomously apply or merge changes to compiler-critical components without explicit developer direction and full verification.
4. **Blind acceptance of fixes.** Applying suggestions without understanding the root cause, the proposed implementation, and its potential impact.
5. **Violating project standards.** Accepting changes that conflict with established coding rules, architectural constraints, compiler invariants, or documented project conventions.
6. **Unnecessary rewrites or refactors.** Introducing broad AI-generated rewrites, speculative abstractions, new dependencies, or unrelated modifications without a concrete, reviewed technical justification.
7. **Unverified optimizations.** Accepting performance claims without reproducible evidence, or trading away correctness, maintainability, or architectural consistency for speculative speed gains.
8. **Bypassing quality controls.** Skipping required tests or reviews; weakening, deleting, or manipulating tests, diagnostics, or checks solely to make AI-assisted changes appear correct.
9. **Fabricating evidence.** Presenting AI-generated explanations, benchmark results, test outcomes, references, or security assessments as verified when they have not been independently checked.
10. **Unauthorized disclosure.** Sending credentials, tokens, private data, unpublished security-sensitive information, or restricted project materials to an external AI service without explicit authorization.
11. **Unverified external material.** Copying AI-suggested third-party code, dependencies, or licensed content without checking provenance, compatibility, and applicable license obligations.

## 4. Mandatory Requirements for AI-Assisted Changes

Before submitting a commit or pull request containing AI-assisted changes, the submitting developer **MUST**:

1. **Review the entire change manually.** Inspect the final diff line by line, including generated scripts, configuration, tests, and documentation where applicable.
2. **Verify project compliance.** Confirm that the change follows all existing coding standards and architectural rules. This policy does not replace those rules.
3. **Verify correctness and scope.** Understand the implementation, confirm its necessity, and ensure it does not alter unrelated behavior or introduce unnecessary changes.
4. **Validate the result.** Run all relevant checks, tests, and benchmarks where applicable. Add or update regression tests when fixing a bug or changing behavior.
5. **Check safety and dependencies.** Review security implications, external dependencies, licensing concerns, and any destructive or privileged operations introduced by scripts or tooling.
6. **Disclose material AI involvement.** If AI-generated or AI-modified content is included, briefly identify the affected areas and describe how the change was verified in the pull request description.

**AI output is never evidence of correctness on its own.** A successful build or passing test suite also does not remove the obligation to review the change against the project's architecture and coding rules.

If the developer cannot fully understand or verify an AI-assisted change, **the change must not be submitted**.

## 5. Review and Enforcement

Maintainers may request additional explanation, tests, benchmarks, or revisions for any AI-assisted contribution. AI involvement does not reduce the normal standard of review.

**Zero-tolerance rule:** If **any** prohibited practice in Section 3 is violated, the affected commit or pull request **MUST be rejected**, even if the resulting code compiles or passes tests. The violation must be corrected before the change can be submitted again.

A contribution that fails any mandatory requirement in Section 4 must likewise be rejected until it is compliant.

## 6. Guiding Principle

> **AI may assist with analysis, verification, and routine automation. It must never replace the developer's understanding, responsibility, or control over the codebase.**

When in doubt, prefer a smaller change, independent reasoning, and direct verification over AI-generated implementation.
