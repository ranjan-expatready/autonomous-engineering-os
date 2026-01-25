# SDLC Board Verification Proof

## Purpose

This document provides verification proof instructions for the GitHub Projects v2 SDLC Board. Due to API token scope limitations (missing `project` and `read:project` scopes), the actual project must be created manually. Once created, follow this verification protocol.

**Created**: 2026-01-25
**Status**: 🔄 AWAITING MANUAL PROJECT CREATION
**Reference**: GITHUB_PROJECT_SDLC_ARTIFACT.md

---

## PRE-CREATION: Token Scope Refresh (Required)

Before creating the GitHub Project v2, you must refresh your GitHub token with the required scopes:

### Option A: Via Web Browser (Recommended)

1. Navigate to: https://github.com/settings/tokens
2. Find your current "Factory" token
3. Click "Edit"
4. Add scopes:
   - ✓ project
   - ✓ read:project
5. Save the token
6. Re-authenticate with `gh auth login` using the updated token

### Option B: Via CLI (Interactive)

```bash
# Run this command and follow the web browser authentication flow
gh auth refresh --hostname github.com -s project -s read:project

# You'll get a one-time code (e.g., 5B2F-10A0)
# Open the URL in your web browser and complete authentication
```

### Verify Token Refresh

```bash
gh auth status
# Should show: Token scopes: 'gist', 'project', 'read:org', 'read:project', 'repo', 'workflow'
```

---

## STEP 1: Create GitHub Project v2

### Method A: Using GitHub Web UI (Recommended)

1. Navigate to: https://github.com/ranjan-expatready/autonomous-engineering-os
2. Click on the "Projects" tab
3. Click "New Project"
4. Select "Project V2" (not "Project V1")
5. Set:
   - **Owner**: `ranjan-expatready/autonomous-engineering-os`
   - **Name**: `Autonomous Engineering OS — SDLC`
   - **Description**: Live SDLC state machine for the Autonomous Engineering OS framework
   - **Template**: None
   - **Visibility**: Private (recommended)
6. Click "Create Project"
7. Note the project URL (e.g., `https://github.com/ranjan-expatready/autonomous-engineering-os/projects/1`)

### Method B: Using GitHub CLI (After Token Refresh)

```bash
# Get repository ID (you should have this: R_kgDOQ-HCzg)
gh api graphql --header 'GraphQL-Features: projects_next_graphql' -f query='
mutation($input: CreateProjectV2Input!) {
  createProjectV2(input: $input) {
    projectV2 {
      id
      number
      url
      title
    }
  }
}
' -f variables='{"input":{"ownerId":"R_kgDOQ-HCzg","title":"Autonomous Engineering OS — SDLC","description":"Live SDLC state machine for the Autonomous Engineering OS framework"}}'
```

**Expected Output**:
```json
{
  "data": {
    "createProjectV2": {
      "projectV2": {
        "id": "PVT_kwDOGZ...",
        "number": 1,
        "url": "https://github.com/ranjan-expatready/autonomous-engineering-os/projects/1",
        "title": "Autonomous Engineering OS — SDLC"
      }
    }
  }
}
```

---

## STEP 2: Configure Custom Fields

After creating the project, navigate to project settings and add these custom fields:

### Field 1: Type (Single Select)

1. Click ⋮ menu (top right) → Settings → Fields
2. Click "New field"
3. Configure:
   - **Name**: Type
   - **Type**: Single select
   - **Options**:
     - Epic (Purple)
     - Feature (Green)
     - Bug (Red)
     - Incident (Red - darker)
     - Tech Debt (Orange)

### Field 2: Risk Tier (Single Select)

1. Click "New field"
2. Configure:
   - **Name**: Risk Tier
   - **Type**: Single select
   - **Options**:
     - T0 (Red) - Critical, production-affecting
     - T1 (Orange) - High risk, requires explicit approval
     - T2 (Yellow) - Medium risk, breaking changes
     - T3 (Green) - Low risk, autonomous execution

### Field 3: Owner (Single Select)

1. Click "New field"
2. Configure:
   - **Name**: Owner
   - **Type**: Single select
   - **Options**:
     - Product (Blue)
     - Code (Green)
     - Reliability (Orange)
     - Knowledge (Purple)
     - Advisor (Gray)

### Field 4: Release (Text)

1. Click "New field"
2. Configure:
   - **Name**: Release
   - **Type**: Text
   - **Description**: Release version (e.g., v1.2.0, TBD, Backlog)

