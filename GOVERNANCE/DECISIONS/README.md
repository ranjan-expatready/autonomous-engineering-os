# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records (ADRs) for the
autonomous-engineering-os project. ADRs document significant technical and
governance decisions with their context, rationale, and consequences.

## Format

Each ADR follows the template in `TEMPLATE.md`. Files are named:

```
ADR-NNNN-short-title.md
```

Where `NNNN` is a zero-padded sequential number.

## Lifecycle

| Status | Meaning |
|--------|---------|
| `proposed` | Under discussion, not yet accepted |
| `accepted` | Decision made and in effect |
| `superseded` | Replaced by a newer ADR (link to successor) |
| `deprecated` | No longer relevant |

## Conventions

1. ADRs are **immutable once accepted**. To change a decision, create a new
   ADR that supersedes the old one.
2. Every ADR must have a **status**, **date**, and **decision** section.
3. ADRs should be **concise** (1-2 pages max).
4. Link to related COMMS, incidents, or governance artifacts where relevant.
5. All ADRs go through PR review with `machine-board` governance gate.

## Index

| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [ADR-0001](ADR-0001-branch-protection-enforcement.md) | Branch protection enforcement model | accepted | 2026-02-22 |
