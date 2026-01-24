# Dev Fast Mode & Staged Quality Gates Artifact

## Summary

This artifact documents the implementation of Dev Fast Mode and Staged Quality Gates, optimizing development speed while preserving safety for core governance and infrastructure. The repository now enables directory-based auto-merge for low-risk changes while maintaining strict human approval for governance, agents, and CI/CD workflows.

---

## Files Modified/Created

### Modified (3 files)
1. **GOVERNANCE/GUARDRAILS.md** (+144 lines) - Added Dev Stage Fast Mode section
2. **GOVERNANCE/RISK_TIERS.md** (+195 lines) - Added approval requirements by tier
3. **.github/workflows/ci.yml** (+22 lines) - Added coverage reporting placeholder

### Created (1 file)
4. **GOVERNANCE/QUALITY_GATES.md** (NEW, 381 lines) - Staged coverage policy with PR checklist

### Total Changes
- **Files Modified**: 3
- **Files Created**: 1
- **Total Lines Added**: ~742
- **Total Lines Removed**: ~3
- **Net Change**: ~739 lines

---

## Changes Summary

### 1. Dev Stage Fast Mode (GOVERNANCE/GUARDRAILS.md)

**New Section**: "Dev Stage Fast Mode" (226 lines total)

**Core Principle**: Speed with Safety - low-risk changes merge with CI, governance changes require human approval.

**Directory-Based Auto-Merge Rules**:

**Auto-Merge with CI Only** (No Human Review Required):
- `APP/**` - Application code (features, bug fixes, refactors)
- `PRODUCT/**` - Product specifications, requirements, user stories
- `BACKLOG/**` - Backlog items, task definitions, tickets
- `FRAMEWORK_KNOWLEDGE/**` - Technical knowledge and best practices
- `ARCHITECTURE/**` - Architecture documentation
- `RUNBOOKS/**` - Operational procedures and guides

**Requirements for Auto-Merge**:
- All CI checks must pass (lint, test-unit, test-integration, security, build)
- Unit tests for new features must exist (see QUALITY_GATES.md)
- Coverage meets current stage threshold (see QUALITY_GATES.md)
- No breaking changes to public APIs (auto-detected)
- No integration with new external services (auto-detected)

**Human Approval Required** (1 Reviewer Minimum):
- `GOVERNANCE/**` - Governance policies (Founder/CTO approval)
- `AGENTS/**` - Agent contracts, roles, prompt templates
- `.github/workflows/**` - CI/CD workflows and automation

**Risk Tier Overrides** (ALWAYS Required):
- **Tier 1 (Critical)**: Production deployment, security credential usage, payment processing, database migration, breaking API changes
- **Tier 2 (High)**: Database schema changes (DDL), external API integration, major dependency upgrades, infrastructure changes with cost impact

**Risk-Based Approval Matrix** (9 directories × 4 tiers):
- Maps directory paths to risk tiers with approval requirements
- Shows: Auto-Merge, Auto-Merge (CI only), or Human Approval Required

