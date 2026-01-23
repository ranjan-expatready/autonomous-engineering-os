# Guardrails — Approval Gates, One-Writer Rule, and Safe Terminal Policy

## Overview

This document defines the guardrails that keep the Autonomous Engineering OS operating safely, efficiently, and within its intended scope. Guardrails are not limitations — they are accelerators that enable autonomous operation by defining clear boundaries.

---

## ONE-WRITER RULE (Critical)

### Rule Statement

**Only Factory (the Autonomous Engineering OS system) writes to the repository. External AIs function in an advisory capacity only.**

### What This Means

**Factory (Permitted Actions):**
- ✅ Write code directly to the repository
- ✅ Create and delete files
- ✅ Execute git commits
- ✅ Modify configuration files
- ✅ Run build and deployment commands
- ✅ Install dependencies via package managers
- ✅ Execute tests and linters

**External AIs (Advisor Role Only):**
- ❌ Never write to the repository
- ❌ Never execute git commands
- ❌ Never modify files directly
- ✅ Provide code suggestions and explanations
- ✅ Answer technical questions
- ✅ Review and analyze code
- ✅ Offer architecture advice
- ✅ Explain trade-offs

### Enforcement

1. **Before any write operation**, the system must verify the source is Factory
2. **Human must explicitly opt-in** to any external AI suggestions
3. **Code suggestions from external AIs** must be:
   - Presented as text/plain
   - Requires explicit "Factory, please implement this" instruction from human
   - Reviewed by Factory before applying

### Violation Handling

If an external AI attempts to write:
1. Immediate stop with clear warning
2. Explain one-writer rule
3. Offer to proceed as advisory-only session
4. Require explicit confirmation from human to ignore (not recommended)

---

## APPROVAL GATES

### Gate Classification System

| Gate Trigger | Risk Tier | Action |
|--------------|-----------|--------|
| Cost projection > $100 | Tier 2 | Wait for human |
| Production deploy | Tier 1 | Require human approval |
| Database migration | Tier 2 | Require human approval |
| Security credential usage | Tier 1 | Require human approval |
| Breaking API change | Tier 2 | Require human approval |
| Delete > 10 files | Tier 3 | Autonomous if tests pass |
| Modify production config | Tier 1 | Require human approval |
| External API integration | Tier 2 | Cost assessment + approval |
| Payment processing change | Tier 1 | Require human approval |

### Detailed Gate Definitions

#### GATE-1: Task Entry Validation
**Trigger**: New task received from human
**Risk Tier**: Assessed during PLANNING state
**Actions**:
- Validate task clarity
- Assign initial risk tier
- Check for conflicting priorities
- Determine if specialized agent needed
**Proceed**: Always proceed to PLANNING

#### GATE-2: Cost Threshold Approval
**Trigger**: Estimated cost exceeds threshold
**Thresholds**:
- Single task: $50 (warn), $100 (require approval)
- Daily cumulative: $200 (warn), $400 (pause)
- Monthly: $2000 (require review)
**Actions**:
- Calculate cost breakdown
- Present to human with alternatives
- Pause execution until approval
**Proceed**: Human approval OR cost reduced under threshold

#### GATE-3: Production Deployment Authorization
**Trigger**: Deployment to production environment
**Risk Tier**: Tier 1 (always requires approval)
**Actions**:
- Verify all tests passing
- Confirm rollback plan exists
- Present deployment summary
- Require explicit human confirm
**Proceed**: Only after explicit human approval
**Note**: Staging/preview deployments are Tier 3 (autonomous)

#### GATE-4: Database Migration Approval
**Trigger**: Any DDL statement (CREATE, ALTER, DROP)
**Risk Tier**: Tier 2 (medium-high risk)
**Actions**:
- Generate migration script
- Create rollback script
- Estimate execution time
- Request human approval
**Proceed**: Human approval after reviewing both scripts

#### GATE-5: Breaking Change Notification
**Trigger**: Changes that could break dependent systems
**Risk Tier**: Tier 2 (medium risk)
**Examples**:
- API endpoint removal or signature change
- Database schema modification
- Authentication flow change
- Critical data structure change
**Actions**:
- Identify affected components
- Generate migration guide
- Assess customer impact
- Request human approval
**Proceed**: Human approval after impact review