---

## STEP 3: Configure Kanban Columns (Statuses)

Configure the project board with these statuses in order:

1. **Backlog** (Default for new issues)
2. **Planned**
3. **In Progress**
4. **In Review (PR Open)**
5. **Waiting for Approval**
6. **Blocked**
7. **Ready for Release**
8. **Done**

**Setup**:
1. Click "⚙️ View settings" → "View configuration"
2. Under "Group by", select "Status"
3. Reorder columns to match the order above (drag-and-drop)
4. Ensure custom fields are visible:
   - Click "⚙️ View settings" → "Show fields"
   - Enable: Type, Risk Tier, Owner, Release

---

## STEP 4: Configure Automation Rules

Navigate to project settings → Automation and add these rules:

| Rule | Trigger | Action |
|------|---------|--------|
| 1 | Issue is created | Set status to "Backlog" |
| 2 | Issue is assigned | Set status to "Planned" |
| 3 | Pull request is opened | Set status to "In Progress" |
| 4 | Pull request is in review | Set status to "In Review (PR Open)" |
| 5 | Pull request requires review | Set status to "Waiting for Approval" |
| 6 | Pull request is merged | Set status to "Done" |
| 7 | CI workflow fails | Set status to "Blocked" |

**Setup Details**:
- **Rule 1**: Trigger=Issue, Condition=Created, Action=Set status=Backlog
- **Rule 2**: Trigger=Issue, Condition=Assignee assigned, Action=Set status=Planned
- **Rule 3**: Trigger=Pull request, Condition=Opened, Action=Find linked issue → Set status=In Progress
- **Rule 4**: Trigger=Pull request, Condition=Status=In review, Action=Set linked issue status=In Review (PR Open)
- **Rule 5**: Trigger=Pull request, Condition=Review required, Action=Set linked issue status=Waiting for Approval
- **Rule 6**: Trigger=Pull request, Condition=Merged, Action=Set linked issue status=Done
- **Rule 7**: Trigger=Workflow run, Condition=Status=Failure, Action=Find linked issue → Set status=Blocked

---

## STEP 5: Configure Views

Create three saved views:

### View 1: Founder View

1. Click "⚙️ View settings" → "Save view"
2. Configure:
   - **Name**: Founder View
   - **Group by**: Status
   - **Sort by**: Risk Tier (descending: T0, T1, T2, T3)
   - **Layout**: Board
   - **Fields visible**: Type, Risk Tier, Owner, Release

### View 2: Engineering View

1. Click "⚙️ View settings" → "Save view"
2. Configure:
   - **Name**: Engineering View
   - **Group by**: Owner
   - **Sort by**: Status (in Kanban order)
   - **Layout**: Board
   - **Fields visible**: Type, Risk Tier, Release

### View 3: Release View

1. Click "⚙️ View settings" → "Save view"
2. Configure:
   - **Name**: Release View
   - **Group by**: Release
   - **Sort by**: Status
   - **Layout**: Table
   - **Fields visible**: Type, Risk Tier, Owner, Status

---

## STEP 6: Verification Test

Create a test issue to verify the setup:

### Test Issue Creation

```bash
# Create a test issue
gh issue create \
  --repo ranjan-expatready/autonomous-engineering-os \
  --title "TEST: SDLC Board Automation Verification" \
  --body "This is a test issue to verify SDLC Board automation is working correctly." \
  --label "test"
```

### Expected Results

1. **Issue Auto-Add to Project**:
   - [ ] Test issue appears in the project immediately after creation
   - [ ] Issue status is set to "Backlog" (default)
   - [ ] Issue is visible in "Founder View"

2. **Assignment Automation**:
   - Add assignee to test issue
   - [ ] Issue status changes to "Planned"

3. **PR Link Automation**:
   - Create a test PR from a new branch:
   ```bash
   git checkout -b test/sdlc-board-verification
   # Make a small change (e.g., edit this file)
   git commit -am "test: SDLC board verification"
   git push origin test/sdlc-board-verification
   gh pr create --title "TEST: SDLC Board Verification" --body "Closes #<test-issue-number>"
   ```
   - [ ] Linked issue status changes to "In Progress"
   - [ ] Issue is visible in "In Progress" column