**Example**:
| Directory Path | T3 (Low) | T2 (High) | T1 (Critical) |
|----------------|-----------|-----------|---------------|
| APP/** | Auto-Merge | Auto-Merge | Human Approval |
| GOVERNANCE/** | Human Approval | Human Approval | Human Approval |

**Monitoring & Audit**:
- Metrics: Auto-merged PRs count, failure rate, risk tier distribution
- Alerts: Spike in auto-merged failures, auto-merge on governance, high-risk bypass

**Implementation Notes**:
- Bot configuration for auto-merge (future enhancement)
- Coverage requirements reference QUALITY_GATES.md
- Safety checks: Auto-detect multi-directory changes, risk tier overrides

---

### 2. Approval Requirements by Risk Tier (GOVERNANCE/RISK_TIERS.md)

**New Section**: "APPROVAL REQUIREMENTS BY RISK TIER" (195 lines total)

**Approval Matrix** (T0-T3):
| Tier | Human Approval | CI Required | Rollback | Auto-Merge |
|------|----------------|-------------|----------|-----------|
| T0 (Informational) | No | No | N/A | Yes |
| T3 (Low Risk) | No (Dev Fast Mode) | Yes | Yes | Yes (CI passes) |
| T2 (High Risk) | Yes (1 reviewer min) | Yes | Yes (tested) | No |
| T1 (Critical) | Yes (explicit auth) | Yes | Yes (tested+ready) | Never |

**Directory-Based Approval Override (Dev Fast Mode)**:
- Explains how Dev Fast Mode changes approval requirements
- Maps directories: Auto-Merge vs. Human Approval Required
- Emphasizes: T1/T2 ALWAYS require human approval regardless of directory

**Approval Workflow by Tier**:
- **Tier 0**: Execute freely, no gates (read-only operations)
- **Tier 3**: Create PR → CI → Auto-merge (Dev Fast Mode) or optional review
- **Tier 2**: Create PR with rollback plan → CI → Human review → Approve → Merge
- **Tier 1**: Present to human → Explicit authorization ("Factory, approved: execute [operation]") → Execute

**Reviewer Requirements**:
- **Tier 3**: Dev Fast Mode enabled: No approval (auto-merge); disabled: Any team member
- **Tier 2**: Any team member + relevant expertise (DB owner, API owner, DevOps)
- **Tier 1**: Repository Owner (Founder) or designated approver; specific approvers by domain
- **Governance Directories**: GOVERNANCE/ (Founder/CTO), AGENTS/ (impact + Founder), .github/workflows/ (DevOps + Founder)

**CODEOWNERS Integration**:
- Example configuration for: governance, agents, workflows, database, payment, app code
- ADDITIVE: CODEOWNERS requirements add to risk tier requirements
- Example:
```bash
# Governance requires Founder approval
GOVERNANCE/** @ranjan-expatready
# Application code - auto-merge allowed
APP/** *
```

---

### 3. Staged Coverage Policy (GOVERNANCE/QUALITY_GATES.md)

**New File**: Complete staged coverage policy (381 lines)

**Coverage Stages Overview**:

| Stage | Coverage Floor | Description | When to Use |
|-------|----------------|-------------|-------------|
| Stage 0 | Tests required | Initial dev, basic tests | Early development |
| Stage 1 | 70% | Minimum viable | MVP launch |
| Stage 2 | 80% | Sustainable quality | Production maturity |
| Stage 3 | 90% critical paths | High reliability | Critical infrastructure (optional) |

**Coverage Measurement**:
- **New Code Coverage** (Preferred): Measures only changed code (diff-cover tools)
- **Global Coverage** (Fallback): Measures entire codebase (standard tools)

**Coverage Configuration Examples**:
- Python: pytest-cov with .coveragerc configuration
- JavaScript/TypeScript: Jest coverage thresholds

---

#### Stage 0: Initial Development

**Coverage Requirement**:
- No explicit coverage percentage
- **Tests required for new features**
- CI must pass green

**Testing Requirements**:
- For new features: At least 1 test per function/module, happy path + 1 error case
- For bug fixes: Regression test demonstrating bug
- For config/docs: No tests required

**CI Gates**:
- Linting passes, Tests pass (if any), No syntax errors, Build succeeds, Security scan completes

**Auto-Merge Eligibility**:
- Eligible for auto-merge in APP/, PRODUCT/, BACKLOG/, FRAMEWORK_KNOWLEDGE/, ARCHITECTURE/, RUNBOOKS/
- Requires CI to pass green

**Transition to Stage 1**: Project reaches production-ready state, MVP deployment planned

---

#### Stage 1: Minimum Viable Coverage (70%)

**Coverage Requirement**: 70% coverage floor (new code or global)

**Testing Requirements** (well-tested areas):
- Business logic (features, controllers, services)
- Critical algorithms
- Data transformations
- API endpoints
- Database operations
- Authentication/authorization

**Acceptable Lower Coverage**:
- Configuration files, Data models, Simple utilities, Legacy migrations, Difficult-to-test UI

**CI Gates**:
- All Stage 0 gates
- Coverage report generated
- Coverage ≥ 70% (enforced)
- Coverage trend report visible
- Codecov or similar configured

**Coverage Exclusions** (documented rationale):
`*/config/*, */__init__.py, */generated/*, */dist/*, */tests/*, */integrations/external/*`

**Auto-Merge Eligibility**:
- Eligible for APP/, PRODUCT/, BACKLOG/ only
- Requires 70%+ coverage on new code
- Requires CI to pass green

**Transition to Stage 2**: Project proven production stability, low bug rate, team velocity allows

---

#### Stage 2: Sustainable Quality (80%)

**Coverage Requirement**: 80% coverage floor, stricter in critical paths

**Testing Requirements** (Additional layer):
- Integration tests for key workflows
- End-to-end tests for critical user journeys
- Contract tests for external APIs
- Performance benchmarks for hot paths

**Critical Path Testing** (90%+ coverage):
- Authentication/authorization flow
- Payment processing
- Data integrity operations
- Core business logic
- API contract compliance

**CI Gates**:
- All Stage 1 gates
- Coverage ≥ 80% (enforced)
- Integration tests pass
- Performance benchmarks within thresholds
- No slow tests (>5 seconds per test)

**Coverage Quality Targets**:
- Line Coverage: 80%
- Branch Coverage: 75%
- Function Coverage: 80%

**Auto-Merge Eligibility**:
- APP/, PRODUCT/ only
- Requires 80%+ coverage on new code
- Requires all integration tests to pass

**Transition to Stage 3** (Optional): High-reliability requirements, regulatory/compliance, customer demands

---

#### Stage 3: High Reliability (90%) - Optional/Targeted

**Coverage Requirement**: 90% on critical paths (targeted), 80% overall

**Testing Requirements**:
- Critical path focus: 90%+ coverage
- Chaos testing for resilience
- Load testing under stress
- Security penetration testing
- Disaster recovery testing

**Reliability Requirements**:
- SLA monitoring and alerting
- Error budget management
- Failure injection testing
- Circuit breaker validation
- Graceful degradation testing

**Coverage Quality Targets** (Stage 3):
- Line Coverage: 90% (critical), 80% (overall)
- Branch Coverage: 85% (critical), 75% (overall)
- Function Coverage: 90% (critical), 80% (overall)

**When to Use Stage 3**:
- Payment processing, Security-sensitive components, Customer data handling, Regulatory compliance, High-availability services (SLA 99.99%+)

---

**Quality Gate Escalation**:
- Coverage degradation blocked, required remediation plan
- Test failures blocked, flaky tests disabled and tracked
- Temporary waiver process with Founder approval required

**Quality Gate Monitoring**:
- Metrics: Coverage trend, test execution time, flaky test rate, coverage-driven bugs, PR failures
- Alerts: Coverage drop, time increase, flaky test rate >5%, coverage degrade >2 weeks

**Implementation Checklists**:
- Stage 0→1: Coverage tool config, 70% threshold, training, communication
- Stage 1→2: Increase to 80%, integration tests, performance benchmarks, coverage trend, update auto-merge
- Stage 2→3: Critical paths, chaos, load, security, SLA, update quality gates

**PR Requirement Checklist**:
- All PRs: CI passes, PR description, patterns, no breaking changes
- New Features: Tests, coverage threshold (Stage 1+), edge cases, regressions
- Governance: Human approval, impact review, rollback plan, doc references
- High-Risk (Tier 2+): Rollback plan, impact assessment, stakeholder notification, monitoring

---

### 4. CI Workflow Coverage Reporting (.github/workflows/ci.yml)

**New Step**: "Coverage Reporting Placeholder" (+22 lines)

**Features**:
- Active (not commented) - displays current quality gate info
- Shows current stage: Stage 0 (Tests required, no coverage threshold)
- Shows next stage: Stage 1 (70% coverage floor)
- Provides steps to enable coverage:
  1. Install coverage tools (pytest-cov, jest, etc.)
  2. Uncomment coverage test step
  3. Set coverage threshold (70% for Stage 1, 80% for Stage 2)
  4. Upload to Codecov or similar
- References GOVERNANCE/QUALITY_GATES.md for policy

**Updated Comments**: Coverage step template updated for Stage 1 threshold (70% instead of 80%)

---

## Recommended GitHub Branch Protection Settings

### Overview

To enable Dev Fast Mode and Staged Quality Gates, configure GitHub branch protection to match the policy defined in GOVERNANCE files.

### Required Branch Protection Configuration

**Navigate to**: Repository → Settings → Branches → main → Edit

---

#### 1. Require Pull Request Before Merging

**Status**: ✅ Enable

**Settings**:
- **Require at least 1 approving review**: Set to `1`
- **Dismiss stale PR approvals when new commits are pushed**: ✅ Enable
- **Require approval from a human reviewer**: ✅ Enable
- **Require review from Code Owners**: ✅ Enable

**Rationale**: Ensures human review for changes requiring approval (Governance, Agents, Workflows, Tier 1/2).

---

#### 2. Require Status Checks to Pass Before Merging

**Status**: ✅ Enable

**Settings**:
- **Require branches to be up to date before merging**: ✅ Enable

**Required Status Checks** (select all):
- ✅ `lint` - Linting and Formatting
- ✅ `test-unit` - Unit Tests
- ✅ `test-integration` - Integration Tests
- ✅ `security` - Security Checks
- ✅ `build` - Build Verification
- ✅ `summary` - CI Summary

**Rationale**: All CI checks must pass before merge. Quality gates enforce coverage thresholds.

---

#### 3. Restrictions

**Do not allow bypassing the above settings**: ✅ Enable

**Impact**:
- Even Repository Owner (Founder) must follow PR workflow
- No exceptions for critical governance changes

**Allow force pushes to main**: ❌ Disable

**Rationale**: Prevents history rewriting, maintains audit trail

**Allow deletions of main**: ❌ Disable

**Rationale**: Prevents accidental or malicious branch deletion

---

#### 4. Protection Rules

**Branch name pattern**: `main`

**Protect matching branches**: ✅ Enable

---

### CODEOWNERS Configuration (Recommended)

**Create File**: `.github/CODEOWNERS`

**Example Configuration**:
```bash
# ==== HIGH-RISK: Requires Founder Approval ====

# Governance requires Founder/CTO approval
GOVERNANCE/** @ranjan-expatready

# Agents require Founder review + agent behavior impact
AGENTS/** @ranjan-expatready

# CI/CD Workflows require DevOps review + Founder approval
.github/workflows/** @devops-team @ranjan-expatready

# ==== MEDIUM-RISK: Require Expert Review ====

# Database changes require DB owner
**/*.sql @database-team

