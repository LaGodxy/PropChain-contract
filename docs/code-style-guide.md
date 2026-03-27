# Code Style Guide

> **Documentation Version**: 1.0.0 — Created March 2026

This document is the canonical code style reference for PropChain Smart Contracts. Style is automatically enforced by `rustfmt`, `clippy`, and the pre-commit hooks. Read this guide to understand *why* the rules exist, not just *what* they are.

**Related documents:**
- [CONTRIBUTING.md](../CONTRIBUTING.md) — contribution workflow
- [docs/commenting-standards.md](commenting-standards.md) — rustdoc conventions
- [`.pre-commit-config.yaml`](../.pre-commit-config.yaml) — automated enforcement

---

## Automated Enforcement

All style checks run automatically. You should never need to manually fix style issues that tooling can fix:

```bash
# Fix formatting automatically
cargo fmt

# Check formatting without fixing (CI mode)
cargo fmt --check

# Run lints (zero-warnings policy)
cargo clippy -- -D warnings

# Run all pre-commit checks on staged files
pre-commit run

# Run all pre-commit checks on every file
pre-commit run --all-files
```

If a `#[allow(clippy::...)]` attribute is required, it **must** have a comment justifying the exception:

```rust
// STYLE-EXCEPTION: this function legitimately takes 8 args because it mirrors
// the on-chain storage struct fields 1:1; splitting it would hurt readability.
#[allow(clippy::too_many_arguments)]
pub fn new(...) -> Self { ... }
```

---

## Formatting (`rustfmt.toml`)

The project's `rustfmt.toml` enforces:

| Setting | Value | Reason |
|---|---|---|
| `max_width` | 100 | Fits on most displays without horizontal scroll |
| `tab_spaces` | 4 | Rust community standard |
| `newline_style` | Unix (`\n`) | Consistent across OS |
| `reorder_imports` | true | Deterministic import order |
| `edition` | 2021 | Current edition |
| `merge_derives` | true | Reduce noise |

**Never** add `// rustfmt::skip` without a comment explaining why.

---

## Naming Conventions

### Types and Structs

```rust
// PascalCase for all type-level items
pub struct PropertyRegistry { ... }
pub enum Error { NotFound, Unauthorized }
pub type PropertyId = u64;
pub trait Mintable { ... }
```

### Functions and Methods

```rust
// snake_case for all value-level items
pub fn register_property(...) -> Result<(), Error> { ... }
let property_count = self.count();
```

### Constants and Statics

```rust
// SCREAMING_SNAKE_CASE — both const and static
pub const MAX_PROPERTIES: u32 = 10_000;
pub const GOVERNANCE_TIMELOCK: u64 = 72 * 3_600;
```

### ink! Specific

```rust
// Contract struct name mirrors the contract's published name
// (matches the name in Cargo.toml [package].name in PascalCase)
#[ink::contract]
pub mod property_token {
    pub struct PropertyToken { ... }  // PascalCase
}

// Storage field names: snake_case
#[ink(storage)]
pub struct PropertyToken {
    owner: AccountId,
    total_supply: Balance,
    token_approvals: Mapping<TokenId, AccountId>,
}
```

### Modules

```rust
// snake_case for module names — mirrors directory structure
mod compliance_checks { ... }
mod storage_utils { ... }
```

---

## File Organisation

A contract source file should be organised in this order:

1. Module-level `#![...]` attributes
2. `use` imports (sorted by `rustfmt`)
3. `pub type` aliases
4. `pub const` / `pub static`
5. `#[ink::contract]` module
   1. `#[ink(storage)]` struct
   2. `impl` block — constructor(s) first, then `#[ink(message)]` methods, then private helpers
   3. `mod tests`
6. Non-contract `impl` blocks and helpers

---

## Error Handling

```rust
// Define a dedicated Error enum per contract — never use String errors
#[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
#[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
pub enum Error {
    /// The caller is not authorised to perform this action.
    Unauthorized,
    /// The property does not exist.
    PropertyNotFound,
    /// Arithmetic overflow.
    Overflow,
}

// Return Result<T, Error> from all fallible messages
#[ink(message)]
pub fn transfer(&mut self, to: AccountId, value: Balance) -> Result<(), Error> {
    let balance = self.balances.get(self.env().caller()).unwrap_or(0);
    let new_balance = balance.checked_sub(value).ok_or(Error::Overflow)?;
    self.balances.insert(self.env().caller(), &new_balance);
    Ok(())
}
```

