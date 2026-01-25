# TRAE_REVIEW Artifact Template

Copy this template for each new Trae review artifact.

## Artifact ID Format

`TRAE-{YYYYMMDD}-{PR-NUMBER}`

Example: `TRAE-20260125-042.yml`

---

## TRAE_REVIEW Artifact Template

```yaml
ARTIFACT_TYPE: TRAE_REVIEW
artifact_id: "TRAE-{timestamp}-{pr-number}"
created_at: "YYYY-MM-DD HH:MM UTC"
created_by: "Factory (based on Trae verdict)"

# REVIEW CONTEXT
pr_number: {pr-number}
pr_url: "https://github.com/owner/repo/pull/{pr-number}"

# TRAE'S VERDICT
verdict: "APPROVE" | "REJECT" | "REQUEST_CHANGES"
signature: "trae-external-reviewer"

# REVIEW SCOPE
review_scope:
  - "List of files/directories reviewed by Trae"

# FINDINGS
security_findings:
  - "Finding 1 or empty list"
  - "Finding 2 or empty list"
policy_violations:
  - "Violation 1 or empty list"
  - "Violation 2 or empty list"

# RECOMMENDATIONS
recommendations: |
  Trae's recommendations and feedback.
  Include guidance for fixes if requested changes.

# METADATA
review_timestamp: "YYYY-MM-DD HH:MM UTC"
expiry_days: 7

# ARTIFACT LINKS
links:
  github_pr: "https://github.com/owner/repo/pull/{pr-number}"
  artifact_file: "COCKPIT/artifacts/TRAE_REVIEW/TRAE-{timestamp}-{pr-number}.yml"
```

---

## Example: Approved PR

```yaml
ARTIFACT_TYPE: TRAE_REVIEW
artifact_id: "TRAE-20260125-042"
created_at: "2026-01-25 10:30 UTC"
created_by: "Factory (based on Trae verdict)"

pr_number: 42
pr_url: "https://github.com/owner/repo/pull/42"

verdict: "APPROVE"
signature: "trae-external-reviewer"

review_scope:
  - "GOVERNANCE/GUARDRAILS.md"
  - "scripts/governance_validator.py"

security_findings: []
policy_violations: []

recommendations: "No security or policy issues found. Trae review: APPROVE - Change is compliant with governance policies and safe to merge."

review_timestamp: "2026-01-25 10:30 UTC"
expiry_days: 7

links:
  github_pr: "https://github.com/owner/repo/pull/42"
  artifact_file: "COCKPIT/artifacts/TRAE_REVIEW/TRAE-20260125-042.yml"
```

---

## Example: Rejected PR (Security Issues)

```yaml
ARTIFACT_TYPE: TRAE_REVIEW
artifact_id: "TRAE-20260125-043"
created_at: "2026-01-25 11:00 UTC"
created_by: "Factory (based on Trae verdict)"

pr_number: 43
pr_url: "https://github.com/owner/repo/pull/43"

verdict: "REJECT"
signature: "trae-external-reviewer"

review_scope:
  - "APP/auth.py"

security_findings:
  - "Hardcoded password in APP/auth.py line 42: password = 'secret123'"
  - "SQL injection risk: User input directly interpolated into SQL query on line 58"

policy_violations: []

recommendations: |
  CRITICAL: Fix security issues before merging:
  1. Remove hardcoded password, use environment variables or secrets manager
  2. Use parameterized queries to prevent SQL injection
  3. Re-request Trae review after fixes

review_timestamp: "2026-01-25 11:00 UTC"
expiry_days: 7

links:
  github_pr: "https://github.com/owner/repo/pull/43"
  artifact_file: "COCKPIT/artifacts/TRAE_REVIEW/TRAE-20260125-043.yml"
```

---

## Example: Request Changes (Minor Issues)

```yaml
ARTIFACT_TYPE: TRAE_REVIEW
artifact_id: "TRAE-20260125-044"
created_at: "2026-01-25 11:30 UTC"
created_by: "Factory (based on Trae verdict)"

pr_number: 44
pr_url: "https://github.com/owner/repo/pull/44"

verdict: "REQUEST_CHANGES"
signature: "trae-external-reviewer"

review_scope:
  - "GOVERNANCE/COST_POLICY.md"
  - "scripts/monitoring.py"

security_findings: []
policy_violations:
  - "Cost threshold modified without justification. Increase from $100 to $500 requires documented business case."

recommendations: |
  MINOR: Address policy violation:
  1. Add justification for cost threshold change in PR description or separate documentation
  2. Consider if threshold change is necessary for business needs
  3. Re-request Trae review after addressing

review_timestamp: "2026-01-25 11:30 UTC"
expiry_days: 7

links:
  github_pr: "https://github.com/owner/repo/pull/44"
  artifact_file: "COCKPIT/artifacts/TRAE_REVIEW/TRAE-20260125-044.yml"
```

---

## Naming Convention

- Directory: `COCKPIT/artifacts/TRAE_REVIEW/`
- Filename: `TRAE-{YYYYMMDD}-{pr-number}.yml`
- Example: `TRAE-20260125-042.yml` (PR #42 on Jan 25, 2026)

---

## Artifact Lifecycle

1. **Factory invokes Trae** when PR is opened (T1-T4 change)
2. **Trae returns verdict** (APPROVE/REJECT/REQUEST_CHANGES)
3. **Factory creates artifact** in `COCKPIT/artifacts/TRAE_REVIEW/TRAE-{YYYYMMDD}-{pr-number}.yml`
4. **Machine Board validates** artifact exists and verdict==APPROVE
5. **PR updated (new commit)** → Factory updates artifact with new Trae verdict
6. **Artifact expires** after 7 days (requires fresh review)

---

## Emergency Override

In emergencies where Trae is unavailable:

```yaml
# Add to artifact:

verdict: "EMERGENCY_OVERRIDE"
emergency_reason: "Trae service unavailable, critical security fix needed"
authorized_by: "@username"
post_merge_review_required: true
```

Machine Board will allow merge with emergency_override flag but will flag for post-merge review.
