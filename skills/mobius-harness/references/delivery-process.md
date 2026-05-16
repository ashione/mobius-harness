# Delivery Process Reference

Use this reference for trigger rules, execution modes, phase gates, phase state, blockers, and change control.

## Trigger and Mode Standard

Use Mobius Harness when a user asks to implement, deliver, create a PR/MR, follow CI/CD, handle a feature end-to-end, or turn a plan/spec into working software. Do not force Mobius Harness for narrow analysis requests such as isolated SQL tuning, API review, bug classification, or commit message writing unless the task expands into delivery.

Choose one mode at the start:

- `Lightweight`: small, low-risk changes. Persisted artifacts are optional, but the final response must include the same facts.
- `Standard`: normal code delivery. Create `.delivery/runs/<run-id>/` and maintain the four core artifacts.
- `Strict`: high-risk, release, security, migration, multi-module, or user-requested audit work. Persist every phase/subphase state transition and all gate exceptions.

Generate `run-id` from the task name in kebab-case, for example `add-user-auth`. If it already exists, append a date or short sequence such as `add-user-auth-20260517` or `add-user-auth-2`. Avoid spaces, random long hashes, and non-descriptive ids.

## Delivery Process Standard

Mobius Harness follows these ordered phases. Each phase has an exit gate; do not move to the next phase until the gate is satisfied or the exception is explicitly recorded.

Large, risky, or blocked phases must be split into subphases. A subphase uses the same status record format as a phase, but with a narrower goal and checklist.

Recommended subphase naming:

- `requirements.discovery`
- `requirements.acceptance`
- `plan.repo-inspection`
- `plan.validation-strategy`
- `development.worktree`
- `implementation.backend`
- `implementation.frontend`
- `verification.local-checks`
- `verification.diff-review`
- `delivery.pr`
- `delivery.ci-followup`

| Phase | Required work | Exit gate |
|---|---|---|
| Requirements | Clarify goal, background, success criteria, scope, non-goals, risks, open questions, and user decisions. | Requirements are specific enough to implement and verify. |
| Plan | Inspect the repo, select specialist skills, define implementation steps, validation commands, acceptance criteria, rollback notes, and checkpoints. | Another agent could implement from the plan without choosing strategy. |
| Local Development | Follow `local-repo-development`, including worktree or branch selection and preservation of unrelated changes. | Worktree or branch and base ref are recorded. |
| Implementation | Make the scoped change and keep the diff coherent. | Changed files are intentional and mapped to acceptance criteria. |
| Verification | Run local checks, review the diff, and scan for sensitive information. | Validation outcomes and unresolved risks are recorded. |
| PR/MR + CI/CD | Commit, open PR/MR when applicable, and track CI/CD to pass, fail, or canceled. | PR/MR state and terminal CI/CD state are recorded or marked not applicable with reason. |
| Report | Summarize what was delivered and what remains. | Delivery report is complete and can be sent to the user or attached to PR/MR. |

## Phase State Standard

Every phase and subphase must record state with these sections:

- `Goal`: the concrete outcome this phase or subphase must achieve.
- `Checklist`: objective checks required to exit this phase or subphase.
- `Todo List`: unfinished actions, preferably with status such as `todo`, `doing`, `blocked`, or `done`.
- `Failure List`: failed commands, blocked checks, rejected assumptions, CI/CD failures, defects found during review, or unresolved risks.
- `Change List`: decisions made, files changed, requirement changes, scope changes, validation changes, or follow-up changes.

Use these status values:

- Phase status: `draft`, `active`, `blocked`, `complete`, `deferred`.
- Todo status: `todo`, `doing`, `blocked`, `done`, `deferred`.
- Failure status: `open`, `investigating`, `fixed`, `accepted`, `deferred`.

When transitioning phases:

- mark checklist items as complete or explicitly deferred,
- move unfinished Todo List items into the next phase,
- carry unresolved Failure List items forward until resolved or accepted,
- record scope or implementation changes in Change List,
- keep enough evidence for another agent to resume,
- do not mark a phase or subphase `complete` without evidence or an explicit accepted exception.

Recommended phase/subphase block:

```md
## <Phase or Subphase Name>

Status: draft | active | blocked | complete | deferred
Phase: <phase-name>
Updated: <timestamp or phase marker>
Evidence: <commands, files, links, PR/MR, CI/CD, or reason unavailable>

### Goal

### Checklist

- [ ] ...

### Todo List

| Item | Status | Owner | Evidence |
|---|---|---|---|

### Failure List

| Failure | Impact | Root Cause | Resolution | Status |
|---|---|---|---|---|

### Change List

| Change | Reason | Files/Links | Approval |
|---|---|---|---|
```

## Blocker and Change Control

When blocked:

1. Self-check discoverable causes first.
2. Record the issue in Failure List.
3. Attempt one minimal recovery action when safe.
4. If still blocked, ask the user for the specific decision needed.
5. Record the user decision or accepted risk in Change List.

Record a Change List item for any:

- scope change,
- acceptance criteria change,
- validation strategy change,
- branch or worktree change,
- skipped gate or gate exception,
- accepted failing check,
- CI/CD failure accepted as out of scope,
- release or rollback change.
