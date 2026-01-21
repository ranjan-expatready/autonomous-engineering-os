# Risk Tiers — Classification and Required Controls

## Overview

This document defines the four-tier risk classification system used by the Autonomous Engineering OS. Every task and operation is assigned a risk tier that determines the required controls, approvals, and autonomy level.

---

## RISK TIERS OVERVIEW

| Tier | Name | Autonomy Level | Human Approval | Examples |
|------|------|----------------|----------------|----------|
| T1 | Critical | None | Always Required | Production deploy, security, payments |
| T2 | High | Limited | Required for Gates | Database changes, breaking changes |
| T3 | Medium | Conditional | Optional for Routine | Feature development, bug fixes |
| T4 | Low | High | Never Required | Documentation, formatting, small refactors |

---

## TIER 0: INFORMATIONAL

**Definition**: Read-only operations that cannot affect system state

**Characteristics:**
- No writes to system
- No irreversible actions
- No external API calls with side effects
- Zero cost impact

**Examples:**
- Reading files
- Viewing logs
- Analyzing code patterns
- Generating reports
- Reviewing test results

**Required Controls:**
- None (fully autonomous)

**Actions:**
- Execute freely without gates

---

## TIER 3: LOW RISK — High Autonomy

**Definition**: Operations that are reversible, have clear rollback paths, and minimal blast radius

**Characteristics:**
- Reversible actions
- Local scope (files, non-critical paths)
- No customer-facing impact
- Can be undone or rolled back
- Standard patterns

**Examples:**

**Code & Development:**
- Formatting and linting fixes
- Adding comments or documentation
- Small refactorings with test coverage
- Creating new feature branches
- Updating README files

**Testing:**
- Running existing test suites
- Writing new unit tests
- Mock implementations
- Test data setup

**Environment:**
- Development environment changes
- Staging deployment (most scenarios)
- Preview deployments
- Temporary feature flags

**Configuration:**
- Non-critical config updates
- Adding new config keys
- Environment variable documentation updates

**Required Controls:**
- Tests must pass (if applicable)
- No regression in existing functionality
- Documentation updated (if user-facing)

**Autonomy:**
- Full autonomy for execution
- No human approval typically required
- Proceed through gates automatically

**Gate Behavior:**
- GATE-1 (Entry): Auto-proceed
- GATE-8 (Deploy): Staging auto-proceed, production requires approval
- GATE-2 (Cost): Automatic check against thresholds

---

## TIER 2: HIGH RISK — Limited Autonomy

**Definition**: Operations with broader impact, requiring careful consideration and explicit approvals

**Characteristics:**
- Harder to reverse
- Broader blast radius
- May affect customers
- Cost considerations
- Integration complexity

**Examples:**

**Database Operations:**
- Schema migrations (tables, columns)
- Index changes
- Data migrations
- Performance-critical query changes
- Backup/restore operations

**API changes:**
- Breaking endpoint changes
- Removing deprecated endpoints
- Major version bumps
- Authentication flow changes

**Infrastructure:**
- Cloud resource scaling (significant)
- New cloud resources (cost impact)
- Network configuration
- CDN configuration changes

**Third-Party Integrations:**
- New API integrations (cost implications)
- Webhook additions/modifications
- Rate limiting changes

**Dependencies:**
- Major dependency upgrades
- Security patch updates (requires testing)
- Changing core libraries

**Required Controls:**
- Rollback plan documented and tested
- Impact assessment completed
- Tests passing (including integration tests)
- Staging deployment verified
- Security review (if applicable)
- Cost analysis documented

**Autonomy:**
- Autonomous planning and implementation
- Human approval required at key gates
- Stop and ask on ambiguity

**Gate Behavior:**
- GATE-1 (Entry): Auto-proceed
- GATE-2 (Cost): Must be under threshold OR approved
- GATE-4 (Migration): Require approval
- GATE-5 (Breaking): Require approval
- GATE-7 (External API): Require approval
- GATE-8 (Deploy): Staging autonomous, production requires approval

---

## TIER 1: CRITICAL — No Autonomy

**Definition**: Operations with significant potential for harm, requiring explicit human authorization

**Characteristics:**
- Irreversible or hard to reverse
- Direct customer impact
- Security implications
- Financial impact
- Reputation risk
- Regulatory/compliance implications

**Examples:**

**Production Deployment:**
- Deploying to production environment
- Production infrastructure changes
- Production configuration updates
- Production database operations

**Security & Compliance:**
- Authentication/authorization system changes
- Encryption key rotation
- Compliance-related code changes
- PCI/GDPR/other regulatory changes
- Vulnerability fixes (deployment)

