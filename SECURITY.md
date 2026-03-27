# Security Policy

> **Documentation Version**: 2.0.0 — Updated March 2026

This document is the top-level security reference for PropChain Smart Contracts. It covers the security pipeline, best practices, audit procedures, and incident response.

**Related documents:**
- [Threat Model](docs/threat-model.md) — threat actors, attack vectors, and mitigations
- [Incident Response](docs/incident-response.md) — step-by-step runbook for security incidents
- [Security Pipeline](docs/security_pipeline.md) — automated CI/CD security toolchain

---

## Table of Contents

1. [Reporting a Vulnerability](#reporting-a-vulnerability)
2. [Security Best Practices](#security-best-practices)
3. [Security Pipeline & Automated Checks](#security-pipeline--automated-checks)
4. [Security Audit Documentation](#security-audit-documentation)
5. [Threat Model Summary](#threat-model-summary)
6. [Incident Response Summary](#incident-response-summary)

---

## Reporting a Vulnerability

**DO NOT** open a public GitHub issue. Use responsible disclosure:

1. Email **security@propchain.io** with subject `[SECURITY] <brief description>`.
2. Include:
   - Description of the vulnerability
   - Reproduction steps (contract method, inputs, expected vs actual behaviour)
   - Potential impact assessment
   - Any suggested mitigations
3. You will receive an acknowledgement within **48 hours**.
4. Our team will triage, fix, and coordinate disclosure with you.
5. We operate a bug bounty — severity-based rewards are paid after fix confirmation.

**Embargo period**: We request a minimum 90-day embargo from disclosure to public announcement to allow dependent integrators to patch.

---

## Security Best Practices

### Reentrancy and Cross-Contract Calls

- Complete all state mutations **before** making cross-contract calls (checks-effects-interactions pattern).
- Never re-enter a contract from a callback unless you have verified the caller.
- Use ink!'s `CallBuilder` with explicit gas limits for cross-contract calls.

### Integer Arithmetic

- Use `checked_add`, `checked_sub`, `checked_mul`, or `saturating_*` for all arithmetic on user-supplied values.
- Never rely solely on `overflow-checks = true` in `Cargo.toml`; that flag is off in release WASM by default.

```rust
// BAD
let total = balance + amount;

// GOOD
let total = balance.checked_add(amount).ok_or(Error::Overflow)?;
```

### Access Control

- Every state-mutating `#[ink(message)]` must enforce caller authorization.
- Use the `only_owner` / `only_role` patterns from `contracts/lib/`.
- Never use `self.env().caller()` as the sole proof of identity for privileged actions without an allowlist or role check.

### `unsafe` Blocks

- `unsafe` is **forbidden** in contract code.
- In non-contract Rust code (tooling, scripts), `unsafe` requires a `// SAFETY:` comment explaining why it is sound.
- CI clippy rules deny `unsafe_code` in the `contracts/` workspace.

### Input Validation

- Validate and bound all user-supplied lengths (strings, arrays, vecs) at the message boundary.
- Return an explicit `Error` variant rather than panicking. Panics in ink! contracts revert the transaction but can hide bugs.

### Storage

- Avoid unbounded storage growth. Use `Mapping` rather than `Vec` for sets that grow proportional to user count.
- Do not store plaintext PII on-chain. Store only hashes or IPFS CIDs.

### Dependency Management

- `cargo deny` and `cargo audit` run in CI and block merges on known CVEs.
- Add new dependencies only if they pass `cargo deny check` and have a credible maintainer.

### Secret Management

- Never commit private keys, mnemonics, or API secrets to the repository.
- CI secrets are stored in GitHub Actions encrypted secrets only.
- The `detect-secrets` pre-commit hook scans every commit for accidental secret leakage.

---

## Security Pipeline & Automated Checks

All contributions pass the following automated security pipeline. See [docs/security_pipeline.md](docs/security_pipeline.md) for configuration details.

| Tool | What it checks | Failure action |
|---|---|---|
| `cargo clippy -- -D warnings` | Lint errors, unsafe code, complexity | Blocks merge |
| `cargo audit` | RustSec CVEs in `Cargo.lock` | Blocks merge |
| `cargo deny check` | License compliance, banned crates, duplicate deps | Blocks merge |
| Custom `security-audit` binary | Unsafe blocks, TODO/FIXME density, cyclomatic complexity | Blocks merge |
| `trivy fs` | OS-level and dependency CVEs | Blocks merge |
| `cargo kani` | Formal verification of critical proof harnesses | Blocks merge |
| `detect-secrets` (pre-commit) | Secret patterns in committed files | Blocks commit |

### Running Locally

```bash
# Full security audit
cargo build --release --bin security-audit
./target/release/security-audit audit --report report.json

# Dependency audit
cargo audit
cargo deny check

# Formal verification
cargo kani

# Pre-commit scan
pre-commit run --all-files
```

---

## Security Audit Documentation

### Scope

The security audit covers all contracts in `contracts/`, the shared library in `contracts/lib/`, and the `security-audit` tooling in `security-audit/`.

Out-of-scope: off-chain SDK code, scripts, frontend.

### Audit History

| Date | Auditor | Scope | Report |
|---|---|---|---|
| Internal | PropChain security team | All contracts v1.x | `report.json` (root) |

> External audits will be listed here when commissioned. See [docs/security_pipeline.md](docs/security_pipeline.md) for the automated audit approach used between external audits.

### Security Score

The `security-audit` tool produces a score (0–100):

| Deduction | Condition |
|---|---|
| –10 points | Each clippy error |
| –2 points | Each clippy warning |
| –5 points | Each `unsafe` block |
| –20 points | Each known CVE in dependencies |

Target score: **≥ 90** for any release branch.

### Requesting a Security Review

For changes to core authentication, cross-contract calls, or cryptographic operations, request a dedicated security review in your PR by adding the label `security-review` and pinging `@security-team`.

---

## Threat Model Summary

See [docs/threat-model.md](docs/threat-model.md) for the full threat model. Key high-severity threats:

| Threat | Mitigation |
|---|---|
| Reentrancy attack on Escrow | CEI pattern enforced; cross-contract call after state update |
| Integer overflow in Token | `checked_*` arithmetic on all user inputs |
| Unauthorized minting | `only_owner` guard + compliance check on all mint messages |
| Malicious oracle data | Median aggregation across N oracles; staleness timeout |
| Compromised admin key | Multisig required for all admin operations |
| Dependency supply-chain attack | `cargo deny` allowlist; `cargo audit` in CI |

---

## Incident Response Summary

See [docs/incident-response.md](docs/incident-response.md) for the full runbook. High-level steps:

1. **Detect** — automated alert or responsible disclosure email.
2. **Contain** — pause affected contracts via admin pause mechanism.
3. **Assess** — determine scope, severity (Critical / High / Medium / Low), and affected users.
4. **Fix** — develop and audit patch on a private branch.
5. **Disclose** — notify affected parties before publishing the fix.
6. **Deploy** — upgrade or redeploy contracts via the proxy upgrade mechanism.
7. **Post-mortem** — write a public post-mortem within 30 days.

**Emergency contacts**: security@propchain.io | Discord `#security-private` (invite-only)

