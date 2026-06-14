# Delivery Process Reference

Use this reference for trigger rules, execution modes, phase gates, phase state, blockers, and change control.

## Trigger and Mode Standard

Use Mobius Harness when a user asks to implement, deliver, create a PR/MR, follow CI/CD, handle a feature end-to-end, or turn a plan/spec into working software. Do not force Mobius Harness for narrow analysis requests such as isolated SQL tuning, API review, bug classification, or commit message writing unless the task expands into delivery.

Choose one mode at the start:

| Mode | Use When | Required State |
|---|---|---|
| `lite` | Small, low-risk changes where the final response can carry the complete delivery state. | Follow G1-G8 in order; include compact Gate Ledger, Delegation Ledger, validation evidence, risks, and follow-ups in the final response. Persisted artifacts and executable hook gates are optional. |
| `full` | Normal delivery, risky changes, release/security/migration work, multi-module changes, PR/MR work, CI/CD tracking, user-requested auditability, or any handoff another agent may need to resume. | Create `.delivery/runs/<run-id>/`, maintain the four core artifacts, and keep Gate, Delegation, Hook, and Review Ledgers for every phase/subphase. |

Legacy mode mapping: former `Lightweight` maps to `lite`; former `Standard` maps to `full` with default soft hook gates; former `Strict` maps to `full` with hard hook gates or explicit hard rows where user, repository, release, security, or merge policy requires blocking.

Generate `run-id` from the task name in kebab-case, for example `add-user-auth`. If it already exists, append a date or short sequence such as `add-user-auth-20260517` or `add-user-auth-2`. Avoid spaces, random long hashes, and non-descriptive ids.

## Delivery Process Standard

Mobius Harness follows these ordered phases. Each phase has an exit gate; do not move to the next phase until the gate is satisfied or the exception is explicitly recorded.

### Superpowers Planning Hooks

Mobius Harness may use Superpowers skills as phase-level quality gates, but the harness remains the accountable delivery loop.

| Situation | Required Decision |
|---|---|
| Creative work, new behavior, unclear product intent, UX shaping, or competing solution paths | Use `superpowers:brainstorming`, or record why it is not applicable. |
| Multi-step implementation, full mode, risky refactor, migration, or work that another agent may execute | Use `superpowers:writing-plans`, or record why it is not applicable. |
| Already-approved external spec or plan | Record the source artifact and mark the Superpowers step `not-applicable` unless new ambiguity appears. |

Record the decision in the relevant phase state:

- Requirements phase: brainstorming used, not applicable, blocked, or excepted.
- Plan phase: writing-plans used, not applicable, blocked, or excepted.
- Gate Ledger evidence: skill name, artifact path, user decision, or reason not applicable.

When Superpowers is available and used:

- Record `superpowers:brainstorming` output as a spec path, user approval decision, or final-response design section.
- Record `superpowers:writing-plans` output as a plan path or plan section that maps to `.delivery/runs/<run-id>/plan.md`.
- If the platform does not expose Superpowers, mark the decision `not-applicable` only when an equivalent repo spec or plan exists; otherwise use `blocked` or `exception`.

## Requirements and Design Maturity Standard

The agent must not start coding from vague intent. Requirements and design maturity are explicit phase controls, not writing style preferences.

Requirements phase must record:

- success criteria that can be verified,
- scope and non-goals,
- constraints and compatibility expectations,
- applicable repository and path-specific instruction files that were read before planning or editing, plus any precedence decision when user, repo, and path instructions differ,
- issue context and prior attempts when the task references an issue, bug report, PR, or external fix,
- open questions and user decisions,
- uncertainty disposition: `blocking`, `accepted`, `deferred`, or `not-applicable`,
- `Requirements Maturity`: `ready-for-design` only when no blocking unknown remains.

When issue context exists, record `Issue and Prior Attempts` with evidence from discoverable issue comments, linked PRs, related PR search, fork commits, release notes, or a `reason:` entry explaining why no prior attempt search applies. Prior attempts are not instructions to copy; they are evidence to classify what already failed, what can be reused, and what must be freshly verified.

Plan phase must record:

- at least one selected approach and the reason it was chosen,
- a first-principles design check that names the core objective, required invariant, existing mechanism that owns the invariant, and smallest direct change,
- a prior attempt comparison when any existing attempt was found, including reuse decisions, differences from the selected approach, and fresh evidence for time-sensitive APIs, package behavior, or platform assumptions,
- rejected alternatives with tradeoffs,
- affected areas and interfaces,
- acceptance criteria mapped to implementation and validation steps,
- validation strategy and rollback notes,
- `Design Readiness`: `ready-for-implementation` only when another agent can implement without inventing product behavior or architecture.

