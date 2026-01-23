# Resume Protocol — How the CTO Agent Restores System State

## Overview

When the CTO Agent is interrupted and later resumes, it must reference the GitHub Project "Autonomous Engineering OS — SDLC" as the live SDLC state reference. This document defines the resume protocol for the CTO Agent.

---

## LIVE SDLC STATE REFERENCE

**The GitHub Project "Autonomous Engineering OS — SDLC" is the authoritative source of truth for all work state.**

### Project Location
- **Name**: Autonomous Engineering OS — SDLC
- **Type**: GitHub Project v2
- **Owner**: `ranjan-expatready/autonomous-engineering-os`
- **URL**: TBD (to be created as part of SDLC setup)

### State Machine Mapping

The GitHub Project board columns map directly to the CTO state machine states:

| GitHub Project Column | CTO State Machine State | Description |
|----------------------|-------------------------|-------------|
| **Backlog** | IDLE → PLANNING Pending | New items awaiting planning |
| **Planned** | PLANNING | Items being planned and designed |
| **In Progress** | EXECUTING | Active implementation work |
| **In Review (PR Open)** | EXECUTING → WAITING_FOR_VALIDATION | Pull requests open, awaiting review |
| **Waiting for Approval** | WAITING_FOR_HUMAN | Items blocked by human approval gates |
| **Blocked** | WAITING_FOR_HUMAN | Items blocked by issues or risks |
| **Ready for Release** | EXECUTING Complete | Items ready for deployment |
| **Done** | IDLE | Completed work |

---

## RESUME PROCEDURE

### Step 1: Load Context from GitHub Project

When resuming, the CTO Agent must:

1. **Access the GitHub Project**
   ```
   Navigate to: https://github.com/ranjan-expatready/autonomous-engineering-os/projects
   Select: "Autonomous Engineering OS — SDLC"
   ```

2. **Read Current State**
   - Review all issues in each column
   - Identify items in "In Progress", "In Review", "Waiting for Approval", or "Blocked" columns
   - Note the custom field values (Type, Risk Tier, Owner, Release)

3. **Identify Active Work**
   - Find the most recent "In Progress" item
   - Check associated pull requests
   - Review any pending approvals

### Step 2: Reconstruct System State

From the GitHub Project, reconstruct:

```
Current State Analysis:
- Active work: [Issue #XXX - Title]
- Location in workflow: [Column name]
- Associated PRs: [PR #YYY - Title]
- Risk Tier: [T0/T1/T2/T3]
- Owner: [Product/Code/Reliability/Knowledge/Advisor]
- Release: [Release name or TBD]
- Status notes: [From issue comments or PR discussions]
```

### Step 3: Verify Against STATE_LEDGER

Cross-reference the GitHub Project state with the `STATE_LEDGER.json`:

```json
{
  "last_execution": {
    "timestamp": "2024-01-23T14:30:00Z",
    "agent": "CTO Agent",
    "state": "PLANNING",
    "active_issue": "XXX",
    "context": {
      "task": "[current task description]",
      "progress": "[progress summary]",
      "next_actions": "[next steps]"
    }
  },
  "decisions_made": [...],
  "pending_approvals": [...]
}
```