#### GATE-6: Configuration Change Authorization
**Trigger**: Modification of production config files
**Risk Tier**: Tier 1 (if production), Tier 3 (if dev/staging)
**Actions**:
- Validate config syntax
- Document change reason
- Create rollback config
**Proceed**: Production → approval required; dev/staging → autonomous

#### GATE-7: External API Integration Approval
**Trigger**: New external API or service integration
**Risk Tier**: Tier 2 (medium risk)
**Actions**:
- Cost analysis of API usage
- Availability and SLA review
- Security assessment
**Proceed**: Human approval after cost/risk review

#### GATE-8: Payment Processing Change
**Trigger**: Any modification to payment flow or integration
**Risk Tier**: Tier 1 (critical)
**Actions**:
- Security review mandatory
- Compliance check (PCI, GAAP if applicable)
- Testing in sandbox environment
**Proceed**: Human approval after security review

#### GATE-9: Ambiguity Resolution Gate
**Trigger**: Task requirements are unclear or conflicting
**Risk Tier**: Dynamic (based on context)
**Actions**:
- Identify specific ambiguities
- Propose clarifying questions
- Pause execution
**Proceed**: Human provides clear answers

---

## SAFE TERMINAL POLICY

### Command Execution Safety

All terminal commands executed by Factory must pass safety validation:

### Approved Command Patterns

**Read Operations (LOW RISK - autonomous):**
```bash
# File reading
cat, less, head, tail
# Directory listing
ls, find (no -delete or -exec)
# Git read operations
git status, git log, git diff, git show
# System info
ps, top, df, du, date
npm list, pip list
```

**Build Operations (MEDIUM RISK - autonomous):**
```bash
# Building
npm run build, npm run compile
python -m build
make build
# Linting and formatting
npm run lint, npm run format
black, prettier, eslint
# Testing
npm test, pytest, cargo test
```

**Dependency Management (MEDIUM RISK - autonomous):**
```bash
# Installing dependencies (non-root)
npm install, npm add
pip install, poetry install
cargo add
```

**Git Write Operations (MEDIUM RISK - autonomous):**
```bash
git add
git commit (no --amend, no --force)
git checkout (no -f flag)
git branch, git merge (no --force)
```

### Prohibited Command Patterns

**NEVER Execute (HIGH RISK - blocked):**
```bash
# Destructive operations
rm, rmdir (any form)
rm -rf (BLOCKED ABSOLUTELY)
mkfs, dd, shred
# System modification (with sudo)
sudo apt, sudo yum, sudo brew
sudo chmod, sudo chown
sudo service, sudo systemctl
# Network exposure (production)
sudo iptables, sudo firewall-cmd
# Security risks
curl | bash, wget | sh eval $(curl)
# Force git operations
git push --force, git push -f
git reset --hard (on shared branches)
git clean -f (without -n dry-run first)
```

### Permitted with Approval

**Require Human Approval Before Execution:**
```bash
# Database operations
psql -d production -c "..."
# Production config changes
kubectl apply -f production/
terraform apply -var-file=production.tfvars
# Large scale deletes
rm large-directory/ (present plan and count first)
# System-wide changes
npm install -g <package>
docker system prune
# Container operations (production)
docker stop <production-container>
kubectl delete deployment/<name>
```

### Command Execution Flow

1. **Validate**: Check against approved/prohibited lists
2. **Analyze**: Determine risk tier
3. **Context-Check**: Verify working directory is correct
4. **Quote**: Quote paths with spaces and special characters
5. **Execute**: Run with timeout
6. **Monitor**: Watch for expected output
7. **Verify**: Confirm success before proceeding

### Safety Override

Human can override safety checks with explicit command:
```
"Factory, execute this high-risk command despite guardrails: [command]"
```

Required responses to proceed with override:
1. Clear acknowledgment of risk
2. Description of what will happen
3. Confirm "proceed at your own risk"

---

## VIOLATION HANDLING

### When a Guardrail is Violated