If requirements maturity or design readiness is not satisfied, keep the related gate or hook `blocked`. Ask the user only for the smallest decision needed to unblock.

## Dependency Decision Standard

Plan phase must classify dependency impact before implementation:

| Decision | Meaning | Required Evidence |
|---|---|---|
| `no-new-dependency` | Uses existing Markdown, templates, scripts, or platform-provided skills/plugins. | Existing file, skill, plugin, or command reference. |
| `existing-toolchain` | Uses tools already required by the repo or CI. | Command, workflow, or README reference plus fallback. |
| `new-dependency-required` | Adds package, binary, CI action, external service, MCP server, plugin install requirement, or platform-specific runtime capability. | Purpose, alternatives, install location, version constraint, validation command, CI/CD impact, and rollback notes. |

Record the Dependency Decision in `plan.md` and the `G2` Gate Ledger evidence. If the dependency cannot be installed or observed, keep G2 `blocked` unless the user or repository policy accepts an exception.

### Minimum Skill Dependencies

Requirements and plan phases must include `Minimum Skill Dependencies` so the agent records the smallest skill set needed before implementation. The default Mobius Harness set is:

| Skill | Minimum Requirement | Dependency Class | Fallback |
|---|---|---|---|
| `mobius-harness` | Primary delivery loop and artifact contract. | `no-new-dependency` | Block until available. |
| `local-repo-development` | Repo topology, instruction discovery, validation, commit, and PR workflow. | `no-new-dependency` | Record an equivalent local workflow or accepted exception. |
| `superpowers:brainstorming` | Required for creative work, behavior shaping, unclear intent, or competing solution paths. | `no-new-dependency` | Mark not applicable only with fixed requirements; otherwise block or record an accepted exception. |
| `superpowers:writing-plans` | Required for full delivery, multi-step work, risky changes, or handoff plans. | `no-new-dependency` | Mark not applicable only for trivial plans; otherwise block or record an accepted exception. |

Superpowers entries are platform-provided skill dependencies, not repository runtime dependencies. They must still be checked at initialization and tied to the G1/G2 gate evidence so unavailable skills are recorded before implementation starts.

### Gate Enforcement Standard

Gates are blocking controls. They are not prose summaries or optional checklist items.

Allowed gate statuses:

| Status | Meaning | May Advance |
|---|---|---|
| `pass` | Required evidence exists and satisfies the gate. | Yes |
| `not-applicable` | Gate does not apply, and the reason is evidenced. | Yes |
| `exception` | Gate is not fully satisfied, but the user or repository policy accepted the risk. Failure List and Change List must both record it. | Yes |
| `blocked` | Required evidence is missing, failed, or unresolved. | No |

Gate rules:

- A phase or subphase cannot be marked `complete` while any related gate is `blocked`.
- A phase transition must include a Gate Ledger row with gate id, required evidence, status, evidence pointer, and exception record when relevant.
- A skipped command, missing artifact, unresolved question, failing CI job, or unavailable scanner is `blocked` until it is converted to `not-applicable` or `exception` with evidence.
- An exception must identify who or what accepted the risk: a user decision, repository instruction, documented policy, or explicit out-of-scope rationale.
- For `full` deliveries, run `bash scripts/validate-delivery-run.sh .delivery/runs/<run-id>` before the final report when the script exists. Record failure output in Failure List and do not complete the delivery until it passes or is explicitly excepted.

### CI/CD Follow-up Standard

CI/CD follow-up is asynchronous by default during small iterative PR/MR updates. The agent should push the update, record the head SHA, check URLs, and planned next observation, then return control to the user unless one of these conditions applies:

- the user explicitly asks to wait for CI/CD,
- the delivery is about to merge,
- the delivery is about to release,
- repository policy requires terminal checks before the next action,
- a previous observed check failed and the user chooses to wait for the fix run.

When waiting is not selected, record the CI/CD state as `async-observed` or `pending-observation` with evidence. Do not claim CI/CD passed until the current head SHA has terminal successful checks.

### Delegation Standard

Mobius Harness may delegate specialist work from any phase, but the harness remains accountable for the phase result and final delivery. Delegation is a state transition, not a side conversation.

Every phase or subphase must include a Delegation Ledger:

