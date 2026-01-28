# TEMPLATE — Bootstrap Guide

Version: v1.0
Owner: Antigravity (CTO)
Status: CANONICAL

## Purpose

TEMPLATE directory provides instructions for using AE-OS as a template.

## Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/ranjan-expatready/ae-os.git my-product
cd my-product
git remote remove origin
git remote add origin https://github.com/YOU/PRODUCT.git
```

### 2. Update Configuration

| Variable | Replace | Files |
|----------|---------|-------|
| ranjan-expatready | YOUR_USERNAME | All GitHub refs |
| ae-os | your-product | Repo refs |
| PVT_kw... | YOUR_PROJECT_ID | GITHUB_PROJECT_SDLC_ARTIFACT.md |

### 3. Configure Secrets (GitHub Settings)

| Secret | Purpose | Source |
|--------|---------|--------|
| DAILY_BRIEF_TOKEN | Daily brief | GitHub PAT |
| TRAE_API_KEY | Trae review | Trae service |

### 4. Create GitHub Project

1. Go to https://github.com/YOU
2. Click Projects → New project
3. Name: "My Product — SDLC"
4. Configure columns: Backlog, Plann

### 5. Update Core Files

| File | Update |
|------|--------|
| FOUNDATION/01_VISION.md | Company Vision |
| README.md | Project name |
| PRODUCT/ | Product requirements |
| FRAMEWORK/PROGRESS.md | Reset progress |

## Verification

```bash
# Check workflows
ls .github/workflows/

# Check secrets
gh secret list

# Test daily brief (dry run)
gh workflow run daily-brief.yml -f dry_run=true
```

---

**Last Updated**: 2026-01-26
**Status**: COMPLETE
