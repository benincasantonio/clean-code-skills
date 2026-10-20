---
name: boy-scout
description: Apply the Clean Code Boy Scout Rule when changing Rust code. Make small, relevant improvements to clarity and maintainability alongside the requested work, and consult focused clean-code skills as needed.
---

# The Boy Scout Rule in Rust

Leave the code easier to understand and maintain with small improvements
related to the task. Complete the requested behavior first; a cleanup should
help that work rather than expand its scope.

## A proportional workflow

1. Read the affected code, callers, and tests to understand its contracts.
2. Identify a concrete source of confusion or unnecessary maintenance work.
3. Make the smallest relevant improvement while preserving behavior and
   public compatibility.
4. Run the relevant checks and explain the improvement briefly.

If no useful cleanup is warranted, leave the surrounding code alone.
Do not manufacture a refactoring to meet a quota.

## Example: clarify an existing transformation

**Before:** redundant narration and vague local names add reading effort.

```rust
fn nonempty_labels(labels: &[String]) -> Vec<&str> {
    let mut r = Vec::new();
    // Loop through labels.
    for x in labels {
        if !x.is_empty() {
            r.push(x.as_str());
        }
    }
    r
}

let labels = vec![String::from("ready"), String::new()];
assert_eq!(nonempty_labels(&labels), ["ready"]);
```

**After:** the transformation reads directly, preserving the signature,
ordering, and empty-string behavior.

```rust
fn nonempty_labels(labels: &[String]) -> Vec<&str> {
    labels
        .iter()
        .map(String::as_str)
        .filter(|label| !label.is_empty())
        .collect()
}

let labels = vec![String::from("ready"), String::new(), String::from(" ")];
assert_eq!(nonempty_labels(&labels), ["ready", " "]);
assert!(nonempty_labels(&[]).is_empty());
```

This removes noise (C3) and clarifies intent (G16). The iterator chain is
acceptable under G36. A well-named loop would also be valid; the goal is
comprehension, not converting every loop.

## Choose the focused skill

| Task | Skill |
|------|-------|
| Review Clean Code concerns across a change | [rust-clean-code](../rust-clean-code/SKILL.md) |
| Clarify domain names or effects | [clean-names](../clean-names/SKILL.md) |
| Improve comments or API explanations | [clean-comments](../clean-comments/SKILL.md) |
| Simplify signatures or separate responsibilities | [clean-functions](../clean-functions/SKILL.md) |
| Reduce duplication or coupling | [clean-general](../clean-general/SKILL.md) |
| Make behavioral tests clearer and more reliable | [clean-tests](../clean-tests/SKILL.md) |

Load only what helps the current work. These skills express *Clean Code*
principles in Rust; they are not a replacement for general Rust guidance.

## Boundaries

- Preserve public names, signatures, and semantics unless their change is
  explicitly part of the task.
- Check feature-gated and downstream use before removing seemingly dead code.
- Keep meaningful documentation, safety explanations, and regression tests.
- Avoid speculative traits, builders, utility layers, or performance tuning.
- Respect the project's edition, minimum supported Rust version, and commands.

For reviews, describe the concrete maintenance problem with its rule ID.
For edits, summarize the actual improvement and validation; do not report a
style preference as a correctness defect.

## Sources

- Robert C. Martin, [*Clean Code*](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), Chapters 1 and 17.
- [The Rust Book: iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html), for the transformation used in the example.
