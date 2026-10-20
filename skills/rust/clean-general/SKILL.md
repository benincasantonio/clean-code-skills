---
name: clean-general
description: Review or improve Rust code through Clean Code principles of cohesive responsibilities, explicit intent, reduced duplication, and controlled coupling. Use for tangled logic, scattered domain rules, or unnecessary abstractions.
---

# General Clean Code Principles in Rust

Judge structure by how easily a reader can understand and safely change it.
Rust mechanisms serve that goal; using more traits, types, or helpers is not
automatically an improvement.

## G5: Give domain knowledge one authoritative home

Consolidate repeated business rules, not every repeated line of syntax.
Two identical expressions may represent independent policies that should be
allowed to change separately. Conversely, different-looking calculations can
encode the same policy and drift apart.

Keep boundary semantics next to the policy (G33), and name units or use a
standard type to make constants meaningful (G25).

**After:** one policy determines when a pending invitation expires.

```rust
use std::time::Duration;

const INVITATION_VALIDITY: Duration = Duration::from_secs(24 * 60 * 60);

fn invitation_expired(age: Duration) -> bool {
    age >= INVITATION_VALIDITY
}

assert!(!invitation_expired(Duration::ZERO));
assert!(!invitation_expired(Duration::from_secs(86_399)));
assert!(invitation_expired(INVITATION_VALIDITY));
```

Reuse this policy in callers that mean invitation expiry. Do not apply it to
an unrelated cache simply because that cache also lasts a day. Ordinary
counts such as zero and one rarely need constants unless they encode a rule.

## G16: Make intent obvious

Prefer a clear loop or iterator expression to a clever expression whose
meaning requires decoding. Introduce a named intermediate value for a domain
concept; do not split every operator into a helper.

Understand the existing algorithm and boundaries before rewriting it (G21).
Maintain the same ordering, error behavior, and externally visible effects
unless the requested change explicitly calls for different behavior.

## G23: Match dispatch to the domain

The book's preference for polymorphism addresses scattered type-code
conditionals. Rust's enums can represent a closed set of alternatives and
place their behavior together without a hierarchy of objects.

**After:** the variants describe the domain and `match` handles every case.

```rust
enum Delivery {
    CollectAtDesk,
    Courier { tracking_number: String },
}

impl Delivery {
    fn instructions(&self) -> String {
        match self {
            Self::CollectAtDesk => String::from("Collect at the reception desk"),
            Self::Courier { tracking_number } => {
                format!("Track shipment {tracking_number}")
            }
        }
    }
}

assert_eq!(
    Delivery::CollectAtDesk.instructions(),
    "Collect at the reception desk"
);
assert_eq!(
    Delivery::Courier {
        tracking_number: String::from("AB123"),
    }
    .instructions(),
    "Track shipment AB123"
);
```

Use a trait when callers need substitutable behavior across implementations,
especially an open set. Do not create a trait and implementation for each
variant merely to eliminate a short, exhaustive `match`. Likewise, a simple
conditional is not a smell just because it branches.

## G30 and G34: Keep responsibilities and abstraction levels coherent

A function can contain several steps that accomplish one task. Extract a
helper when it names a meaningful subordinate operation or separates a
distinct responsibility.

For example, keep interpreting a record separate from opening a file or
choosing a retry policy when those details obscure each other. Use `Result`
and `?` to let the caller handle failure without burying the successful flow.
Preserve useful error context rather than returning an empty collection when
parsing fails (G26).

Do not introduce a wrapper for each expression or split a short function
solely to meet a line limit. A sequence of tiny, vague helpers can make the
reader work harder than a straightforward local flow.

## G36: Reduce knowledge of internal structure

Repeatedly reaching through another module's private representation creates
coupling. Place behavior with the type that owns the invariant, and expose
the smallest useful operation (G8, G14).

Dot count does not measure coupling. Iterator adapters and builders can form
a single readable operation.

**Acceptable:** this chain describes a transformation without exposing a
collaborator's internals.

```rust
fn nonempty_labels(labels: &[String]) -> Vec<&str> {
    labels
        .iter()
        .map(String::as_str)
        .filter(|label| !label.is_empty())
        .collect()
}

let labels = vec![
    String::from("ready"),
    String::new(),
    String::from("waiting"),
];
assert_eq!(nonempty_labels(&labels), ["ready", "waiting"]);
assert!(nonempty_labels(&[]).is_empty());
```

Public data records may deliberately expose their structure. Do not add
pass-through getters everywhere just to satisfy a literal "one dot" rule.

## Review checklist

- Identify the actual duplicated knowledge or responsibility.
- Preserve existing contracts, feature-gated behavior, and public APIs.
- Prefer private or restricted visibility for internal details; do not
  narrow an established public API as incidental cleanup.
- Validate empty inputs, limits, error paths, and other relevant boundaries.
- Explain how a proposed abstraction reduces the reader's work. If it does
  not, keep the simpler code.

For the full G1–G36 catalog, see
[rust-clean-code](../rust-clean-code/SKILL.md).

## Sources

- Robert C. Martin, [*Clean Code*](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), Chapters 3, 6–7, 10, and 17 (especially G5, G16, G23, G25, G30, G36).
- [The Rust Book: exhaustive matching](https://doc.rust-lang.org/book/ch06-02-match.html).
- [The Rust Book: iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html).
- [The Rust Reference: visibility](https://doc.rust-lang.org/reference/visibility-and-privacy.html).
