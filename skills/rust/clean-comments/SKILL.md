---
name: clean-comments
description: Apply Clean Code when writing or reviewing Rust comments and rustdoc. Remove redundant or stale prose while preserving rationale, public contracts, legal notices, and explanations of safety invariants.
---

# Clean Comments in Rust

Use comments to communicate information a reader cannot reliably infer from
the code. Clear names and structure reduce the need for narration; they do
not replace API contracts or explanations of constraints.

## C1: Keep information where it belongs

Keep author names and change logs in version control. Retain required legal
notices and references that explain why a workaround or compatibility
constraint exists. An issue link with a technical explanation can be useful;
a ticket number by itself is not an explanation.

## C2: Keep explanations current

When changing behavior, update affected rustdoc, examples, and comments.
Remove explanations of obsolete code. Do not retain a comment simply because
it was once accurate.

## C3: Remove narration, preserve intent

**Before:** the comment merely repeats the operation.

```rust
let mut displayed_line = 0;
displayed_line += 1; // Increment the line.
assert_eq!(displayed_line, 1);
```

**After:** naming makes the conversion clear; the comment records its reason.

```rust
fn displayed_line_index(zero_based_index: usize) -> Option<usize> {
    // The editor protocol numbers displayed lines from one.
    zero_based_index.checked_add(1)
}

assert_eq!(displayed_line_index(0), Some(1));
assert_eq!(displayed_line_index(usize::MAX), None);
```

Preserve a `SAFETY:` explanation that states why an unsafe operation's
preconditions hold. A statement such as "this is safe" supplies no reasoning.
The task here is accurate explanation, not a general unsafe-code audit.

## C4: Document the caller's contract

Use `///` for item documentation and `//!` for module-level context when
needed. Explain behavior, units, boundaries, and meaningful failure cases.
Include applicable `# Errors`, `# Panics`, and `# Safety` sections; do not
add empty sections merely to satisfy a template.

**After:** rustdoc captures a non-obvious inclusive limit.

```rust
/// Returns whether the payload fits within the byte limit.
///
/// A payload exactly equal to the limit is accepted.
fn fits_payload_limit(payload: &[u8], limit_bytes: usize) -> bool {
    payload.len() <= limit_bytes
}

assert!(fits_payload_limit(b"abc", 3));
assert!(!fits_payload_limit(b"abcd", 3));
```

For public APIs, add concise runnable examples when they clarify usage.
Keep examples synchronized with the contract and validate them with
`cargo test --doc`. A useful rustdoc example is executable documentation,
even though its source appears inside a comment.

Do not remove documentation solely because the function body is short.
Conversely, a private helper with an obvious name does not need a doc comment
that repeats that name in a sentence.

## C5: Remove abandoned implementations

Delete commented-out implementations and debug leftovers. Version control
preserves old code. Distinguish these from explanatory examples, supported
configuration samples, and intentional compile-fail documentation.

## Review questions

- Does the comment explain a decision, constraint, or contract?
- Is it still true after the change?
- Could a clearer name remove the need for this comment?
- Would deleting it lose information about behavior or safety?

## Sources

- Robert C. Martin, [*Clean Code*](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), Chapters 4 and 17 (C1–C5).
- [Rust API Guidelines: documentation](https://rust-lang.github.io/api-guidelines/documentation.html).
- [The rustdoc book: documentation tests](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html).
