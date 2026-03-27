# Automated Security Audit Pipeline

> **Documentation Version**: 2.0.0 — Updated March 2026

This project includes a comprehensive automated security pipeline that runs on every commit and pull request. It is the first line of defence between a code change and a production deployment.

**Related documents:**
- [SECURITY.md](../SECURITY.md) — policy, best practices, and vulnerability reporting
- [Threat Model](threat-model.md) — what the pipeline is defending against
- [Incident Response](incident-response.md) — what to do when the pipeline catches something

## Components

### 1. Static Analysis
- **Clippy**: Rust's standard linter with strict security settings (`-D warnings`).
- **Custom Security Tool** (`security-audit`): A custom Rust binary that scans for:
  - `unsafe` blocks
  - `TODO`/`FIXME` comments
  - Cyclomatic complexity violations
  - Unbounded `Vec` usage in contract code
- **Trivy**: Scans filesystem and dependencies for known CVEs.

### 2. Dependency Scanning
- **cargo-audit**: Checks `Cargo.lock` against the RustSec Advisory Database.
- **cargo-deny**: Enforces license allowlist, bans duplicate dependency versions, and restricts unknown crate registries (see `deny.toml`).

### 3. Formal Verification
- **Kani Rust Verifier**: Used to formally verify critical safety properties of smart contracts.
- **Proof Harnesses**: Located in `contracts/lib/src/lib.rs` under `mod verification`.
- Properties verified include: no arithmetic overflow in balance operations, no unreachable panic paths in critical messages.

### 4. Secret Detection
- **detect-secrets**: Pre-commit hook that scans every commit for accidental secret leakage (API keys, mnemonics, private keys). Configured via `.secrets.baseline`.

### 5. Fuzzing
- **proptest**: Property-based tests exercise arithmetic and input-validation code with randomised inputs to catch edge cases not covered by unit tests.

## Running Locally

```bash
# Build and run the custom audit tool
cargo build --release --bin security-audit
./target/release/security-audit audit --report report.json

# Dependency audit
cargo audit
cargo deny check

# Static analysis
cargo clippy -- -D warnings

# Formal verification
cargo install --locked kani-verifier
cargo kani setup
cargo kani

# Full pre-commit suite (includes secret detection, fmt, clippy)
pre-commit run --all-files
```

## CI/CD Integration

The pipeline is defined in `.github/workflows/security.yml` and runs on every push and pull request to `main` and `develop`.

Steps in order:

1. **Format check** — `cargo fmt --check`
2. **Clippy** — `cargo clippy -- -D warnings`
3. **Tests** — `cargo test`
4. **Audit tool** — custom security scanner, generates `report.json`
5. **cargo-audit** — RustSec CVE check
6. **cargo-deny** — license and dependency policy
7. **Trivy** — filesystem CVE scan
8. **Kani** — formal verification proofs
9. **Secret scan** — `detect-secrets audit`

All steps must pass before a PR can be merged.

## Security Score

The `security-audit` tool calculates a score (0–100):

| Deduction | Condition |
|---|---|
| –10 points | Each clippy error |
| –2 points | Each clippy warning |
| –5 points | Each `unsafe` block in contract code |
| –20 points | Each known CVE in dependencies |
| –3 points | Each `TODO`/`FIXME` in contract code |

**Minimum acceptable score**: 90 on any release branch.

The score is written to `report.json` at the root and archived as a CI artifact.

## Adding New Security Checks

1. Add the tool to `security-audit/src/` or as a new CI step in `.github/workflows/security.yml`.
2. Update the score deduction table above.
3. Update `deny.toml` if a new dependency is introduced.
4. Document the tool in the Components section above.

