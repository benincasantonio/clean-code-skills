---
name: clean-names
description: Apply Clean Code to names in Rust when identifiers hide domain meaning, confuse units or abstractions, or conceal effects. Use for naming and renaming functions, types, traits, fields, bindings, and modules.
---

# Clean Names in Rust

Names should reduce the mental work needed to understand behavior. Preserve
the project's domain vocabulary and compatibility when renaming.

## N1: Reveal intent

Replace names such as `proc`, `data2`, and `tmp` when the domain meaning is
known. Explain what the value represents, not just its Rust type.

| Unclear | Intent-revealing |
|---------|------------------|
| `d` for a retry delay | `retry_delay` |
| `process(records)` for validation | `validate_records(records)` |
| `n` for an upload limit | `max_upload_bytes` |

Short names inside a small, obvious expression can be entirely clear.

## N2: Name the appropriate abstraction

A caller asks for an account balance, not a particular map lookup.
Prefer `account_balance` to `get_hashmap_entry_balance` when storage is an
implementation detail. When storage itself is the contract, name it honestly;
an iterator, byte slice, or buffer need not masquerade as a domain object.

## N3: Use established vocabulary

Use domain terms consistently and Rust's conventional casing:
`snake_case` for functions and bindings, `UpperCamelCase` for types and traits,
and `SCREAMING_SNAKE_CASE` for constants.

Conversion names help a reader understand ownership and work:

- `as_` conventionally exposes a cheap borrowed view.
- `to_` conventionally performs a conversion without consuming a non-`Copy` input.
- `into_` conventionally consumes its receiver; it does not promise zero cost.

**After:** the names distinguish viewing a draft from consuming it.

```rust
struct Draft(String);

impl Draft {
    fn as_str(&self) -> &str {
        &self.0
    }

    fn into_text(self) -> String {
        self.0
    }
}

let draft = Draft(String::from("Ready for review"));
assert_eq!(draft.as_str(), "Ready for review");
assert_eq!(draft.into_text(), "Ready for review");
```

Follow standard `iter`, `iter_mut`, and `into_iter` naming when exposing those
operations. Simple getters usually omit `get_`; do not rename standard
operations such as `get(index)` that already have an established meaning.

## N4: Distinguish concepts and units

Avoid ambiguous pairs such as `first` and `second` when `source` and
`destination` express the relationship. Use names such as `timeout_seconds`
when a primitive's unit would otherwise be unclear, or a type such as
`Duration` when it fits the interface.

Distinguish a byte offset from a character position. A more verbose name
cannot repair a wrongly defined contract, so clarify the contract first.

## N5: Match detail to scope

Use enough detail to disambiguate longer-lived or widely visible names.
Conventional generic parameters (`T`, `E`, `K`, `V`) and lifetimes (`'a`,
`'src`) need not be expanded merely to avoid single letters.

A descriptive lifetime can clarify a relationship between several inputs;
an obvious local iterator binding rarely needs a long name.

## N6: Avoid redundant encodings

Prefer `users` to `vec_users` and `CustomerRepository` to
`ICustomerRepository`. Rust types already supply that information.

Unit suffixes and standard conversion names carry behavioral information;
do not strip them as if they were Hungarian notation. Keep meaningful
domain distinctions such as `CustomerId` and `OrderId`.

## N7: Reveal side effects

`get_next` can conceal removal from a queue; `pop_next` communicates it.
`load_or_create` communicates a possible write that `load` alone would hide.

**After:** the operation name and mutable receiver agree.

```rust
use std::collections::VecDeque;

struct WorkQueue {
    pending: VecDeque<String>,
}

impl WorkQueue {
    fn pop_next(&mut self) -> Option<String> {
        self.pending.pop_front()
    }
}

let mut queue = WorkQueue {
    pending: VecDeque::from([String::from("send receipt")]),
};
assert_eq!(queue.pop_next().as_deref(), Some("send receipt"));
assert_eq!(queue.pop_next(), None);
```

Before renaming, check callers, docs, tests, exported interfaces, and any
external format that depends on the name. Do not turn a naming cleanup into
an accidental compatibility break.

## Sources

- Robert C. Martin, [*Clean Code*](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), Chapters 2 and 17 (N1–N7).
- [Rust API Guidelines: naming](https://rust-lang.github.io/api-guidelines/naming.html), for conventions that communicate intent.