**Validation Rules**:
- If GitHub Project and STATE_LEDGER agree → Resume seamlessly
- If conflict detected → Prioritize GitHub Project (it's the source of truth)
- If STATE_LEDGER missing → Reconstruct from GitHub Project alone

### Step 4: Report Resume Status

After loading context, report:

```
CTO AGENT RESUMED

Time: [current timestamp]
Session ID: [unique identifier]

Current State from GitHub Project:
- Total active items: [N]
- In Progress: [M] items
- Awaiting Review: [K] items
- Blocked: [L] items
- Ready for Release: [J] items

Most Recent Work:
- Issue: #[number] - [title]
- Status: [column]
- Owner: [owner field value]
- Risk Tier: [risk tier]
- Next Action: [what to do next]

State Ledger Sync: [MATCHED/DISCREPANCY/MISSING]
If discrepancy: [explain and action taken]

Ready to continue from: [state]
```

---

## AGENT TASK UPDATE PROTOCOL

### When Working on a Task

All agents (Product, Code, Reliability, Knowledge, Advisor) must update the GitHub Project when:

1. **Task Started** → Move to "In Progress"
2. **PR Created** → Move to "In Review (PR Open)"
3. **PR Merged** → Move to "Done"
4. ** Approval Needed** → Move to "Waiting for Approval"
5. **Blocked** → Move to "Blocked" (add blocking reason)
6. **Ready for Release** → Move to "Ready for Release"

### Update Commands (via GitHub CLI)

```bash
# Move item to In Progress
gh project item-edit --owner ranjan-expatready --project "Autonomous Engineering OS — SDLC" --id <item-id> --field-id status --value "In Progress"

# Update custom fields
gh project item-edit --owner ranjan-expatready --project "Autonomous Engineering OS — SDLC" --id <item-id> --field-id Type --value "Feature"
gh project item-edit --owner ranjan-expatready --project "Autonomous Engineering OS — SDLC" --id <item-id> --field-id "Risk Tier" --value "T3"
gh project item-edit --owner ranjan-expatready --project "Autonomous Engineering OS — SDLC" --id <item-id> --field-id Owner --value "Code"
gh project item-edit --owner ranjan-expatready --project "Autonomous Engineering OS — SDLC" --id <item-id> --field-id Release --value "v1.0.0"
```

---

## DAILY STANDUP REPORT (Founder View)

Each day, the CTO Agent generates a standup report from the GitHub Project:

```
DAILY STANDUP REPORT
Date: [YYYY-MM-DD]

BOARD SNAPSHOT:
┌─────────────────┬───────┐
│ Column          │ Count │
├─────────────────┼───────┤
│ Backlog         │   12  │
│ Planned         │    3  │
│ In Progress     │    2  │
│ In Review       │    1  │
│ Waiting Approval│    1  │
│ Blocked         │    0  │
│ Ready for Release│   4  │
│ Done            │   25  │
└─────────────────┴───────┘

BY OWNER (Engineering View):
- Product: 3 items active
- Code: 2 items in review
- Reliability: 1 release prep
- Knowledge: 1 documentation task
- Advisor: 0 active

BY RELEASE (Release View):
- v1.2.0: 4 items ready, 2 in review
- v1.1.1: 1 item blocked (dependency)
- v1.1.0: 25 items done

AT-RISK ITEMS:
- #(issue): "Title" -Blocked since 2 days, awaiting human approval

FOUNDER ACTION NEEDED:
- Approve #[issue] - "Title" (in Waiting Approval since 1 day)
```

---

## STATE LEDGER INTEGRATION

The `STATE_LEDGER.json` file complements, but does not replace, the GitHub Project. Together they form the nervous system:

- **GitHub Project** = Visual SDLC (human-readable, interactive)
- **STATE_LEDGER** = AI Brain (machine-readable, persistent)

When updating STATE_LEDGER, include pointer to GitHub Project:

```json
{
  "last_execution": {
    "github_project_url": "https://github.com/ranjan-expatready/autonomous-engineering-os/projects/[project-number]",
    "active_issue_number": 123,
    "active_issue_title": "Implement user authentication"
  }
}
```

---

## AUTOMATED TRANSITIONS

Automation rules configure GitHub Project column movements:

### Default Automation
1. Issue created → Backlog
2. Issue assigned → Planned
3. PR linked → In Progress
4. PR opened → In Review
5. PR requires approval → Waiting for Approval
6. PR merged → Done
7. CI failed → Blocked

### Agent-Initiated Transitions
When automation rules don't apply (e.g., manual review decisions), agents must:
1. Comment on the issue explaining the transition
2. Move the item manually to the correct column
3. Update custom fields if needed
4. Record in STATE_LEDGER

---

## TROUBLESHOOTING RESUME

### Cannot Find GitHub Project

**Symptom**: Project URL not accessible

**Steps**:
1. Check if project was renamed
2. Verify you have access permissions
3. If project deleted, reconstruct from STATE_LEDGER and repository history

### Discrepancy Between GitHub Project and STATE_LEDGER

**Symptom**: Conflicting state information

**Resolution Protocol**:
1. GitHub Project wins (source of truth)
2. Update STATE_LEDGER to match GitHub Project
3. Add audit note in STATE_LEDGER explaining the correction
4. Alert human if discrepancy indicates data corruption

### Cannot Identify Active Work

**Symptom**: No items in In Progress, but system was in EXECUTING state

**Diagnosis**:
1. Check issue history for unexpected state changes
2. Review recent commits and PRs
3. Examine logs for errors or interruptions
4. Reconstruct from timeline of events

---

## COMPLIANCE CHECKLIST

On every resume:

```
[ ] 1. Accessed GitHub Project: "Autonomous Engineering OS — SDLC"
[ ] 2. Read all active items (In Progress, In Review, Waiting, Blocked, Ready for Release)
[ ] 3. Identified most recent work item
[ ] 4. Checked associated pull requests and discussions
[ ] 5. Cross-referenced with STATE_LEDGER.json
[ ] 6. Validated state consistency
[ ] 7. Reported resume status to human
[ ] 8. Documented any discrepancies found
[ ] 9. Confirmed ready to continue from current state
```

---

## VERSION HISTORY

- v1.0 (2024-01-23): Initial resume protocol, GitHub Project integration, state mapping
