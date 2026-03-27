# Incident Response Runbook

> **Documentation Version**: 1.0.0 — Created March 2026

This runbook defines the procedures for detecting, containing, assessing, resolving, and learning from security incidents affecting PropChain smart contracts.

**Related documents:**
- [SECURITY.md](../SECURITY.md) — reporting vulnerabilities and best practices
- [Threat Model](threat-model.md) — known threats and mitigations
- [Disaster Recovery](DISASTER_RECOVERY.md) — infrastructure recovery procedures

---

## Severity Levels

| Level | Definition | SLA (fix deployed) |
|---|---|---|
| **P0 — Critical** | Active exploit, funds at risk, total contract compromise | 4 hours |
| **P1 — High** | Exploitable vulnerability, no confirmed active exploit | 24 hours |
| **P2 — Medium** | Vulnerability requires specific preconditions to exploit | 7 days |
| **P3 — Low** | Hardening issue, defence-in-depth gap, no direct exploit path | 30 days |

---

## Contacts

| Role | Contact | Channel |
|---|---|---|
| Security team lead | security@propchain.io | Email + Discord `#security-private` |
| On-call engineer | Rotated weekly — see internal runbook | PagerDuty |
| Legal counsel | legal@propchain.io | Email only |
| Communications lead | comms@propchain.io | Email + Slack |
| External auditor (retained) | Engagement letter on file | Email |

---

## Phase 1 — Detection

### Sources

Incidents may be detected via:

- **Automated monitoring**: on-chain event alerts (unusual transfer volumes, pause events, upgrade events)
- **CI / security pipeline**: failed `cargo audit`, `trivy`, or `security-audit` runs
- **Responsible disclosure**: email to security@propchain.io
- **Community report**: Discord, GitHub, or social media

### Initial Triage (≤ 1 hour)

1. Open a **private** GitHub Security Advisory draft (do not use a public issue).
2. Assign a severity level using the table above.
3. Page the on-call engineer if severity is P0 or P1.
4. Create a private incident channel in Discord (`incident-YYYY-MM-DD`).
5. Record initial findings in the advisory draft:
   - Affected contracts and message(s)
   - Reproduction steps
   - Estimated impact (funds at risk, accounts affected)

---

## Phase 2 — Containment

### Pause Affected Contracts

All PropChain contracts implement a pause mechanism controlled by the security multisig:

```bash
# Pause a single contract (requires M-of-N multisig signatures)
./scripts/deploy.sh --action pause --contract <contract-name> --network <network>
```

Pause as soon as a P0 or P1 is confirmed. Do not wait for a full root-cause analysis.

### Isolate the Bridge

If the `bridge` contract is involved, trigger the emergency halt:

```bash
./scripts/deploy.sh --action bridge-halt --network <network>
```

This stops all inbound and outbound cross-chain transfers.

### Freeze Governance

If governance is compromised or a malicious proposal is near execution:

```bash
./scripts/deploy.sh --action governance-freeze --network <network>
```

The timelock guardian (multisig) can veto any pending proposal.

---

## Phase 3 — Assessment

Within **2 hours** of detection for P0/P1:

1. **Scope**: which contracts are affected? Which storage slots or balances were changed?
2. **Impact**: what is the maximum exploitable value? How many user accounts are affected?
3. **Vector**: what is the exact call sequence to reproduce the issue?
4. **Propagation**: can the exploit spread to other contracts via cross-contract calls?

Document all findings in the private GitHub Security Advisory.

---

## Phase 4 — Fix

1. Create a **private fork** or branch from the affected release tag. Do not push the fix branch to the public remote until the embargo lifts.
2. Develop the minimal fix:
   - Prefer the smallest possible change.
   - Add a regression test that reproduces the exact exploit vector.
3. Internal review: at least **two maintainers** must review the fix on the private branch.
4. If P0 or P1: request an expedited review from the retained external auditor.
5. Run the full test suite and security pipeline on the fix branch:
   ```bash
   cargo test
   cargo clippy -- -D warnings
   cargo audit
   ./target/release/security-audit audit --report report.json
   ```
6. Prepare upgrade payload using the proxy upgrade mechanism.

---

## Phase 5 — Disclosure

Before deploying the fix publicly:

1. Notify known integrators (SDK users, frontends, exchanges) via the private channel at least **24 hours before** the fix is deployed (for P2/P3) or **simultaneously** with deployment (for P0/P1 where delay risks further harm).
2. Prepare a public security advisory that includes:
   - CVE / advisory ID
   - Affected versions
   - Fixed versions
   - Severity and CVSS score
   - Description (without exploit code)
   - Credit to reporter (if consented)
3. Legal counsel reviews the advisory before publication.

---

## Phase 6 — Deployment

### Upgrade Procedure

1. Propose the upgrade via the proxy contract (requires multisig initiation):
   ```bash
   ./scripts/upgrade.sh --contract <name> --code <path-to-new-wasm> --network <network>
   ```
2. Collect M-of-N multisig signatures (minimum 3-of-5 for production).
3. After the timelock expires (72 hours in normal operation; can be bypassed as 24-hour emergency by security multisig for P0), execute the upgrade.
4. Verify the upgrade on-chain:
   ```bash
   ./scripts/health-check.sh --network <network>
   ```
5. Unpause affected contracts:
   ```bash
   ./scripts/deploy.sh --action unpause --contract <contract-name> --network <network>
   ```

### Rollback

If the upgrade introduces a regression:

1. Re-propose the previous known-good code hash via the proxy.
2. Apply the same multisig + timelock flow.
3. There is no instant rollback — plan upgrades carefully.

---

## Phase 7 — Post-Mortem

A post-mortem document must be published **within 30 days** of incident resolution for any P0 or P1.

### Post-Mortem Template

```markdown
## Incident Post-Mortem — <title>

**Date**: YYYY-MM-DD
**Severity**: P0 / P1 / P2 / P3
**Duration**: X hours from detection to fix deployment
**Affected contracts**: ...
**Reporter credit**: ...

### Timeline
| Time (UTC) | Event |
|---|---|
| HH:MM | Detected via ... |
| HH:MM | On-call paged |
| ... | ... |

### Root Cause
...

### Impact
- Funds at risk: $X (actual loss: $Y)
- Affected accounts: N
- Downtime: X minutes (contracts paused)

### Fix Summary
...

### What Went Well
...

### What Went Wrong
...

### Action Items
| Item | Owner | Due |
|---|---|---|
| ... | @handle | YYYY-MM-DD |
```

Post-mortems are published to `docs/adr/` with status `Incident Post-Mortem`.

---

## Runbook Review

This runbook is reviewed:
- After every P0 or P1 incident
- Quarterly by the security team
- Whenever contacts or escalation paths change