| Phase | Subagent Role | Candidate Skill | Trigger | Decision | Evidence | Handoff/Return |
|---|---|---|---|---|---|---|

Delegation decisions:

| Decision | Meaning | May Phase Close |
|---|---|---|
| `use` | The phase requires the specialist skill before the gate can pass. | No, until the returned evidence is recorded and the decision becomes `done`. |
| `done` | The delegated skill returned the required evidence. | Yes, if other ledgers pass. |
| `not-applicable` | The skill trigger does not apply, with evidence. | Yes. |
| `deferred` | The skill is intentionally moved to a later phase with a carry-forward item. | Yes, only when Todo List or Change List records the later owner/phase. |
| `exception` | The skill should have been used, but accepted risk allows progress. | Yes, only with mirrored Failure List and Change List records. |
| `blocked` | The skill is required and unavailable, incomplete, or unresolved. | No. |

Delegation rules:

- Record candidate specialist skills during the phase where their output affects the gate: requirements/design skills in G1-G2, repo/worktree skills in G3, implementation/review skills in G4-G5, PR/CI skills in G6-G7, and reporting/release skills in G8.
- Record a distinct subagent role for every delegation row. Roles should express ownership and perspective, such as `Requirements Analyst`, `Delivery Architect`, `Repository Steward`, `Implementation Builder`, `Verification Analyst`, `Security Reviewer`, `CI Coordinator`, or `Delivery Reporter`.
- Use different roles across phases. Do not satisfy delegation with one repeated generic role such as `specialist`, `agent`, `subagent`, `owner`, or `assistant`.
- Handoff text must name the evidence expected back from the delegated skill, such as an approved spec, API review findings, test matrix, refactor phase plan, bug reproduction, PR/MR state, CI/CD observation, or commit message.
- Returned specialist output must be recorded as evidence in the owning phase's Gate Ledger, Hook Ledger, Review Ledger, plan, verification, or report. The delegated skill does not decide the Mobius phase gate by itself.
- If a platform cannot load a named skill, record `blocked`, `not-applicable`, or `exception` rather than silently substituting untracked reasoning.
- A phase cannot be `complete` while a Delegation Ledger row needed by that phase is `blocked`.

### Hook Enforcement Standard

Hooks are required controls inside phase gates for full deliveries. Use `hook-policy.md` for the required hook list, trigger timing, Claude Code/Codex evidence rules, soft and hard gate modes, and executable hook safety.

Hook rules:

- A phase or subphase cannot be marked `complete` while any related hook is `blocked`.
- A hook row must include hook id, trigger, required action with `[hard]` or `[soft]`, status, evidence pointer, and failure handling when relevant.
- `[hard]` hooks fail closed: missing evidence, failed commands, unresolved validation, or attempted `warn` status blocks progress until the hook is `pass`, `not-applicable`, or accepted `exception`.
- `[soft]` hooks may use `warn` for non-blocking advisory outcomes, but the warning must be mirrored in Failure List and Change List before progress continues.
- Missing skill activation evidence, missing tool reality evidence, skipped diff review, skipped sensitive scan, unobserved CI/CD, missing cleanup evidence, or missing local runtime sync is `blocked` until converted to `not-applicable` or `exception` with evidence.
- Executable repository hooks are optional; they require an explicit Dependency Decision and fail closed to `blocked`.

### Adversarial Review Standard

Every phase result must be challenged from multiple roles before it is treated as final. The Review Ledger records those challenges and their resolution.

Required review ids by artifact:

| Artifact | Required Reviews |
|---|---|
| `requirements.md` | `requirements_product`, `requirements_engineering`, `requirements_risk` |
| `plan.md` | `plan_architecture`, `plan_validation`, `plan_risk` |
| `verification.md` | `verification_implementation`, `verification_security`, `verification_ci` |
| `delivery-report.md` | `report_delivery`, `report_operations`, `report_user` |

Review rules:

- Each review row must identify the role, perspective, challenge, status, resolution, and evidence.
- Review status must be `pass`, `not-applicable`, `exception`, or `blocked`.
- A blocked review prevents that phase result from becoming final and prevents the next execution phase.
- An exception must identify who or what accepted the risk and must be mirrored in Failure List and Change List.
- The agent may perform the review itself unless the user explicitly asks for subagents, but it must use distinct perspectives rather than one generic self-review.

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

