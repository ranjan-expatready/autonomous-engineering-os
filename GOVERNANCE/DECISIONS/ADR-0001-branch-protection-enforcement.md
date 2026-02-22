# ADR-0001: Branch Protection Enforcement Model

- **Status**: accepted
- **Date**: 2026-02-22
- **Deciders**: Architect (governance CTO), Founder (Ranjan), Claude (executor)

## Context

The `autonomous-engineering-os` repository had no branch protection on `main`.
Direct pushes bypassed all CI checks. The CI workflow itself was broken
(services: null causing 0-second failures on every run). Governance was
advisory-only with no enforcement mechanism.

INCIDENT-0001 (wrong PROJECT mount) demonstrated that without enforcement
gates, invalid state can propagate undetected through the governance loop.

## Decision

Adopt a phased branch protection model:

**Phase 1 (implemented 2026-02-22):**
- Required status check: `machine-board` (governance validator)
- Require PR before merge
- Require conversation resolution
- Enforce for admins
- Block force pushes and branch deletions

**Phase 2 (after CI stabilizes across 2-3 PRs):**
- Add required check: `Unit Tests`
- Add required check: `gitleaks`

**Phase 3 (future):**
- Add required check: `Security Checks` (when Trivy is configured)

The repo was made public (from private) to enable branch protection on the
GitHub Free plan.

## Consequences

### Positive

- Every change to `main` must pass governance validation
- CI checks are enforced, not advisory
- Direct push path is eliminated — no bypassing checks
- Phased approach avoids deadlocking merges with unproven checks

### Negative

- Repo is now public (was private) — code and governance artifacts are visible
- Single required check (`machine-board`) is a single point of failure
- Admins cannot bypass in emergencies (enforce_admins = true)

### Neutral

- PR-based workflow adds friction to small changes (acceptable trade-off)
- Phase 2 checks need manual addition by Founder after verification

## References

- COMMS-0007: Patch set proposal (CI fixes + daily-brief)
- COMMS-0008: Verified required check names
- COMMS-0009: Branch protection blocker escalation
- INCIDENT-0001: Wrong PROJECT mount
- PR #36: CI workflow repair
- PR #37: Daily-brief no-PR mode
