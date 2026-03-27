# Threat Model — PropChain Smart Contracts

> **Documentation Version**: 1.0.0 — Created March 2026

This document describes the threat actors, attack surfaces, and mitigations for the PropChain on-chain system. It is reviewed and updated with every major release.

**Related documents:**
- [SECURITY.md](../SECURITY.md) — policy, reporting, and best practices
- [Incident Response](incident-response.md) — response runbook
- [Security Pipeline](security_pipeline.md) — automated toolchain

---

## System Overview

PropChain is a Substrate/ink! smart contract system for tokenized real estate. The on-chain components are:

| Contract | Role |
|---|---|
| `property-token` | ERC-721/PSP34 property NFT |
| `property-management` | Lifecycle management for registered properties |
| `escrow` | Atomic sale / settlement engine |
| `fractional` | Fractional ownership shares |
| `compliance_registry` | KYC/AML status registry |
| `oracle` | Price and property data feeds |
| `governance` | DAO governance and voting |
| `bridge` | Cross-chain asset transfer |
| `staking` | PROP token staking |
| `ai-valuation` | On-chain AI valuation oracle |
| `fees` | Dynamic fee calculation |
| `insurance` | Property insurance pool |
| `zk-compliance` | Zero-knowledge compliance proofs |
| `proxy` | Upgradeable proxy (admin controlled) |

**Trust boundary**: Everything outside the WASM sandbox (RPC nodes, off-chain oracles, IPFS, frontend) is untrusted.

---

## Threat Actors

| Actor | Motivation | Capability |
|---|---|---|
| External attacker | Financial gain, disruption | Can submit arbitrary transactions, observe all on-chain state |
| Malicious token holder | Drain funds, manipulate governance | Holds PROP tokens; can vote and call any public message |
| Compromised oracle | Feed false property data | Controls one or more oracle accounts |
| Insider / compromised admin | Rug pull, unauthorized upgrade | Holds admin / owner keys |
| Dependency supply chain | Backdoor via malicious crate | Can introduce vulnerable Rust code via `Cargo.lock` updates |
| Miner / validator | MEV, front-running | Can reorder or delay transactions |

---

## Attack Surface

### 1. Public Contract Messages

Every `#[ink(message)]` is callable by any account. Risk: unauthenticated state mutation.

**Controls:**
- All state-mutating messages check caller authorization before any state change.
- Role-based checks use `contracts/lib/` helpers (`only_owner`, `only_role`).
- Compliance check (`compliance_registry.require_compliance`) on all high-value messages.

### 2. Cross-Contract Calls

Contract A calls Contract B, which can re-enter Contract A (reentrancy), or B can be replaced with a malicious contract.

**Controls:**
- Checks-Effects-Interactions (CEI) pattern: mutate storage before any cross-contract call.
- Hard-coded contract address allowlist for privileged callee contracts.
- Explicit gas limit on all `CallBuilder` invocations.

### 3. Oracle Data Integrity

The `oracle` and `ai-valuation` contracts ingest off-chain data. A malicious or compromised oracle feed can manipulate property valuations and escrow settlement prices.

**Controls:**
- Median aggregation across at least 3 independent oracles.
- Staleness check: data older than `MAX_ORACLE_AGE` blocks is rejected.
- Price deviation guard: reject updates that deviate more than `MAX_PRICE_DELTA`% from the last accepted value.
- Oracle operator keys are multisig controlled.

### 4. Governance Attacks

Flash-loan or token acquisition attacks can temporarily acquire enough voting power to pass malicious proposals.

**Controls:**
- Voting power snapshot at proposal creation block (no flash-loan attack vector).
- Quorum requirement: at least 10% of circulating supply must vote.
- Timelock: proposals must wait `GOVERNANCE_TIMELOCK` blocks between passing and execution.
- Veto power held by security multisig for the first 12 months.

### 5. Proxy / Upgrade Mechanism

The `proxy` contract enables code upgrades. A malicious or unauthorized upgrade can replace all contract logic.

**Controls:**
- Upgrade requires M-of-N multisig approval (currently 3-of-5).
- 72-hour timelock between upgrade proposal and execution.
- Upgrade events are emitted on-chain and monitored by automated alerts.
- Storage layout compatibility check in CI before any upgrade.

### 6. Bridge and Cross-Chain Transfers

The bridge contract locks assets on one chain and mints counterparts on another. A bug here can cause double-spend or permanent fund lock.

**Controls:**
- Nonce-based replay protection on all bridge messages.
- Cryptographic proof verification (Merkle proof from source chain).
- Daily transfer cap per source account.
- Emergency pause controlled by multisig.

### 7. Integer Arithmetic

Overflow or underflow in balance calculations can lead to unbounded minting or draining of funds.

**Controls:**
- All arithmetic on user-supplied values uses `checked_*` or `saturating_*`.
- CI clippy rule `integer_arithmetic` warns on unchecked ops.
- Fuzz tests (via `proptest`) exercise arithmetic edge cases.

### 8. Access Control Bypass

Incorrect implementation of ownership / role checks can allow unauthorized callers to execute privileged messages.

**Controls:**
- Shared `only_owner` / `only_role` macros from `contracts/lib/` used uniformly.
- Integration tests assert that every privileged message reverts when called by a non-authorized account.
- Clippy custom lint flags functions named `admin_*` or `owner_*` that lack the guard macro.

### 9. Denial of Service

An attacker can grief the contract by filling storage or forcing expensive iterations.

**Controls:**
- Per-account storage deposit requirement (ink! storage rent).
- Bounded `Mapping` reads; no unbounded `Vec` iteration in O(n) messages.
- Gas limit enforced on all cross-contract calls.

### 10. Dependency Supply Chain

A compromised crate in `Cargo.lock` can introduce backdoors.

**Controls:**
- `cargo deny` allowlist of permitted licenses and registries.
- `cargo audit` against RustSec advisory database in every CI run.
- Dependency PRs require a maintainer to review `Cargo.lock` diff.

---

## Risk Matrix

| Threat | Likelihood | Impact | Residual Risk |
|---|---|---|---|
| Reentrancy | Low (CEI enforced) | Critical | Low |
| Malicious oracle feed | Medium | High | Medium |
| Governance flash-loan attack | Low (snapshot voting) | High | Low |
| Unauthorized upgrade | Low (multisig + timelock) | Critical | Low |
| Integer overflow | Low (checked math) | High | Low |
| Access control bypass | Low (shared guards) | High | Low |
| Bridge double-spend | Low (nonce + proof) | Critical | Low |
| Supply-chain attack | Medium | High | Medium |
| DoS / storage griefing | Medium | Medium | Low |

---

## Out-of-Scope Threats

The following are explicitly not mitigated at the contract layer:

- **Substrate / Polkadot consensus attacks** — handled by the validator set.
- **Frontend XSS / CSRF** — handled by frontend security practices.
- **IPFS content availability** — IPFS is used only for metadata; contracts store CIDs, not content.
- **Private key compromise of individual non-admin users** — users are responsible for their own key management.

---

## Threat Model Review Cycle

This document is reviewed:

- Before every major release.
- After any security audit finding of High severity or above.
- After any incident.
- At minimum, every 6 months.

Reviewer signs off by merging a PR that bumps the documentation version header.
