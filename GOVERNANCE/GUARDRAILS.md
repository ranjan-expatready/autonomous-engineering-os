# Guardrails — Approval Gates, One-Writer Rule, and Safe Terminal Policy

## Overview

This document defines the guardrails that keep the Autonomous Engineering OS operating safely, efficiently, and within its intended scope. Guardrails are not limitations — they are accelerators that enable autonomous operation by defining clear boundaries.

## DEFAULT OPERATION MODE

### Safe Autonomy Mode (MANDATORY DEFAULT)

**CRITICAL RULE": The Autonomous Engineering OS MUST operate in "Safe Autonomy Mode" by default. "Allow all commands" or "Auto High" modes are PROHIBITED as default settings.**

### Default Configuration

**Operation Mode**: Safe Autonomy (Allowlist-based Execution)
- Only commands on the approved allowlist execute autonomously
- All other commands require explicit human approval
- Risk assessment required for every command before execution
- Dry-run verification mandatory for file operations

**Risk Level**: Low/Medium (autonomous), High (requires approval)
- LOW RISK: Informational commands (read-only, no side effects) — autonomous
- MEDIUM RISK: Non-destructive operations within repo root — autonomous with logging
- HIGH RISK: Any destructive operation, system modifications, or file changes outside repo — requires human approval

### Override Mechanism

Human can temporarily elevate authorization with explicit consent:
```
"Factory, I authorize elevated execution for task [X]. Proceed with caution."
```

Elevated mode:
- Lasts for single task only
- Resets to Safe Autonomy after task completion
- Requires explicit re-authorization for subsequent tasks

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

**REQUIRE HUMAN APPROVAL BEFORE EXECUTION (ALL CATEGORIES):**

#### System Commands (NEVER Autonomous)
```bash
sudo                    # ANY command with sudo prefix
sudo rm                 # Absolutely blocked
sudo chmod, sudo chown  # Permission changes
sudo apt, sudo yum, sudo brew # System package management
sudo service, sudo systemctl # Service management
sudo iptables, sudo firewall-cmd # Firewall changes
```

#### Destructive Operations (NEVER Autonomous)
```bash
rm -rf                  # Absolutely blocked under all circumstances
rm -r                   # Requires approval, must show affected files first
rmdir                   # Requires approval
mkfs, dd, shred        # High risk filesystem operations
```

#### Package Management (NEVER Autonomous)
```bash
npm install -g          # Global package installation
pip install --user      # User-level installation outside repo
brew install            # macOS system package management
apt-get install         # Linux system package management
docker pull <image>     # Requires approval due to bandwidth and security
```

#### Docker Operations (NEVER Autonomous)
```bash
docker system prune     # Removes all unused data - destructive
docker volume rm        # Requires approval
docker network rm       # Requires approval
docker rmi -f           # Force remove images - blocked
docker-compose down -v  # Removes volumes - blocked
```

#### Secrets and Credentials (NEVER Autonomous)
```bash
# ANY operation that accesses or modifies secrets/credentials:
# Example commands that must be approved:
echo $API_KEY
cat ~/.env
chmod 600 ~/.ssh/id_rsa
ssh-keygen
openssl genrsa
# Reading or writing to: ~/.ssh, ~/.aws, ~/.config/*, ~/.credentials/*
# Any environment file operations outside repo root:
cat /etc/environment
echo export KEY > ~/.bashrc
```

#### Filesystem Writes Outside Repository Root (NEVER Autonomous)
```bash
# Writing outside the repo root requires approval:
# Repo root: /Users/ranjansingh/Desktop/autonomous-engineering-os/
# Blocked unless explicitly approved:
touch /tmp/file.txt
echo "content" > ~/config.json
cp file ~/Documents/
mkdir /var/log/app
kubectl apply -f ~/k8s/
terraform apply (state files outside repo)
```

### Permitted with Approval Examples (Additional Context)

**Require Human Approval Before Execution:**
```bash
# Database operations
psql -d production -c "..."
# Production config changes
kubectl apply -f production/
terraform apply -var-file=production.tfvars
# Large scale deletes (within repo root)
rm large-directory/ (present plan and count first)
# Container operations (production)
docker stop <production-container>
kubectl delete deployment/<name>
# Any operation with production credentials
# Any file write operation outside repository root
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

## MCP USAGE RULES

### Allowed MCP Servers

All agents are authorized to use the following MCP servers:
- **filesystem** — Read/write any file on the Mac (scoped to the repository root)
- **docs** — Inject up-to-date technical documentation into prompts (HTTP-based official MCP server)

### Built-in Tools (Not MCP Servers)

**Factory also provides built-in tools for web content fetching:**
- **FetchUrl** — Fetch web pages, APIs, and documentation (built-in tool, not an MCP server)
- **WebSearch** — Search the web for information (built-in tool, not an MCP server)

These built-in tools are used instead of a dedicated fetch MCP server to ensure reliable web content access.

### Agent Responsibilities

**Agents MUST:**
- ✅ Use filesystem MCP instead of guessing file contents
- ✅ Use docs MCP instead of relying on training data
- ✅ Use FetchUrl built-in tool to verify external facts and APIs
- ✅ Use WebSearch built-in tool to search for current information
- ✅ Verify data authenticity before acting on fetched content
- ✅ Maintain appropriate isolation between agent contexts

**Agents are NOT allowed to:**
- ❌ Access credentials, secrets, or private keys via filesystem MCP
- ❌ Modify files outside the repository unless explicitly approved
- ❌ Use fetch tools to circumvent rate limits or terms of service
- ❌ Cache sensitive data from docs MCP without proper handling

### MCP Safety Guidelines

1. **Filesystem MCP Access**
   - Limit operations to the repository root: `/Users/ranjansingh/Desktop/autonomous-engineering-os`
   - Read operations are autonomous
   - Write operations follow all existing guardrails
   - Never access system directories like `/etc`, `/var`, `~/.ssh`

2. **Docs MCP Integration**
   - Use for framework documentation, API references, best practices
   - Cross-reference with multiple sources for critical decisions
   - Consider publication dates when using docs
   - Prefer official documentation over third-party sources
   - Server runs over HTTP at `https://modelcontextprotocol.io/mcp`

3. **Built-in FetchUrl Tool Usage**
   - Verify URL authenticity before making requests
   - Respect rate limits and robots.txt
   - Validate TLS certificates automatically
   - Cache responses appropriately to reduce unnecessary fetches

### MCP Error Handling

When an MCP server fails:
1. Log the failure with context
2. Attempt fallback for docs MCP: use WebSearch or FetchUrl built-in tool
3. Notify human if critical for task completion
4. Never proceed with guessing when MCP data is unavailable

Note: FetchUrl and WebSearch are Factory built-in tools, not MCP servers. They should always be available as fallback options.

### MCP Server Management

Adding or removing MCP servers requires:
- Risk assessment of new server
- Update to this governance document
- Human approval for production-impacting changes
- Testing in safe environment before use

---

## Version History

- v1.1 (MCP Integration): Added MCP Usage Rules for filesystem, fetch, and docs servers
- v1.0 (Initial): Core guardrails, one-writer rule, approval gates, safe terminal policy