- **Never** `panic!` in contract code; panics abort and may leave state inconsistent.
- **Never** use `unwrap()` in contract code outside `#[cfg(test)]`.
- **Never** use `expect()` in contract code outside `#[cfg(test)]`.

---

## Integer Arithmetic

```rust
// Always use checked or saturating arithmetic on user inputs
// BAD
let new_supply = self.total_supply + amount;

// GOOD
let new_supply = self.total_supply
    .checked_add(amount)
    .ok_or(Error::Overflow)?;
```

`saturating_*` is acceptable when overflow indicates a business rule violation rather than a bug (e.g., capping a counter at max value).

---

## Storage

```rust
// Use Mapping for collections that grow per-user
// BAD (O(n) unbounded iteration)
owners: Vec<AccountId>,

// GOOD (O(1) access)
owners: Mapping<AccountId, bool>,
```

- Do not store user-controlled strings longer than 256 bytes without explicit length checks.
- Never store sensitive data (PII, private keys) on-chain. Store hashes or IPFS CIDs only.

---

## Comments and Documentation

Full rustdoc conventions are in [docs/commenting-standards.md](commenting-standards.md). Short summary:

```rust
/// Short one-line summary (imperative mood, no trailing period for single line).
///
/// Longer description if needed. Explains *why*, not *what* — the code
/// already shows what it does.
///
/// # Arguments
///
/// * `value` - The amount to transfer, in smallest denomination.
///
/// # Errors
///
/// Returns [`Error::Unauthorized`] if the caller is not the token owner.
/// Returns [`Error::Overflow`] if the balance would overflow.
///
/// # Example
///
/// ```rust
/// assert!(contract.transfer(alice, 100).is_ok());
/// ```
#[ink(message)]
pub fn transfer(&mut self, to: AccountId, value: Balance) -> Result<(), Error> {
```

### Inline Comments

```rust
// Use inline comments sparingly — only when the *why* is non-obvious
let snapshot_block = self.env().block_number()
    .saturating_sub(1); // snapshot at N-1 to prevent flash-loan attacks
```

Do **not** comment self-evident code:

```rust
// BAD: comment just repeats the code
// increment the counter
self.count += 1;
```

---

## Testing Style

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use ink::env::test;

    // Test name: <unit>_<scenario>_<expected_outcome>
    #[test]
    fn transfer_zero_balance_returns_overflow_error() {
        // Arrange
        let mut contract = PropertyToken::new(AccountId::from([0x1; 32]));

        // Act
        let result = contract.transfer(AccountId::from([0x2; 32]), u128::MAX);

        // Assert
        assert_eq!(result, Err(Error::Overflow));
    }
}
```

- Use Arrange / Act / Assert structure with blank-line separators.
- Test name describes scenario + expected outcome.
- Each test should test exactly one behaviour.
- Do not share mutable state between tests.

---

## Pre-commit Hooks

The `.pre-commit-config.yaml` enforces the following on every commit:

| Hook | What it enforces |
|---|---|
| `rust-fmt` | `cargo fmt --check` |
| `rust-clippy` | `cargo clippy -- -D warnings` |
| `cargo-check` | Type-checks without building |
| `trailing-whitespace` | No trailing spaces |
| `end-of-file-fixer` | Files end with a newline |
| `detect-secrets` | No accidental secret leakage |
| `mdformat` | Consistent Markdown formatting |
| `shfmt` | Shell script formatting |
| `hadolint` | Dockerfile linting |
| `cargo-contract-build` | Contracts compile to WASM |
| `cargo-contract-test` | Contract unit tests pass |

To install hooks:

```bash
./scripts/setup-pre-commit.sh
```

---

## Code Style Monitoring

The clippy configuration in `clippy.toml` enforces these thresholds:

| Metric | Threshold |
|---|---|
| Cognitive complexity per function | 30 |
| Function arguments | 7 |
| Lines per function | 100 |
| Type complexity | 250 |

If CI clippy fails, **fix the code** — do not raise the threshold without a team discussion and a comment in `clippy.toml` explaining the new value.

Review `clippy.toml` thresholds quarterly and lower them as the codebase matures.