1. **Immediate Stop**: Halt current execution
2. **Explain**: Clearly state which guardrail was triggered
3. **Context**: Show the specific action that caused violation
4. **Path Forward**: Offer options:
   - Adjust action to stay within guardrails
   - Request explicit override from human
   - Suggest alternative approach

### Severity Levels

**Informational** (Log and continue):
- Approached but did not cross threshold
- Multiple small operations that sum to large

**Warning** (Pause and notify):
- Approaching cost threshold
- Unusual pattern detected
- Best practice not followed

**Critical** (Block and require approval):
- Security risk detected
- Production deployment attempt
- Database migration without approval
- One-writer rule violation

---

## GUARDRAIL MAINTENANCE

### Updating Guardrails

Guardrails themselves cannot be modified autonomously. To update:

1. Human proposes change
2. Factory analyzes impact
3. Update committed with clear rationale
4. Version history maintained

### Review Cycle

Guardrails should be reviewed:
- Monthly: Cost thresholds
- Quarterly: Risk tier assignments
- On incident: Related guardrails
- Before major feature: Relevant gates

---

## DOCUMENTATION SOURCES POLICY

### Overview

This section defines how agents should use different documentation sources to ensure accurate, up-to-date, and reliable information when working with frameworks, libraries, and APIs.

### Documentation Sources

**1. Repo Doctrine (project-specific knowledge)**
- Location: `FRAMEWORK_KNOWLEDGE/` directory, inline code comments, README files
- When to use: For project-specific patterns, decisions, architecture, and conventions
- Priority: **HIGHEST** - Always check repo doctrine first before external sources

**2. docs (Official MCP Documentation Server)**
- Type: HTTP-based MCP server
- URL: `https://modelcontextprotocol.io/mcp`
- When to use: For official Model Context Protocol documentation and reference
- Priority: HIGH - Use as primary source for MCP protocol specifics

**3. docs_arabold (Arabold Docs MCP Server)**
- Type: Stdio-based MCP server with local indexing
- Capabilities: Index and search documentation from libraries, frameworks, and APIs
- When to use: For technical documentation of third-party libraries, frameworks, APIs
- Features: Supports versioned documentation, semantic search, local caching
- Priority: HIGH - Use for versioned APIs, framework documentation, and technical facts

**4. context7 (Remote Context Provider)**
- Type: HTTP-based MCP server
- URL: `https://mcp.context7.com/mcp`
- When to use: For general context and knowledge retrieval when local sources are insufficient
- Priority: MEDIUM - Use as fallback when other sources lack specific information

**5. Built-in Tools (Factory Native)**
- **FetchUrl**: Built-in tool for fetching web pages, APIs, and documentation
- **WebSearch**: Built-in tool for searching the web for current information
- When to use: For recent news, current events, or when other documentation sources are unavailable
- Priority: LOW - Use only as last resort or for time-sensitive information

### Priority Order

**Documentation Source Hierarchy** (from highest to lowest priority):

1. **Repo Doctrine** → Project-specific knowledge (FRAMEWORK_KNOWLEDGE/)
2. **docs MCP** → Official MCP protocol documentation
3. **docs_arabold MCP** → Indexed library/framework documentation
4. **context7 MCP** → Remote context knowledge
5. **Built-in Tools (WebSearch/FetchUrl)** → Web search and page fetching

### Decision Matrix

| Question | Recommended Source |
|----------|-------------------|
| How does this project pattern work? | Repo Doctrine (FRAMEWORK_KNOWLEDGE/) |
| What is the MCP tool format? | docs MCP (official server) |
| What is the React useEffect API signature? | docs_arabold MCP (versioned docs) |
| How do I implement JWT authentication? | docs_arabold MCP (auth library docs) |
| What's the industry standard for X? | context7 MCP or WebSearch |
| What are recent updates to framework Y? | WebSearch (current information) |

### Usage Guidelines

**Mandatory Requirements**:
1. **Always check repo doctrine first** for project-specific patterns and decisions
2. **Use docs_arabold for versioned APIs** - do not rely on memory for API signatures that change
3. **Prefer docs_arabold over WebSearch** for established frameworks and libraries
4. **Validate documentation publication dates** when using docs_arabold for critical decisions
5. **Document the source** when referencing external documentation in code or decisions

