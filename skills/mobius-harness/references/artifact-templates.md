# Artifact Templates Reference

Use these templates as the default shape for persisted Delivery Episode Package files. Add subphase blocks when the phase is large, risky, blocked, or needs an auditable handoff.

## requirements.md template

```md
# Requirements

Status: draft | active | blocked | complete | deferred
Phase: requirements
Updated: <timestamp or phase marker>
Evidence: <user request, repo files, issue links, or reason unavailable>

## Phase State

### Goal

### Checklist

- [ ] Goal is explicit.
- [ ] Success criteria are verifiable.
- [ ] Scope and non-goals are explicit.
- [ ] High-impact unknowns are resolved or recorded.

### Todo List

| Item | Status | Owner | Evidence |
|---|---|---|---|

### Failure List

| Failure | Impact | Root Cause | Resolution | Status |
|---|---|---|---|---|

### Change List

| Change | Reason | Files/Links | Approval |
|---|---|---|---|

## Goal

## Background

## Success Criteria

## Scope

## Non-Goals

## Risks

## Open Questions

## User Decisions
```

## plan.md template

```md
# Plan

Status: draft | active | blocked | complete | deferred
Phase: plan
Updated: <timestamp or phase marker>
Evidence: <repo inspection commands, files, issue links, or reason unavailable>

## Phase State

### Goal

### Checklist

- [ ] Affected areas are identified.
- [ ] Specialist skills are selected or rejected with reason.
- [ ] Implementation steps are ordered.
- [ ] Validation strategy covers success criteria.
- [ ] Rollback or mitigation notes are recorded.

### Todo List

| Item | Status | Owner | Evidence |
|---|---|---|---|

### Failure List

| Failure | Impact | Root Cause | Resolution | Status |
|---|---|---|---|---|

### Change List

| Change | Reason | Files/Links | Approval |
|---|---|---|---|

## Repo Findings

## Specialist Skills

## Implementation Steps

## Validation Strategy

## Acceptance Criteria

## Rollback Notes

## Checkpoints
```

## verification.md template

```md
# Verification

Status: draft | active | blocked | complete | deferred
Phase: verification
Updated: <timestamp or phase marker>
Evidence: <commands, diff, scanner output summary, PR/MR links, CI/CD links, or reason unavailable>

## Phase State

### Goal

### Checklist

- [ ] Local validation commands are run or marked unavailable with reason.
- [ ] Diff review is complete.
- [ ] Sensitive information scan is complete.
- [ ] PR/MR state is recorded or marked not applicable.
- [ ] CI/CD terminal state is recorded or marked not applicable.

### Todo List

| Item | Status | Owner | Evidence |
|---|---|---|---|

### Failure List

| Failure | Impact | Root Cause | Resolution | Status |
|---|---|---|---|---|

### Change List

| Change | Reason | Files/Links | Approval |
|---|---|---|---|

## Local Commands

| Command | Result | Evidence |
|---|---|---|

## Command Results

## Diff Review

### Requirements Compliance

### Implementation Quality

### Test Adequacy

### Security and Sensitive Information

## Sensitive Information Scan

## PR/MR

## CI/CD

## Unresolved Risks
```

## delivery-report.md template

```md
# Delivery Report

Status: draft | active | blocked | complete | deferred
Phase: report
Updated: <timestamp or phase marker>
Evidence: <artifact links, commands, PR/MR links, CI/CD links, or reason unavailable>

## Phase State

### Goal

### Checklist

- [ ] Requirements result is summarized.
- [ ] Implementation and changed files are summarized.
- [ ] Validation, review, and sensitive scan are summarized.
- [ ] PR/MR and CI/CD state are summarized.
- [ ] Risks and follow-ups are explicit.

### Todo List

| Item | Status | Owner | Evidence |
|---|---|---|---|

### Failure List

| Failure | Impact | Root Cause | Resolution | Status |
|---|---|---|---|---|

### Change List

| Change | Reason | Files/Links | Approval |
|---|---|---|---|

## Summary

## Requirements Result

## Implementation Summary

## Changed Files

## Validation Summary

## PR/MR and CI/CD

## Risks and Follow-ups

## Release Notes
```
