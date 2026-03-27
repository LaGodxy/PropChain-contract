# Documentation Maintenance Process

> **Documentation Version**: 1.0.0 — Created March 2026

This document defines the process for keeping PropChain documentation accurate, versioned, and up to date.

## Versioning Policy

Documentation versions follow **MAJOR.MINOR.PATCH** semantics, independent of but synchronized with contract releases:

| Change type | Version bump | Example |
|---|---|---|
| Complete rewrite of a doc | MAJOR | `1.x.x → 2.0.0` |
| New section added or significant expansion | MINOR | `1.0.x → 1.1.0` |
| Typo fix, link update, minor correction | PATCH | `1.0.0 → 1.0.1` |

Every documentation file includes a version banner at the top:

```markdown
> **Documentation Version**: X.Y.Z — Updated Month YYYY
```

Bump this in the same PR as the content change.

## Ownership

| Document | Owner |
|---|---|
| `docs/architecture.md` | Protocol team |
| `docs/contracts.md` | Contract authors (per contract) |
| `docs/security_pipeline.md`, `SECURITY.md`, `docs/threat-model.md`, `docs/incident-response.md` | Security team |
| `docs/deployment.md` | DevOps team |
| `docs/code-style-guide.md` | All contributors / maintainers |
| `docs/tutorials/` | Developer relations |
| `CONTRIBUTING.md`, `DEVELOPMENT.md` | All maintainers |
| `docs/adr/` | Author of the decision |

If you change code in a module, you own the documentation update for that module.

## When to Update Documentation

### Required (blocks PR merge)
- Any change to a public `#[ink(message)]` signature
- Any change to storage layout
- New contract deployed or removed
- New CLI script or changed script behaviour
- New environment variable or configuration key
- Security policy or incident response changes

### Recommended
- Refactoring that changes internal design significantly
- Performance improvements worth highlighting
- Bug fixes that users or integrators should know about

## Documentation Review Checklist

Reviewers should verify:

- [ ] Version header bumped if content changed
- [ ] All new `#[ink(message)]` functions have rustdoc (`///`)
- [ ] `docs/contracts.md` API table up to date
- [ ] Links in updated doc still resolve (no broken relative paths)
- [ ] Code examples in docs compile (if testable)
- [ ] Architecture diagrams updated if topology changed
- [ ] `CHANGELOG.md` entry added for user-visible changes

## Directory Structure

```
docs/
├── adr/                        # Architecture Decision Records (never delete)
│   └── NNNN-description.md
├── tutorials/                  # Beginner-friendly task guides
├── architecture.md             # System overview and design
├── best-practices.md           # Integration best practices
├── BRIDGE_GUIDE.md             # Cross-chain bridge usage
├── code-style-guide.md         # Code style reference
├── commenting-standards.md     # Rustdoc standards
├── compliance-*.md             # Compliance and regulatory docs
├── contracts.md                # API reference for all contracts
├── deployment.md               # Deployment procedures
├── DISASTER_RECOVERY.md        # Disaster recovery runbook
├── documentation-maintenance.md # This file
├── dynamic-fees-and-market.md  # Fee model documentation
├── error-handling.md           # Error catalogue
├── health-checks.md            # Health monitoring
├── incident-response.md        # Security incident runbook
├── integration.md              # SDK / frontend integration guide
├── logging.md                  # Logging conventions
├── onboarding-checklist.md     # New contributor checklist
├── security_pipeline.md        # CI security toolchain
├── storage-patterns.md         # On-chain storage patterns
├── testing-guide.md            # Testing strategy
├── threat-model.md             # Threat model and mitigations
└── troubleshooting-faq.md      # Common problems
```

## Architecture Decision Records (ADRs)

For every significant design decision:

1. Copy `docs/adr/0001-record-architecture-decisions.md` as a template.
2. Number sequentially: `docs/adr/NNNN-short-title.md`.
3. Set status to `Proposed` → `Accepted` / `Rejected` / `Superseded`.
4. Link to the ADR from the relevant `docs/` or contract source file.

ADRs are **immutable once accepted** — supersede them with a new ADR rather than editing.

## Regular Maintenance Schedule

| Cadence | Activity |
|---|---|
| Every PR | Update docs for changed behaviour (required) |
| Weekly | Scan for broken links (`scripts/check-links.sh` if available) |
| Monthly | Review tutorial accuracy against latest contract ABIs |
| Per release | Full doc audit — bump MINOR version on any outdated files |
| Annually | Full rewrite review — deprecate or archive stale docs |

## Creating New Documentation

1. Determine the right file: is it a tutorial, a reference, or a runbook?
2. Use an existing file as a template for structure.
3. Add the version header.
4. Link to the new file from the relevant parent doc (`docs/contracts.md`, `CONTRIBUTING.md`, `DEVELOPMENT.md`, etc.).
5. Add an entry to the directory structure table above.

## Deprecating Documentation

1. Add a deprecation notice at the top of the file:
   ```markdown
   > **DEPRECATED**: This document is superseded by [new-doc.md](new-doc.md) as of vX.Y.
   ```
2. Keep the file for at least one release cycle.
3. Delete in a follow-up PR after the release.