**When to Use Each Source**:

**Repo Doctrine**:
- Project architecture decisions
- Code patterns and conventions
- Framework choices and rationale
- Team best practices

**docs MCP**:
- Official MCP protocol tool formats
- MCP server implementation guidelines
- Factory-specific MCP capabilities

**docs_arabold MCP**:
- Framework API documentation (React, Vue, Django, etc.)
- Library reference documentation
- Version-specific API changes
- Technical implementation details

**context7 MCP**:
- General knowledge queries
- Industry best practices
- Coding patterns when local docs are insufficient

**WebSearch/FetchUrl**:
- Recent news and announcements
- Current API pricing or policies
- Very new libraries not yet indexed
- Finding additional sources for complex problems

### Documentation Verification

Before acting on documentation:
1. **Check publication date** - Prefer recent documentation
2. **Verify version compatibility** - Ensure docs match library version in use
3. **Cross-reference multiple sources** - For critical decisions
4. **Check for deprecation warnings** - Ensure APIs are not deprecated
5. **Test in safe environment** - Before applying to production code

---

## STATE MANAGEMENT POLICY

### Overview

The Autonomous Engineering OS maintains determinism through state tracking in the STATE/ directory. All state changes are tracked in STATUS_LEDGER.md and LAST_KNOWN_STATE.md to enable deterministic resume after interruption.

### State Files

**STATE/STATUS_LEDGER.md**:
- Human-readable ledger of current system state
- Tracks: objectives, issues, PRs, artifacts, blockers, next actions, risk tier, gates
- Updated continuously during operations
- Read during resume protocol to understand current state

**STATE/LAST_KNOWN_STATE.md**:
- Concise snapshot at meaningful milestones
- Captured after: planning completed, PR opened, CI passed, deploy ready, etc.
- Provides deterministic resume point after interruption
- Updated after every state machine transition

### PR State Update Requirements

**MANDATORY**: Every pull request MUST update STATE/STATUS_LEDGER.md and STATE/LAST_KNOWN_STATE.md.

**When to Update STATE Files**:

**Within the PR** (Preferred):
- Include STATE/STATUS_LEDGER.md update in the PR itself
- Include STATE/LAST_KNOWN_STATE.md update in the PR itself
- Document state changes in commit messages
- Reviewers must verify STATE files are updated

**Immediately After Merge** (If not in PR):
- If PR does not include STATE file updates, update immediately after merge
- Commit: "chore: update STATE ledger and last known state"
- Push to main (no PR required for STATE updates only)
- Verify updates reflect merged changes

**Which PRs Must Update STATE**:

