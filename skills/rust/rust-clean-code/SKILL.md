---
name: rust-clean-code
description: Apply Robert C. Martin's Clean Code principles when writing, reviewing, or refactoring Rust. Focus on meaningful names, cohesive functions, useful comments, duplication, coupling, clear error paths, and behavioral tests.
---

# Clean Code in Rust: Complete Reference

Apply the intent of *Clean Code*, first edition, especially Chapter 17's
smells and heuristics, to Rust. The C/E/F/G/N/T identifiers retain their
relationship to the book; the wording below is deliberately contextualized.
The book's Java-specific J rules are not a separate Rust ruleset.

This track concerns readability, maintainability, and disciplined refactoring.
It complements general Rust guidance such as
[rust-skills](https://github.com/leonardomso/rust-skills), but does not require
another skill. Do not expand a cleanup into a language tutorial, optimization
project, or framework migration.

Respect the target project's edition, minimum supported Rust version, and
public contracts. The examples in this track use edition 2024 and work with
Rust 1.90.0; this is an example baseline, not a required project upgrade.

## Comments (C1-C5)

- C1: Keep change history in version control. Preserve legal notices and links that explain a technical constraint.
- C2: Update or remove comments when their explanation no longer matches the code.
- C3: Remove comments that merely repeat syntax; retain intent, contracts, and non-obvious invariants.
- C4: Write concise, accurate rustdoc and explanations. Describe relevant errors, panics, and safety obligations.
- C5: Remove abandoned commented-out implementations; executable documentation examples are not dead code.

See [clean-comments](../clean-comments/SKILL.md) for examples.

## Environment (E1-E2)

- E1: Make the documented build reproducible with one project command, usually `cargo build`; keep required features and setup discoverable.
- E2: Make the documented test suite easy to run, usually `cargo test`; document any separate integration or feature-specific suites.

Use the project's existing commands rather than replacing its workflow.

## Functions (F1-F4)

- F1: Reduce the concepts a caller must juggle. Group related parameters when they form a domain concept; do not enforce an arbitrary count or introduce a builder just to shorten a signature.
- F2: Prefer returning computed results. Intentional mutation through `&mut` or a writer can clearly express the operation; do not clone data merely to avoid an output parameter.
- F3: Avoid flags that hide unrelated operations. Use named operations or a meaningful mode type when that clarifies the call; ordinary boolean domain data is legitimate.
- F4: Remove genuinely unused functions after checking callers, tests, feature gates, and downstream public API commitments.

See [clean-functions](../clean-functions/SKILL.md).

## General (G1-G36)

- G1: Avoid mixing unrelated languages or responsibilities in a source file. Rustdoc examples and necessary embedded queries are legitimate.
- G2: Honor the behavior callers reasonably expect, including documented error and mutation contracts.
- G3: Handle relevant empty inputs, invalid values, numeric limits, and text boundaries deliberately.
- G4: Preserve compiler checks and validated invariants. Do not suppress diagnostics or introduce unsafe code just to make a refactoring pass.
- G5: Give each piece of domain knowledge one authoritative home; similar syntax alone does not justify an abstraction.
- G6: Keep a module's public concepts at a coherent abstraction level.
- G7: Keep shared abstractions independent of particular implementations. Rust traits should not depend on downcasting to specific implementors.
- G8: Expose only what callers need. Use private or restricted visibility to protect invariants, while preserving established public APIs.
- G9: Remove dead code and stale paths after checking conditional compilation and external users.
- G10: Keep bindings close to their use, with scopes that make dependencies and resource lifetimes easy to see.
- G11: Use consistent vocabulary, error conventions, and patterns within the project.
- G12: Remove clutter such as redundant wrappers, unused imports, and stale scaffolding.
- G13: Avoid artificial coupling through unrelated traits, shared utility modules, or global state.
- G14: Put behavior near the data and invariants it needs; avoid repeatedly reaching into another module's internals.
- G15: Avoid selector parameters that multiplex unrelated jobs; make the chosen operation clear at the call site.
- G16: Make intent visible. Prefer a straightforward loop or iterator expression over cleverness.
- G17: Put code where a reader following the domain would expect to find it.
- G18: Use methods for behavior belonging to a type; keep independent transformations as free functions rather than inventing receiver objects.
- G19: Name intermediate concepts when doing so explains an expression; skip bindings that merely rename an obvious value.
- G20: Make function names and signatures agree with their effects and failure behavior.
- G21: Understand the algorithm and its edge cases before simplifying it.
- G22: Make dependencies explicit through module paths, parameters, and declared Cargo dependencies.
- G23: Choose dispatch to fit the domain. Use enums and exhaustive `match` for a closed set of alternatives; use traits when substitutable behavior is needed.
- G24: Follow project conventions, rustfmt, and relevant Clippy diagnostics. Respect the project's edition and minimum supported Rust version.
- G25: Name domain constants and use meaningful units; do not give every literal a ceremonial constant.
- G26: State contracts precisely, including units, ranges, failure conditions, and mutation.
- G27: Encode important invariants in types and construction boundaries when that removes reliance on caller convention.
- G28: Name complex domain predicates when they deserve a concept; do not extract every condition.
- G29: Avoid double negatives; a simple early rejection such as `if !valid` can still be clearest.
- G30: Give each function one coherent responsibility. An extractable expression alone does not establish a second responsibility.
- G31: Make required ordering visible through arguments, construction, or explicit state transitions; add state types only when they protect a real protocol.
- G32: Let domain meaning and existing conventions drive structure rather than arbitrary limits or patterns.
- G33: Centralize boundary rules so callers do not duplicate offsets, range calculations, or validation.
- G34: Keep the steps within a function at a consistent abstraction level; separate orchestration from substantial parsing or persistence details.
- G35: Supply configuration at an appropriate boundary rather than hiding environment reads inside domain logic.
- G36: Avoid reaching through unstable implementation details. Readable iterator and builder chains are not violations based on their dot count.

See [clean-general](../clean-general/SKILL.md) for worked examples.

### Error handling as clear control flow

The book's preference for exceptions over error codes aims to keep failures
from obscuring normal logic. In Rust, use `Result` and `?` for recoverable
failures, and `Option` for meaningful absence. Preserve useful context for the
caller instead of silently converting failure into an empty value.

Separate substantial error recovery or I/O from domain computation when they
are different responsibilities (G20, G26, G30, G34). Do not invent an exception
mechanism or require a new error-handling crate. Panics can express broken
internal invariants; tests may use `expect` to fail clearly. Treat these as
contract decisions rather than banning every occurrence of `unwrap`.

## Names (N1-N7)

- N1: Reveal the domain meaning of a value or operation.
- N2: Name the abstraction the caller uses, not incidental storage details.
- N3: Use domain terms and established Rust vocabulary, including conversion and iterator naming conventions.
- N4: Distinguish concepts and units unambiguously.
- N5: Match name detail to scope; conventional `T`, `E`, and `'a` remain appropriate.
- N6: Avoid redundant type encodings and interface prefixes; meaningful units and Rust's `as_`, `to_`, and `into_` conventions are not Hungarian notation.
- N7: Reveal effects such as creating, consuming, persisting, or clearing data.

See [clean-names](../clean-names/SKILL.md).

## Tests (T1-T9)

- T1: Test observable behavior and plausible failure modes, not facts already guaranteed by the type system.
- T2: Use available coverage reports to find missing scenarios; executed lines do not prove useful assertions.
- T3: Keep inexpensive tests of important defaults and contracts even when their implementation is simple.
- T4: Resolve ignored tests: fix defects or document the external requirement and how to run the test; do not delete a required behavior test to hide failure.
- T5: Exercise the relevant input, state, numeric, and error boundaries.
- T6: Add a regression test and nearby cases that share the cause of a bug.
- T7: Look for common causes across failures before patching each symptom.
- T8: Use coverage gaps to investigate missing scenarios or awkward dependency boundaries.
- T9: Keep feedback fast and deterministic; distinguish unit tests from slower integration suites rather than imposing a universal time limit.

Use one behavior per test, with as many related assertions as that behavior
needs. See [clean-tests](../clean-tests/SKILL.md).

## Applying the rules

For a review, identify the concrete maintenance problem and relevant rule
identifier, then propose the smallest useful change. Do not report violations
based only on line counts, argument counts, or dots.

For an edit, preserve behavior and compatibility, validate the affected paths,
and briefly explain the improvement. Load focused skills only when their
examples help the task. Avoid speculative traits, builders, wrappers, or
unrelated cleanups.

## Sources

- Robert C. Martin, [*Clean Code*, first edition](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), especially Chapters 2–7, 9–10, and 17. These are contextualized summaries, not quotations.
- [The Rust Book: references and borrowing](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html), for explicit mutation contracts.
- [The Rust Book: when to panic](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html), for failure contracts.
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/), for language conventions that support clear interfaces.