**Financial:**
- Payment processing changes
- Billing system modifications
- Subscription logic changes
- Refund processing changes
- Pricing changes

**Data:**
- Production data deletion or modification
- Data export/transfer operations
- PII handling changes
- Data retention policy changes

**Reputation:**
- Customer-incidenting changes
- Publicly-visible product changes
- Marketing-landing page changes

**System Critical:**
- Load balancer configuration
- DNS changes
- SSL certificate management
- Core infrastructure scaling

**Required Controls:**
- Explicit human authorization before execution
- Production rollback plan tested and ready
- Incident response plan documented
- Change request prepared
- Stakeholder notification (if applicable)
- Security review completed
- Compliance review completed
- Monitoring/alerting configured
- Post-deployment verification plan

**Autonomy:**
- Zero autonomy for execution
- Can plan and present options
- Can prepare implementation
- Cannot execute without explicit human command

**Gate Behavior:**
- ALL GATES: Require human approval
- GATE-8 (Deploy): Production deploy requires explicit approval
- GATE-3 (Production Deploy): Always requires approval

---

## RISK ASSESSMENT FLOW

### Step 1: Identify Operation Type

Is this operation:
1. Read-only? → Tier 0
2. Write/modify? → Continue to Step 2

### Step 2: Determine Blast Radius

- Local to single file/function? → Tier 3 candidate
- Affects multiple components? → Tier 2 candidate
- System-wide or customer-facing? → Tier 1 candidate

### Step 3: Assess Reversibility

- Can easily roll back? → Tier 3 or lower
- Rollback requires migration/script? → Tier 2
- Rollback difficult or impossible? → Tier 1

### Step 4: Check Special Categories

- Production environment? → Tier 1 (minimum)
- Security/credentials? → Tier 1
- Payment/financial? → Tier 1
- Database schema? → Tier 2
- Breaking API change? → Tier 2

### Step 5: Final Assignment

Assign the highest applicable tier based on the above analysis.

---

## RISK TIER ELEVATION

### Automatic Elevation

Operations automatically elevate to higher tier if:

1. **Time Pressure**: Urgent deployment for hotfix → Tier 1
2. **Recent Failures**: Similar operations recently failed → +1 tier
3. **System Load**: Operating under high load → +1 tier
4. **Customer Impact**: Known customer-impacting issue → Tier 1
5. **Recent Releases**: Multiple recent production releases → +1 tier

### Manual Elevation

Human can elevate operation to higher tier at any time with explicit request:
```
"Factory, elevate this to Tier 1 due to [reason]"
```

---

## RISK DOCUMENTATION

### Required per Operation

For Tier 2 and Tier 1, must document:

1. Risk Tier with justification
2. Potential impacts (positive and negative)
3. Mitigation strategies
4. Rollback plan
5. Monitoring/alerting plan
6. Alternative approaches considered

### Location

- Per-task: Documented in BACKLOG/ item
- Per-commit: In commit message rationale
- Per-deployment: In deployment notes

---

## RISK TIER EXCEPTION HANDLING

### When Risk Tier is Unclear

If classification is ambiguous:
1. Assign higher tier (safety-first principle)
2. Document uncertainty
3. Request human clarification

### When Rapid Response is Required

Critical incidents (outages, security breaches):
1. Elevate to Tier 1
2. Incident response mode activated
3. Human approval may be given proactively
4. Document decisions made under pressure

---

## RISK TIER AUDIT

### Tracking

Maintain log of:
- Operation performed
- Risk Tier assigned
- Human approvals obtained
- Actual outcome
- Lessons learned

### Review

- Weekly: Tier 2 operations review
- Monthly: Tier 1 operations review
- Quarterly: Tier classification criteria review

---

## EXAMPLE RISK TIER ASSIGNMENTS

| Operation | Initial Tier | Rationale |
|-----------|--------------|-----------|
| Fix typo in README | T3 | Low risk, reversible, local |
| Add new API endpoint | T3 | Standard feature dev |
| Modify API response structure | T2 | Breaking change, customer impact |
| Update database column | T2 | Migration requires rollback plan |
| Deploy to staging | T3 | Non-production |
| Deploy to production | T1 | Production environment |
| Add payment processing | T1 | Financial, security, compliance |
| Change auth library | T2 | Security, requires testing |
| Update SSL certificate | T1 | Production, security, customer-facing |
| Refactor utility function | T3 | Tests cover, reversible |
| Delete unused database table | T2 | Data loss risk, migration needed |

---

## VERSION HISTORY

- v1.0 (Initial): Four-tier classification system, assessment flow, examples