4. **Custom Fields**:
   - [ ] Set Type field to "Feature"
   - [ ] Set Risk Tier field to "T3"
   - [ ] Set Owner field to "Code"
   - [ ] Set Release field to "v1.0.0"

5. **View Verification**:
   - [ ] "Founder View" shows issue grouped by status, sorted by risk tier
   - [ ] "Engineering View" shows issue grouped by owner (Code)
   - [ ] "Release View" shows issue grouped by release (v1.0.0)

### Test Cleanup

After verification:

```bash
# Close test issue and PR
gh issue close <test-issue-number> --comment "Verification complete. Test artifacts cleaned up."
gh pr close <test-pr-number> --comment "Verification complete. Test artifacts cleaned up."

# Delete test branch
git branch -D test/sdlc-board-verification
git push origin --delete test/sdlc-board-verification
```

---

## STEP 7: Update Evidence Index

After successful verification, update `FRAMEWORK/EVIDENCE_INDEX.md`:

```markdown
### GITHUB PROJECT v2: SDLC BOARD

**Status**: ✅ OPERATIONAL
**Created**: [DATE]
**Project URL**: https://github.com/ranjan-expatready/autonomous-engineering-os/projects/[NUMBER]

**Evidence**:
- [ ] Project created successfully
- [ ] 4 custom fields configured (Type, Risk Tier, Owner, Release)
- [ ] 8 Kanban columns configured (Backlog → Done)
- [ ] 7 automation rules active
- [ ] 3 views created (Founder, Engineering, Release)
- [ ] Test issue verified automation working

**Test Issue**: #[TEST-ISSUE-NUMBER]
**Test PR**: #[TEST-PR-NUMBER]
```

Also update `FRAMEWORK/PROGRESS.md`:

```markdown
| SDLC Board Documentation | ✅ COMPLETE | 100% | [DATE] |
```

And update `STATE/STATUS_LEDGER.md`:

```markdown
| [DATE] | SDLC Board Created and Wired | Project URL + verification test | ✅ |
```

---

## STEP 8: Integration Confirmation

Verify that SDLC Board is integrated into the resume protocol:

1. Read `RUNBOOKS/resume-protocol.md`
2. Verify STEP 4: "Check SDLC Board Status" exists
3. Verify resume success criteria includes SDLC Board check
4. Confirm resume protocol references the project URL

---

## TROUBLESHOOTING

### Issue: Automation Not Working

**Symptoms**: Issues not moving automatically

**Solutions**:
1. Verify GitHub token has `project` and `read:project` scopes
2. Check automation rules are enabled in project settings (⋮ → Automation)
3. Review rule triggers and conditions for syntax errors
4. Check GitHub Actions logs for automation errors

### Issue: Custom Fields Not Showing

**Symptoms**: Type, Risk Tier, Owner, Release not visible on items

**Solutions**:
1. Click "⚙️ View settings" → "Show fields"
2. Enable custom fields checkboxes
3. Refresh project view

### Issue: Items Not Syncing from Repository

**Symptoms**: New issues don't appear in project

**Solutions**:
1. Add issues manually if automation doesn't work: `gh project item-add`
2. Check automation rule for "Issue created" is active
3. Verify repository is linked to project

---

## ROLLBACK PLAN

If verification fails:

1. **Delete Test Evidence**:
   ```bash
   gh issue close <test-issue-number> --comment "Rolling back: test cleanup"
   gh pr close <test-pr-number> --comment "Rolling back: test cleanup"
   git push origin --delete test/sdlc-board-verification
   ```

2. **Delete Project** (if needed):
   - Navigate to project → Settings
   - Scroll to bottom
   - Click "Delete project"

3. **Update Documentation**:
   - Update `FRAMEWORK/EVIDENCE_INDEX.md` to reflect rollback
   - Update `FRAMEWORK/PROGRESS.md` to mark as BLOCKED or DEFERRED

---

## SUCCESS CRITERIA

SDLC Board verification is successful when:

- [ ] Project created at correct URL
- [ ] All 4 custom fields configured
- [ ] All 8 Kanban columns present
- [ ] All 7 automation rules active
- [ ] All 3 views created and working
- [ ] Test issue auto-adds to project
- [ ] Test issue moves through statuses correctly
- [ ] Custom fields can be set and viewed
- [ ] All views display correctly
- [ ] Documentation updated with project URL

---

**Last Updated**: 2026-01-25
**Status**: 🔄 AWAITING MANUAL PROJECT CREATION AND VERIFICATION