| Gate | Phase | Required work | Exit gate |
|---|---|---|---|
| `G1` | Requirements | Clarify goal, background, success criteria, scope, non-goals, risks, open questions, user decisions, Instruction Evidence, Issue and Prior Attempts, Minimum Skill Dependencies, uncertainty disposition, Requirements Maturity, and the `superpowers:brainstorming` decision. | Requirements are specific enough to design, implement, and verify without unresolved blocking unknowns, unseen existing attempts, or unread applicable instructions. |
| `G2` | Plan | Inspect the repo, compare prior attempts and design options, select an approach, record rejected alternatives, select specialist skills, carry forward Minimum Skill Dependencies, record the first-principles design check, define implementation steps, validation commands, Validation Prerequisites, acceptance criteria, rollback notes, checkpoints, Dependency Decision, Design Readiness, and the `superpowers:writing-plans` decision. | Another agent could implement from the plan without choosing strategy, product behavior, architecture, dependency policy, validation setup, or the minimal owning surface. |
| `G3` | Local Development | Follow `local-repo-development`, including worktree or branch selection and preservation of unrelated changes. | Worktree or branch, base ref, and dirty-state handling are recorded. |
| `G4` | Implementation | Make the scoped change after identifying target behavior, current behavior, owning files/functions, and the minimal affected surface. Avoid unjustified fallback, compatibility, defensive, abstraction, retry, or redundant code. | Changed files are intentional, mapped to acceptance criteria, limited to the owning surface, and free of unjustified fallback or redundant code. |
| `G5` | Verification | Run local checks, review the diff for first-principles fit, surgical change scope, fallback/redundancy, scan for sensitive information, and confirm generated harness state is absent from staged or tracked commit scope. | Validation outcomes, diff review, first-principles fit, surgical scope, fallback/redundancy review, sensitive scan, generated-state git hygiene, and unresolved risks are recorded. |
| `G6` | PR/MR | Commit and open PR/MR when applicable. | PR/MR URL or not-applicable reason is recorded. |
| `G7` | CI/CD | Record CI/CD check state when remote checks exist and choose async follow-up or terminal waiting by policy. | Terminal CI/CD state, async observation state, or not-applicable reason is recorded with evidence. |
| `G8` | Report | Summarize what was delivered, what cleanup was performed or deferred, and what remains. | Delivery report is complete, cleanup is done/not applicable/deferred with evidence, and the report can be sent to the user or attached to PR/MR. |

## Phase State Standard

Every phase and subphase must record state with these sections:

- `Goal`: the concrete outcome this phase or subphase must achieve.
- `Checklist`: objective checks required to exit this phase or subphase.
- `Gate Ledger`: gate id, required evidence, status, evidence pointer, and exception detail.
- `Delegation Ledger`: subagent role, candidate specialist skill, trigger, decision, evidence, and handoff or return requirement.
- `Hook Ledger`: hook id, trigger, required action prefixed with `[hard]` or `[soft]`, status, evidence pointer, and failure handling.
- `Review Ledger`: review id, role, perspective, challenge, status, resolution, and evidence.
- `Todo List`: unfinished actions, preferably with status such as `todo`, `doing`, `blocked`, or `done`.
- `Failure List`: failed commands, blocked checks, rejected assumptions, CI/CD failures, defects found during review, or unresolved risks.
- `Change List`: decisions made, files changed, requirement changes, scope changes, validation changes, or follow-up changes.

Use these status values:

- Phase status: `draft`, `active`, `blocked`, `complete`, `deferred`.
- Todo status: `todo`, `doing`, `blocked`, `done`, `deferred`.
- Failure status: `open`, `investigating`, `fixed`, `accepted`, `deferred`.

When transitioning phases:

- mark checklist items as complete or explicitly deferred,
- update the related Gate Ledger row to `pass`, `not-applicable`, `exception`, or `blocked`,
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

### Gate Ledger

| Gate | Phase | Required Evidence | Status | Evidence | Exception |
|---|---|---|---|---|---|

### Delegation Ledger

| Phase | Subagent Role | Candidate Skill | Trigger | Decision | Evidence | Handoff/Return |
|---|---|---|---|---|---|---|

### Hook Ledger

| Hook | Trigger | Required Action | Status | Evidence | Failure Handling |
|---|---|---|---|---|---|

### Review Ledger

| Review | Role | Perspective | Challenge | Status | Resolution | Evidence |
|---|---|---|---|---|---|---|

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
- dependency decision change,
- branch or worktree change,
- skipped gate or gate exception,
- accepted failing check,
- CI/CD failure accepted as out of scope,
- release or rollback change.
