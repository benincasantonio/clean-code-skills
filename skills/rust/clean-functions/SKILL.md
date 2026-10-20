---
name: clean-functions
description: Apply Clean Code to Rust functions and methods when signatures, mixed responsibilities, mode flags, or hidden effects make code difficult to understand. Preserve explicit mutation and public contracts.
---

# Clean Functions in Rust

Apply *Clean Code*'s function heuristics to caller comprehension, rather than
enforcing a particular signature shape. Keep one coherent responsibility and
one abstraction level per function (G30, G34).

## F1: Reduce argument complexity

Ask whether the parameters describe a single meaningful concept. A long list
can expose a missing concept; a four-argument function is not automatically
wrong. Count the ideas a caller must understand, not just commas.

**Before:** callers must remember which same-typed number means what.

```rust
fn contains(start: usize, end: usize, position: usize) -> bool {
    start <= position && position < end
}

assert!(contains(2, 5, 4));
```

**After:** an existing standard type expresses the half-open interval.

```rust
use std::ops::Range;

fn contains(window: Range<usize>, position: usize) -> bool {
    window.contains(&position)
}

assert!(contains(2..5, 4));
assert!(!contains(2..5, 5));
```

Use an existing domain struct or standard type before inventing an options
object. Add a builder only if construction has a real readability problem.
This signature change is suitable for an internal helper; changing a public
function requires the project's compatibility process.

## F2: Make outputs and mutation explicit

Returning a computed value often makes data flow easiest to follow. Rust also
makes exclusive mutation explicit in the signature and call site. Buffer
reuse, writers, and state-changing methods can be honest interfaces.

**Acceptable:** the name and `&mut` both communicate the intended update.

```rust
fn append_line(output: &mut String, line: &str) {
    output.push_str(line);
    output.push('\n');
}

let mut report = String::from("Header\n");
append_line(&mut report, "Ready");
assert_eq!(report, "Header\nReady\n");
```

Do not replace this with cloning the whole report merely to satisfy a literal
ban on output arguments. Conversely, a query named `line_count` should not
silently rewrite its input. Choose borrowed inputs or owned values to express
the operation's actual contract, not as a blanket preference.

## F3: Make the chosen operation clear

A call like `render_status(true, false)` hides the meaning of both switches.
Split unrelated operations. If one operation has a genuine domain mode, an
enum can make that choice explicit without duplicating the operation.

**After:** one rendering responsibility with a named presentation choice.

```rust
enum StatusFormat {
    Compact,
    Sentence,
}

fn render_status(format: StatusFormat) -> &'static str {
    match format {
        StatusFormat::Compact => "ready",
        StatusFormat::Sentence => "The service is ready.",
    }
}

assert_eq!(render_status(StatusFormat::Compact), "ready");
assert_eq!(
    render_status(StatusFormat::Sentence),
    "The service is ready."
);
```

A boolean representing actual domain data, such as `set_enabled(enabled)`,
does not necessarily combine responsibilities. Do not automatically replace
every boolean with an enum.

## F4: Verify that a function is dead before deleting it

Check callers, tests, feature-gated code, examples, and downstream public API
commitments. An exported library function can have users outside the
repository. Remove obsolete internal helpers; use the project's compatibility
process for public removals.

## Keep the success path readable

Use `Result` and `?` to express failure without burying the useful work. A
parser can return a typed error without also logging, reading files, retrying,
or terminating the process.

**After:** parse one domain value and explain its failure to the caller.

```rust
#[derive(Debug, PartialEq)]
enum SeatCountError {
    InvalidNumber,
    OutOfRange,
}

fn parse_seat_count(input: &str) -> Result<u16, SeatCountError> {
    const MAX_SEATS: u16 = 500;

    let seats: u16 = input.parse().map_err(|_| SeatCountError::InvalidNumber)?;
    if !(1..=MAX_SEATS).contains(&seats) {
        return Err(SeatCountError::OutOfRange);
    }
    Ok(seats)
}

assert_eq!(parse_seat_count("12"), Ok(12));
assert_eq!(parse_seat_count("many"), Err(SeatCountError::InvalidNumber));
assert_eq!(parse_seat_count("0"), Err(SeatCountError::OutOfRange));
assert_eq!(parse_seat_count("1"), Ok(1));
assert_eq!(parse_seat_count("500"), Ok(500));
assert_eq!(parse_seat_count("501"), Err(SeatCountError::OutOfRange));
assert_eq!(
    parse_seat_count("65536"),
    Err(SeatCountError::InvalidNumber)
);
```

Here `InvalidNumber` means the input cannot be parsed as a `u16`;
`OutOfRange` means it is representable but outside the allowed seat count.
At an I/O boundary, preserve
the underlying cause and add context the caller needs rather than discarding
it. An `Option` is appropriate when absence is normal, not as a way to hide
errors. Keep simple expressions inline; extract a helper when it names a
useful responsibility, not merely because extraction is possible.

## Review questions

- Can a reader understand the call without opening the implementation?
- Do the name, ownership, mutation, and return type tell the same story?
- Does extraction improve cohesion, or just scatter a straightforward flow?
- Does the change preserve public behavior and failure contracts?

## Sources

- Robert C. Martin, [*Clean Code*](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), Chapters 3 and 17 (F1–F4, G30, G34).
- [The Rust Book: mutable references](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#mutable-references).
- [The Rust Book: recoverable errors and propagation](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html).
