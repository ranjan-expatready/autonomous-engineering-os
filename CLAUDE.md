# CLAUDE.md — Project Constitution

This file is the primary context document for AI agents (Claude, Trae, etc.)
operating in this repository. Read this file on every session start.

## Identity

- **Repo**: autonomous-engineering-os
- **Purpose**: Reusable framework and operating system for autonomous software
  development, governed by AI agents with human-in-the-loop oversight.
- **Language**: Python (FastAPI skeleton in APP/)
- **Test framework**: pytest
- **Package manager**: pip (requirements.txt in APP/)
- **CI**: GitHub Actions (ci.yml, machine-board.yml, daily-brief.yml)

## Repository Structure

```
GOVERNANCE/          Guardrails, quality gates, risk tiers, cost policy, ADRs
GOVERNANCE/DECISIONS/  Architecture Decision Records (ADR-NNNN format)
AGENTS/              Agent roles, contracts, best practices
FRAMEWORK_KNOWLEDGE/ Autonomy principles, engineering standards
APP/                 Application code (FastAPI)
APP/tests/           Application unit tests (pytest)
tests/               Governance/framework tests (pytest)
scripts/             CI and automation scripts
COCKPIT/             Operational artifacts (daily briefs, approvals)
ARCHITECTURE/        Product architecture (to be filled)
PRODUCT/             Product requirements (to be filled)
BACKLOG/             Work items (to be filled)
```

## Governance Rules

1. **All changes go through PRs.** Direct pushes to `main` are blocked by
   branch protection. No exceptions.
2. **Required checks must pass.** Currently: `machine-board`. Phase 2 will
   add `Unit Tests` and `gitleaks`.
3. **ADRs for significant decisions.** Any architectural or governance change
   must be recorded in `GOVERNANCE/DECISIONS/ADR-NNNN-title.md`.
4. **Tests for new features.** Every new feature must include tests. See
   `GOVERNANCE/QUALITY_GATES.md` for the staged coverage policy.
5. **No secrets in code.** Gitleaks scans every PR. Never commit API keys,
   tokens, or credentials.

## Development Workflow

```
1. Create feature branch from main
2. Make changes
3. Push and open PR targeting main
4. machine-board governance check runs automatically
5. CI checks run (Unit Tests, gitleaks, etc.)
6. Merge when all required checks pass
```

## Running Tests

```bash
# Governance tests
python -m pytest tests/ -v

# APP tests
pip install -r APP/requirements.txt
python -m pytest APP/tests/ -v

# All tests
pip install -r APP/requirements.txt
python -m pytest tests/ APP/tests/ -v
```

## Key Files

| File | Purpose |
|------|---------|
| `GOVERNANCE/GUARDRAILS.md` | Engineering guardrails and constraints |
| `GOVERNANCE/QUALITY_GATES.md` | Staged quality gate policy |
| `GOVERNANCE/DEFINITION_OF_DONE.md` | What "done" means for work items |
| `GOVERNANCE/RISK_TIERS.md` | Risk classification for changes |
| `GOVERNANCE/DECISIONS/README.md` | ADR conventions and index |
| `AGENTS/ROLES.md` | Agent role definitions |
| `AGENTS/CONTRACTS.md` | Agent interaction contracts |
| `scripts/governance_validator.py` | machine-board governance check script |

## Conventions

- **Commit messages**: Use conventional commits (`feat:`, `fix:`, `chore:`,
  `docs:`). Keep the subject line under 72 characters.
- **Branch names**: Use prefixes (`feat/`, `fix/`, `chore/`, `docs/`).
- **PR descriptions**: Include a Summary section and Test Plan.
- **ADR format**: Follow `GOVERNANCE/DECISIONS/TEMPLATE.md`.

## What NOT to Do

- Do not push directly to `main` (branch protection will reject it)
- Do not commit secrets, tokens, or credentials
- Do not skip tests for new features
- Do not modify ADRs after they are accepted (create a superseding ADR)
- Do not bypass governance checks
