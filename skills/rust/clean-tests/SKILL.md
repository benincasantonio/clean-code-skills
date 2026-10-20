---
name: clean-tests
description: Apply Clean Code to Rust tests when behavior is unclear, boundaries are missing, or failures are slow and unreliable. Emphasize readable behavioral assertions, independence, repeatability, and focused regression coverage.
---

# Clean Tests in Rust

Tests should make a behavior understandable and make changing it safe.
Follow the book's F.I.R.S.T. principles: fast feedback, independent tests,
repeatable execution, self-validating results, and timely tests alongside the
change. One behavior per test can require several related assertions.

## T1: Cover behavior that could break

Test successful behavior, expected errors, and relevant state changes.
Assertions should describe a contract, not mirror each implementation step.
Do not write runtime tests solely to prove compiler-enforced type facts.

**After:** each test describes one reservation outcome, including its effect
on stock. The error-path assertion catches a partial mutation.

```rust
#[derive(Debug, PartialEq)]
struct InsufficientStock;

fn reserve(stock: &mut u16, requested: u16) -> Result<(), InsufficientStock> {
    let remaining = stock.checked_sub(requested).ok_or(InsufficientStock)?;
    *stock = remaining;
    Ok(())
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn reserving_available_stock_deducts_the_requested_amount() {
        let mut stock = 5;

        reserve(&mut stock, 2).expect("stock should cover the request");

        assert_eq!(stock, 3);
    }

    #[test]
    fn insufficient_stock_leaves_the_quantity_unchanged() {
        let mut stock = 2;

        let result = reserve(&mut stock, 3);

        assert_eq!(result, Err(InsufficientStock));
        assert_eq!(stock, 2);
    }

    #[test]
    fn reserving_all_available_stock_is_allowed() {
        let mut stock = u16::MAX;

        reserve(&mut stock, u16::MAX).expect("an exact request should fit");

        assert_eq!(stock, 0);
    }

    #[test]
    fn reserving_zero_from_empty_stock_is_a_no_op() {
        let mut stock = 0;

        assert_eq!(reserve(&mut stock, 0), Ok(()));
        assert_eq!(stock, 0);
    }
}
```

## T2: Use coverage as evidence

Use the project's coverage tooling, if available, to find branches without
meaningful tests. Read the assertions as well as the report. A line can run
without its behavior being verified. Do not add a new tool or chase a
percentage just to perform a small cleanup.

## T3: Keep useful simple tests

A test of an important default or a simple boundary can be inexpensive and
valuable. Keep it when it protects a contract. Do not add tests of trivial
implementation details merely to increase the count.

## T4: Resolve ignored tests

Do not hide a regression with `#[ignore]`. Fix the defect or clarify the
contract. A test genuinely requiring external setup can use an explanatory
reason such as `#[ignore = "requires the local test database"]`; document the
setup and the command that runs it.

`cargo test -- --ignored` runs ignored tests, but it does not supply missing
services. Remove a test only when its requirement is obsolete.

## T5: Exercise meaningful boundaries

Choose boundaries from the contract: empty input, exact limits, one beyond a
limit, absent values, malformed input, and failure without partial mutation.
For text, distinguish byte boundaries from character boundaries where the
operation requires it.

The reservation example defines zero as valid and accepts a request exactly
equal to available stock. Another domain may choose different rules; make
those choices visible in test names and assertions.

## T6: Test near a discovered bug

Add a regression test that fails before the fix, then cover neighboring cases
that share its cause. An off-by-one fix calls for both sides of the boundary,
not unrelated tests throughout the codebase.

## T7: Find patterns in failures

Several failing tests may share a state leak, clock assumption, or dependency
problem. Investigate that common cause instead of adding sleeps or increasing
timeouts until tests happen to pass.

## T8: Let gaps inform the design

Untested branches can reveal missing cases. Difficulty arranging a test can
reveal hidden I/O or mixed responsibilities. Separate domain computation from
effects when it improves the production design; do not introduce a trait for
every function just to mock it.

## T9: Keep feedback practical and deterministic

Prefer in-memory inputs for unit tests. Use real dependencies in integration
tests when their behavior is what needs verification. Document slower suites
separately; avoid a universal 100 ms limit.

Rust's test runner can execute tests concurrently. Give tests independent
state and resources rather than depending on ordering, a shared filename, or
global environment mutation. Avoid elapsed-time assertions for business
behavior when an explicit time input can express the same contract.

## Organization and execution

- Put unit tests near the implementation in `#[cfg(test)] mod tests`.
- Use `tests/` for integration tests of the public interface.
- Keep rustdoc examples executable when they explain the API.
- Use `cargo test` for the usual suite and `cargo test --doc` for documentation
  examples. A test block shown in a Markdown guide must also be run as actual
  Rust tests; compiling the guide alone does not execute its `#[test]` items.
- Follow the project's feature matrix, edition, and minimum supported Rust
  version. `--all-features` is not appropriate when features are incompatible.

Use `expect` in tests when an unexpected error should fail the test with a
helpful message. Use error assertions for expected failure; reserve
`#[should_panic]` for a documented panic contract.

## Sources

- Robert C. Martin, [*Clean Code*](https://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), Chapters 9 and 17 (T1–T9).
- [The Rust Book: test organization](https://doc.rust-lang.org/book/ch11-03-test-organization.html).
- [The Cargo Book: cargo test](https://doc.rust-lang.org/cargo/commands/cargo-test.html).
- [The rustdoc book: documentation tests](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html).