# Payment processing requires security + finance + Founder
**/payment/** @security-team @finance-team @ranjan-expatready

# ==== LOW-RISK/FAST-MERGE: Any Team Member ====

# Application code - auto-merge allowed, optional review
APP/** *

# Product specs - auto-merge allowed, optional review
PRODUCT/** *

# Backlog items - auto-merge allowed, optional review
BACKLOG/** *

# Framework knowledge - auto-merge allowed, optional review
FRAMEWORK_KNOWLEDGE/** *

# Architecture docs - auto-merge allowed, optional review
ARCHITECTURE/** *

# Runbooks - auto-merge allowed, optional review
RUNBOOKS/** *
```

**Rules**:
- CODEOWNERS requirements are ADDITIVE to risk tier requirements
- If T1 but no CODEOWNERS: Founder approval still required
- If T3 but specific CODEOWNERS listed: Those owners must approve

---

### GitHub Auto-Merge Configuration (Future Enhancement)

**When Auto-Merge is Enabled**:

**Auto-Merge Eligible Directories**:
- APP/, PRODUCT/, BACKLOG/, FRAMEWORK_KNOWLEDGE/, ARCHITECTURE/, RUNBOOKS/

**Auto-Merge Requirements**:
- All CI checks pass
- Coverage meets current stage threshold (if enforced)
- No CODEOWNERS blocking
- No breaking changes detected

**Auto-Merge Configuration** (GitHub UI):
1. Repository → Settings → General → Pull Requests
2. Enable "Automatically merge pull requests"
3. "Auto-merge branches": `main` only
4. "Auto-merge method": Squash and merge
5. "Auto-merge when": All required checks pass

---

## Decision Matrix Summary

### Quick Reference: What Requires Review?

| Change Type | Human Review | Risk Tier | Auto-Merge |
|-------------|--------------|-----------|-----------|
| App feature (low risk) | No | T3 | Yes |
| Bug fix (APP/) | No | T3 | Yes |
| Refactor (APP/) | No | T3 | Yes |
| Production deploy | Yes (Founder) | T1 | Never |
| Security credential | Yes (Founder) | T1 | Never |
| Payment change | Yes (Founder) | T1 | Never |
| Gov policy update | Yes (Founder/CTO) | T3 (dir override) | No |
| Agent contract update | Yes (Founder) | T3 (dir override) | No |
| CI/CD workflow change | Yes (Founder) | T3 (dir override) | No |
| Database schema | Yes (expert) | T2 | No |
| External API integration | Yes (expert) | T2 | No |
| Breaking API change | Yes (expert) | T2 | No |
| Product spec update | No | T3 | Yes |
| Backlog item | No | T3 | Yes |
| Knowledge doc update | No | T3 | Yes |
| Architecture doc | No | T3 | Yes |
| Runbook update | No | T3 | Yes |

---

## Workflow Comparison

### Before This Change

**All PRs Required Human Review**:
```
Create PR → Human Review → Approve → Merge
(Minimum time: Hours or days regardless of content)
```

**Issues**:
- Slow iteration for simple changes
- Bottleneck for low-risk work
- Reduced developer autonomy

---

### After This Change

**Two Workflows**:

**Low-Risk (Fast Mode)**:
```
Create PR → CI Passes → Auto-Merge
(Time: Minutes, no review blocker)
```

**High-Risk (Standard Review)**:
```
Create PR → CI Passes → Human Review → Approve → Merge
(Time: Hours or days, as needed for governance)
```

**Benefits**:
- Fast iteration for application/product work
- Protected governance and infrastructure
- Progressive quality gates (no strict coverage initially, scales with maturity)
- Clear decision matrix for approvals

---

## Safety Guarantees

### What Remains Protected (No Changes)

✅ **Direct pushes to main** still forbidden
✅ **Governance changes** require Founder/CTO approval
✅ **Agent changes** require impact review + Founder approval
✅ **CI/CD workflow changes** require DevOps review + Founder approval
✅ **Tier 1 changes** still require explicit authorization
✅ **Tier 2 changes** still require human approval
✅ **All CI checks** must pass before merge (lint, test-unit, test-integration, security, build)

### What Becomes Faster (With Safety)

✅ **Application code features** (APP/**): Auto-merge with CI & tests
✅ **Product specs** (PRODUCT/**): Auto-merge with CI
✅ **Backlog updates** (BACKLOG/**): Auto-merge with CI
✅ **Knowledge docs** (FRAMEWORK_KNOWLEDGE/**): Auto-merge with CI
✅ **Architecture docs** (ARCHITECTURE/**): Auto-merge with CI
✅ **Runbooks** (RUNBOOKS/**): Auto-merge with CI

---

## Risk Assessment

### Low Risk

- Documentation changes only (governance policies, stages)
- No application code modifications
- No existing CI workflow changes (added placeholder only)
- Quality gates are staged, not immediately enforced (Stage 0 active)

### Operational Impact

- Auto-merge requires GitHub configuration (see recommended settings above)
- Team must adapt to directory-based rules
- Quality gate progression requires active management
- May require CODEOWNERS configuration

### Mitigations

- Tier overrides ensure high-risk changes still require approval
- Risk tier takes precedence over directory rules
- Quality gates are progressive, not strict initially
- Clear documentation and decision matrices provided
- Rollback plan: Disable Dev Fast Mode by updating policy and disabling GitHub auto-merge

---

## Success Criteria

- [ ] Fast mode enables auto-merge for APP/, PRODUCT/, BACKLOG/ with CI
- [ ] Governance directories (GOVERNANCE/, AGENTS/, Workflows/) require approval
- [ ] Risk tier overrides enforced (T1/T2 always require approval)
- [ ] Quality gates defined and documented (Stage 0-3)
- [ ] Coverage reporting visible in CI (Stage 0: display only)
- [ ] Clarity of decision matrix and workflows
- [ ] Team understands new rules and can operate efficiently
- [ ] GitHub branch protection configured per recommendations
- [ ] Optional: CODEOWNERS file created

---

## Next Steps After Merge

1. **Configure GitHub Branch Protection** (15-30 minutes):
   - Follow recommended settings section above
   - Use `RUNBOOKS/repo-governance.md` for detailed instructions

2. **Set Up CODEOWNERS** (Optional but recommended) (10-15 minutes):
   - Create `.github/CODEOWNERS` file
   - Use example configuration above
   - Test approval flow with sample PR

3. **Configure GitHub Auto-Merge** (Future):
   - When ready to enable auto-merge for eligible directories
   - Configure auto-merge method (Squash and merge)
   - Set auto-merge when: All required checks pass

4. **Monitor Metrics** (Ongoing):
   - Track auto-merged PRs vs. manually reviewed
   - Monitor coverage trends when advancing stages
   - Review failure rates for auto-merged PRs

5. **Quality Gate Progression** (Ongoing):
   - Decide: Remain Stage 0 or advance to Stage 1?
   - If Stage 1: Set up coverage tools, enforce 70% threshold
   - If Stage 2: Increase to 80%, add integration tests
   - Optional: Stage 3 for high-reliability components

---

## Reference Documentation

### In This Repository

- **GOVERNANCE/GUARDRAILS.md** - Main Branch Protection Policy, Dev Stage Fast Mode section
- **GOVERNANCE/RISK_TIERS.md** - Approval Requirements by Risk Tier section
- **GOVERNANCE/QUALITY_GATES.md** - Complete staged coverage policy
- **.github/workflows/ci.yml** - Coverage reporting placeholder step
- **RUNBOOKS/repo-governance.md** - Branch protection setup guide
- **RUNBOOKS/branch-protection-checklist.md** - Quick 2-minute checklist
- **.github/PULL_REQUEST_TEMPLATE.md** - PR template

### Related Artifacts

- **REPO_GOVERNANCE_ENFORCEMENT_ARTIFACT.md** - PR-only governance setup
- **ARABOLD_MCP_INSTALLATION_ARTIFACT.md** - Docs MCP server installation

---

## Version History

- v1.0 (2026-01-23): Initial implementation of Dev Fast Mode and Staged Quality Gates

---

## Contact Information

**Repository Owner**: ranjan-expatready
**Repository**: autonomous-engineering-os
**Primary Contact**: ranjan-expatready (via GitHub)

---

**Status**: ✅ IMPLEMENTATION COMPLETE
**Next Action**: Configure GitHub branch protection using recommended settings
**PR Created**: https://github.com/ranjan-expatready/autonomous-engineering-os/pull/2
