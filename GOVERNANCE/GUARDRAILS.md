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

## COMPLIANCE STATEMENT

Every action taken by the Autonomous Engineering OS must satisfy:

1. ✅ One-writer rule not violated
2. ✅ Relevant approval gates evaluated
3. ✅ Command safety rules followed
4. ✅ Cost within thresholds
5. ✅ Risk acceptable for context
6. ✅ Rollback plan exists for irreversible actions
7. ✅ Human approval obtained for Tier 1 gates

---

## Version History

- v1.0 (Initial): Core guardrails, one-writer rule, approval gates, safe terminal policy