**ALL PRs must update STATE files**, including:
- Application code changes (APP/**)
- Product specification changes (PRODUCT/**)
- Backlog updates (BACKLOG/**)
- Governance changes (GOVERNANCE/**)
- Agent changes (AGENTS/**)
- Workflow changes (.github/workflows/**)
- Documentation changes (any directory)
- Infrastructure changes
- ANY change to the repository

**Exceptions**: None. All PRs must update state.

### STATE Update Checklist

Before merging a PR, the PR MUST include updates to:

**STATE/STATUS_LEDGER.md**:
- [ ] Current objective updated (if changed)
- [ ] Active issues updated (new issues added/closed)
- [ ] Active PRs updated (PR opened/closed/merged)
- [ ] Last completed artifact updated (if any artifact delivered)
- [ ] Current blockers updated (new blockers added/resolved)
- [ ] Next actions updated (prioritized list)
- [ ] Current risk tier updated (if changed)
- [ ] Required gates status updated (if gates cleared/new gates)

**STATE/LAST_KNOWN_STATE.md**:
- [ ] State machine position updated (IDLE/PLANNING/EXECUTING/WAITING_FOR_HUMAN)
- [ ] Active task updated (if changed)
- [ ] Work-in-progress items updated (if changed)
- [ ] GitHub state updated (branch, commits, issues, PRs)
- [ ] CI/CD state updated (if CI run completed)
- [ ] Risk assessment updated (if changed)
- [ ] Quality state updated (if tests/coverage changed)
- [ ] Governance compliance updated (if guardrails changed)
- [ ] Agent coordination updated (if handoffs)
- [ ] Blockers updated (added/resolved)
- [ ] Next actions updated (ordered list)
- [ ] Context preservation updated (if important context changed)
- [ ] Validation checks completed

### PR Templates with STATE Updates

When creating a PR, the PR description MUST include a section:

```markdown
## STATE Updates

### STATUS_LEDGER.md Updates
- Current objective: [Updated/Unchanged]
- Active issues: [Added/Closed X issues]
- Active PRs: [Added/Closed X PRs]
- Last completed artifact: [New artifact / Unchanged]
- Current blockers: [Added/Resolved X blockers]
- Next actions: [Updated priority list]

### LAST_KNOWN_STATE.md Updates
- State machine position: [IDLE/PLANNING/EXECUTING/WAITING_FOR_HUMAN]
- Active task: [Updated to: / Unchanged]
- GitHub state: [Branch updated / PRs updated / Unchanged]
- CI/CD state: [CI results updated / Unchanged]
- Risk tier: [Updated to: / Unchanged]
- Quality state: [Coverage: X% updated / Tests updated / Unchanged]
```

### Reviewer Responsibility When Reviewing PRs

**Checklist for PR Reviewers**:
```bash
[ ] STATE/STATUS_LEDGER.md included in PR (or will be updated immediately after merge)
[ ] STATE/LAST_KNOWN_STATE.md included in PR (or will be updated immediately after merge)
[ ] Current objective correctly reflects PR changes
[ ] Active issues list is accurate
[ ] Active PRs list is accurate
[ ] Last completed artifact documented (if PR delivers artifact)
[ ] Current blockers documented (if any)
[ ] Next actions prioritized correctly
[ ] Risk tier assessment is correct
[ ] Required gates status is accurate

If STATE files are NOT in PR:
[ ] Confirm PR description states "STATE files will be updated immediately after merge"
[ ] Verify commit message includes STATE update
[ ] Reviewer should merge with expectation of STATE update
```

### STATE File Validation

**Pre-Merge Checks** (automated or manual):

1. Check existence:
   - STATE/STATUS_LEDGER.md must exist
   - STATE/LAST_KNOWN_STATE.md must exist

2. Check content:
   - STATUS_LEDGER.md should not be empty template
   - LAST_KNOWN_STATE.md should have current state populated
   - No placeholder values (e.g., "[CURRENT OBJECTIVE]" should be filled)

3. Check consistency:
   - STATUS_LEDGER and LAST_KNOWN_STATE should be consistent
   - GitHub state should match STATE files
   - CI status should match STATE files

4. Check completeness:
   - All required fields filled
   - No sections skipped (unless N/A)
   - Timestamps populated
   - Links valid

**State File Validation Failure**:
- If STATE files not updated: BLOCK PR and require update
- If STATE files have placeholder values: BLOCK PR and require fix
- If STATE files are inconsistent: BLOCK PR and require fix

---

## MAIN BRANCH PROTECTION POLICY

### Overview

This section defines the mandatory branch protection rules for the `main` branch. These rules enforce PR-only governance, ensuring all changes go through proper review and validation before merging to main.

### Core Principle

**DIRECT PUSHES TO MAIN ARE FORBIDDEN.**

All changes to the `main` branch MUST occur through pull requests with required reviews and checks.

### Required Branch Protection Settings

The following settings MUST be configured on the `main` branch in GitHub:

#### 1. Require Pull Request Before Merging
**Setting**: Enable "Require a pull request before merging"

**Requirements**:
- Require at least 1 approving review before merging
- Dismiss stale PR approvals when new commits are pushed
- Require approval from a human reviewer (not automated)
- Require review from CODEOWNERS when defined

#### 2. Require Status Checks to Pass Before Merging
**Setting**: Enable "Require status checks to pass before merging"

**Required Checks** (from `.github/workflows/ci.yml`):
- `lint` - Linting and Formatting checks
- `test-unit` - Unit Tests
- `test-integration` - Integration Tests
- `security` - Security Checks
- `build` - Build Verification
- `summary` - CI Summary

**Additional Settings**:
- Require branches to be up to date before merging
- Require approval from all code owners when available

#### 3. Disallow Force Pushes to Main
**Setting**: Enable "Do not allow bypassing the above settings"

**Impact**:
- `git push --force` to main is blocked
- `git push -f main` is blocked
- History rewriting on main is impossible
- Ensures audit trail is preserved

#### 4. Disallow Deletion of Main
**Setting**: Enable "Restrict deletions"

**Impact**:
- Branch cannot be deleted via GitHub UI
- `git push origin --delete main` is blocked
- Prevents accidental or malicious branch removal

### Review Requirements by Directory

Different directories require different levels of approval:

#### High-Risk Directories (Require Human Approval - 1 reviewer minimum)

Changes to the following directories **MUST** receive at least 1 human approval:

```
GOVERNANCE/           # Governance policies (guardrails, cost policy, risk tiers)
AGENTS/               # Agent contracts, roles, best practices, prompt templates
.github/workflows/    # CI/CD workflows and automation
```

**Rationale**: These directories define the operating rules, agent behaviors, and automated processes. Changes here can fundamentally alter how the Autonomous Engineering OS operates.

#### Medium-Risk Directories (Automated Review or 1 reviewer)

Changes to these directories may proceed with automated checks or require 1 reviewer:

```
FRAMEWORK_KNOWLEDGE/  # Technical knowledge base
ARCHITECTURE/         # System architecture documentation
PRODUCT/              # Product specifications and requirements
RUNBOOKS/             # Operational procedures
```

**Rationale**: These directories contain knowledge and documentation but have less operational impact than governance files.

#### Low-Risk Directories (Automated Review Only)

Changes in these directories require only CI checks to pass:

```
APP/                  # Application code (when populated)
COMPLETION_STATUS.md  # Project progress tracking
INITIALIZATION_SUMMARY.md  # Project initialization records
README.md             # Project documentation
```

**Rationale**: Application code follows standard software engineering practices with automated testing and quality gates.

---

### Dev Stage Fast Mode

**Purpose**: Enable rapid development iterations for low-risk changes while maintaining safety for core governance and infrastructure.

#### Core Principle

**Speed with Safety**: Low-risk changes can merge autonomously with CI passing, while governance and infrastructure changes retain human review requirements.

#### Directory-Based Auto-Merge Rules

**Auto-Merge with CI Only** (No Human Review Required):

Changes in the following directories can merge automatically when all CI checks pass:

```
APP/**                # Application code (features, bug fixes, refactors)
PRODUCT/**            # Product specifications, requirements, user stories
BACKLOG/**            # Backlog items, task definitions, tickets
FRAMEWORK_KNOWLEDGE/** # Technical knowledge and best practices
ARCHITECTURE/**       # Architecture documentation
RUNBOOKS/**           # Operational procedures and guides
```

**Requirements for Auto-Merge**:
- All CI checks must pass (lint, test-unit, test-integration, security, build)
- Unit tests for new features must exist (see QUALITY_GATES.md)
- Coverage meets current stage threshold (see QUALITY_GATES.md)
- No breaking changes to public APIs (auto-detected)
- No integration with new external services (auto-detected)

**Review**: Optional (bot or human can review at discretion)

---

**Human Approval Required** (1 Reviewer Minimum):

Changes in the following directories **MUST** receive human approval before merging:

```
GOVERNANCE/**          # Governance policies, guardrails, cost policy, risk tiers
AGENTS/**              # Agent contracts, roles, best practices, prompt templates
.github/workflows/**   # CI/CD workflows and automation
```

**Rationale**: These directories define the core operating rules, agent behaviors, and automated execution policies. Changes here fundamentally alter how the Autonomous Engineering OS operates and require human judgment.

**Requirements**:
- All CI checks must pass
- At least 1 human reviewer approval
- Changes to GOVERNANCE/ require explicit Founder/CTO approval
- Changes to AGENTS/ require explicit review of agent behavior impact
- Changes to .github/workflows/ require explicit review of CI/CD impact

---

#### Risk Tier Overrides

**Regardless of directory**, human approval is **ALWAYS REQUIRED** for:

**Tier 1 (Critical) Changes**:
- Production deployment (GATE-3)
- Security credential usage
- Payment processing changes
- Database migration on production
- Breaking API changes

**Tier 2 (High) Changes**:
- Database schema changes (DDL)
- External API integration (new third-party service)
- Major dependency upgrades
- Infrastructure changes with cost impact

**Rationale**: Certain changes carry inherent risk regardless of which directory they touch. Risk tier classification takes precedence over directory-based rules.

---

#### Risk-Based Approval Matrix

| Directory Path | T0 (Info) | T3 (Low Risk) | T2 (High Risk) | T1 (Critical) |
|----------------|-----------|---------------|----------------|---------------|
| APP/** | N/A | Auto-Merge (CI only) | Auto-Merge (CI only) | Human Approval Required |
| PRODUCT/** | Auto-Merge | Auto-Merge (CI only) | Auto-Merge (CI only) | Human Approval Required |
| BACKLOG/** | Auto-Merge | Auto-Merge (CI only) | Auto-Merge (CI only) | Human Approval Required |
| FRAMEWORK_KNOWLEDGE/** | Auto-Merge | Auto-Merge (CI only) | Human Approval Required | Human Approval Required |
| ARCHITECTURE/** | Auto-Merge | Auto-Merge (CI only) | Human Approval Required | Human Approval Required |
| RUNBOOKS/** | Auto-Merge | Auto-Merge (CI only) | Human Approval Required | Human Approval Required |
| GOVERNANCE/** | Auto-Merge | Human Approval Required | Human Approval Required | Human Approval Required |
| AGENTS/** | Auto-Merge | Human Approval Required | Human Approval Required | Human Approval Required |
| .github/workflows/** | N/A | Human Approval Required | Human Approval Required | Human Approval Required |

**Legend**:
- **Auto-Merge**: Merges automatically when CI passes (no review)
- **Auto-Merge (CI only)**: Merges automatically when CI passes (optional review)
- **Human Approval Required**: Requires at least 1 human review before merge

---

#### Implementation Notes

**Bot Configuration** (Future Enhancement):
- Configure GitHub bot to auto-merge eligible PRs when CI passes
- Use GitHub's Auto-merge feature with required status checks
- Optional: Configure CODEOWNERS to auto-approve designated directories

**Coverage Requirements**:
- See `GOVERNANCE/QUALITY_GATES.md` for staged coverage requirements
- Current stage enforces coverage thresholds before auto-merge

**Safety Checks**:
- System will auto-detect if change touches multiple directories
- If change spans both auto-merge and approval-required paths → Approval required
- Risk tier classification always overrides directory rules when higher-risk

---

#### Monitoring and Audit

**Metrics to Track**:
- Auto-merged PRs count vs. manually reviewed PRs
- Failure rate of auto-merged PRs (CI failures, regressions)
- Risk tier distribution of merged PRs
- Directory distribution of merged PRs
- Average time from PR creation to merge (auto vs manual)

**Alerts**:
- Spike in auto-merged PR failures (indicates weak CI)
- Auto-merge attempts on governance directories (blocked by policy)
- High-risk changes bypassing review (policy violation)

---

#### Transition Process

**If Dev Fast Mode is Disabled**:
All changes revert to requiring human approval regardless of directory.

To enable/disable Dev Fast Mode:
1. Update this section policy
2. Configure or disable GitHub auto-merge rules
3. Document change in governance changelog

---

### Enforced Workflow

The REQUIRED workflow for changes to main:

```
Step 1: Create Feature Branch
   git checkout -b feature/your-feature-name

Step 2: Make and Commit Changes
   git add .
   git commit -m "descriptive commit message"
   git push -u origin feature/your-feature-name

Step 3: Open Pull Request
   - Use GitHub UI to create PR from branch to main
   - Fill in PR template (.github/PULL_REQUEST_TEMPLATE.md)
   - Tag appropriate reviewers based on changed directories

Step 4: CI Checks Run
   - All CI checks must pass (lint, test, security, build)
   - Fix any failing checks before proceeding

Step 5: Code Review
   - Human reviewer(s) review changes
   - Approve if quality and governance standards are met
   - Request changes if improvements are needed

Step 6: Merge
   - After approval and CI passing, merge the PR
   - Use "Squash and merge" or "Rebase and merge" (not "Merge commit")
   - Delete feature branch after merge
```

### Bypassing Branch Protection

**CRITICAL RULE**: Branch protection settings should NOT allow bypassing by administrators, maintainers, or human reviewers.

**Rationale**: Even the Repository Owner (Founder) must follow the same governance rules. No exceptions.

If an emergency fix is required:
1. Create a hotfix branch following standard workflow
2. Create PR with "emergency/hotfix" label
3. Request expedited review
4. Merge with proper approvals after review

### Codeowners (Recommended)

Create a `.github/CODEOWNERS` file to define who must approve changes to specific directories:

**Example CODEOWNERS**:
```
# High-risk directories require Founder approval
GOVERNANCE/           @ranjan-expatready
AGENTS/               @ranjan-expatready
.github/workflows/    @ranjan-expatready

# Framework knowledge can be reviewed by any contributor
FRAMEWORK_KNOWLEDGE/  *

# Application code follows standard review process
APP/                  *

# Documentation updates are low-risk
*.md                  * @ranjan-expatready
```

### Violation Detection

If someone attempts to push directly to main, GitHub will reject the push with:

```
ERROR: Protected branch update failed for main.
At least 1 approving review is required by reviewers with write access.
```

If CI checks are failing:
```
ERROR: Protected branch update failed for main.
Required status check "lint" is expected (2/2 required).
```

### Monitoring and Alerts

GitHub provides built-in notifications for:
- Branch protection rule violations
- Failing CI checks on PRs
- Approvals requested
- Merge events

Configure GitHub repository settings to receive:
- Email notifications for PR reviews
- Slack/Discord notifications for CI failures
- Security alerts for vulnerabilities

### Periodic Review

Branch protection rules should be reviewed:
- **Quarterly**: Ensure all required checks exist and pass
- **On major framework changes**: Update review requirements if needed
- **After security incidents**: Strengthen protection if necessary

### Compliance Verification

**Checklist for verifying branch protection compliance**:

```
[ ] 1. Branch protection is enabled on main
[ ] 2. Require PR before merging is enabled
[ ] 3. At least 1 human approval is required
[ ] 4. All CI checks are required to pass
[ ] 5. Force pushes are disabled
[ ] 6. Branch deletion is disabled
[ ] 7. No bypass of rules is allowed for admins
[ ] 8. CODEOWNERS file is configured (recommended)
[ ] 9. CI workflow runs on every PR
[ ] 10. Recent PRs followed the enforcement rules
```

### Reference Files

- **Branch Protection Setup Guide**: See `RUNBOOKS/repo-governance.md` for step-by-step GitHub UI instructions
- **CI Workflow Definition**: See `.github/workflows/ci.yml` for required checks
- **PR Template**: See `.github/PULL_REQUEST_TEMPLATE.md` for PR format requirements

---

## COMPLIANCE STATEMENT

Every action taken by the Autonomous Engineering OS must satisfy:

1. ✅ One-writer rule not violated
2. ✅ Relevant approval gates evaluated
3. ✅ Command safety rules followed
4. ✅ Cost within thresholds
5. ✅ Risk acceptable for context
6. ✅ Rollback plan exists for irreversible actions
7. ✅ Human approval obtained for Tier 1 gates
8. ✅ Main branch protection policy followed (PR-only workflow)
9. ✅ STATE files updated for every PR (STATUS_LEDGER.md + LAST_KNOWN_STATE.md)

---

## Version History

- v1.2 (Dev Fast Mode): Added Dev Stage Fast Mode for directory-based auto-merge with CI
- v1.1 (Branch Protection): Added Main Branch Protection Policy and PR-only enforcement
- v1.0 (Initial): Core guardrails, one-writer rule, approval gates, safe terminal policy
