---
name: runner
description: "State-driven, contract-driven, approval-gated agentic software delivery workflow for coding tasks. Use when Codex is asked to run an orchestrated, multi-agent, swarm-review, incremental implementation, contract-first workflow with explicit pre-implementation approval gates, implementation autopilot after plan approval, post-completion refinement loop, per-chunk commits, structural integrity checks, dead-code cleanup, State Store artifacts, bounded agent roles, review loop control, self-correction, validation, and final risk reporting. 특히 마스터 오케스트레이터, 요구사항 계약, 구현 계획, 승인 게이트, implementation autopilot, 완료 후 개선 루프, chunk 단위 커밋, 구조 검증, dead code 제거, State Store, 리뷰 스웜, 회귀 검증 같은 형식의 작업 플로우를 요청할 때 사용."
---

# Runner

## Core Architecture

Run as a workflow-driven, contract-driven, state-driven, incremental execution system. Do not treat this as a loose multi-agent brainstorm.

The Master Orchestrator does not carry the whole context directly. It routes work through a central State Store and lets each role fetch only the artifacts it needs.

Use actual subagents only when the user explicitly asks for subagents, delegation, parallel agent work, or a review swarm and the tools are available. Otherwise, emulate the roles sequentially with isolated passes and clear role labels.

## Hard Rules

- NEVER implement immediately after the initial request.
- ALWAYS create the Requirement Contract first.
- ALWAYS stop after the Requirement Contract.
- ALWAYS create the Implementation Plan before implementation.
- ALWAYS stop after creating the Implementation Plan and wait for approval before implementation.
- After Gate 2 approval, run implementation in autopilot mode: do not ask for additional approval for normal implementation, validation, review repair, plan refinement within the approved contract, or per-chunk commits.
- During implementation, ask for approval only for risky actions, destructive actions, or work that changes the approved Requirement Contract.
- NEVER allow reviewer infinite loops.
- NEVER allow reviewer architecture takeover.
- NEVER allow executor uncontrolled replanning.
- ALWAYS prefer incremental execution.
- ALWAYS treat each completed implementation chunk as a default git commit boundary.
- ALWAYS treat the Requirement Contract as the source of truth.
- ALWAYS validate whether the chosen structure and direction are right for the existing system, not only whether the code works.
- ALWAYS remove dead code made obsolete by the current approved work; do not perform unrelated cleanup sweeps.
- NEVER make material decisions silently; record assumptions, tradeoffs, and direction changes in Decision Records and user-facing progress.
- ALWAYS choose structures that reduce future maintainer confusion, cross-user bottlenecks, hidden coupling, and follow-on work friction.
- ALWAYS start the Refinement Loop only after the full workflow traversal completes (all chunks committed, Final Validation produced); never let it replace stage-level loops or enter mid-workflow.
- During Refinement Loop iterations, SKIP Gate 1 and Gate 2 user approval and self-author the next prompt under autopilot — but the initial run still requires Gate 1 and Gate 2 approval.
- NEVER exceed the frame of the approved Requirement Contract and Implementation Plan during autonomous refinement; treat any frame-exceeding improvement as a Gate 3 contract-changing action and stop for approval.
- ALWAYS bound autonomous refinement with `max_refinement_iterations` and exit early on no-progress.
- Treat approval as explicit only. Valid approval examples include `approve`, `proceed`, `go ahead`, `진행해`, `승인`, and `이대로 해`.
- If approval is missing at any gate, stop the workflow.

## State Store

Maintain a compact State Store. The State Store is the source of shared workflow memory, not a transcript dump.

Required artifacts:

- `Requirement Contract`
- `Implementation Plan`
- `Plan Patch History`
- `Review Reports`
- `Validation Logs`
- `Decision Records`
- `Known Constraints`
- `Risk Reports`
- `Current Codebase Snapshot`
- `Remaining Known Issues`
- `Chunk Commit History`
- `Follow-up Candidates`

Rules:

- Store only concise, reusable artifacts and decisions.
- Let each agent read only the artifact subset required for its role.
- Update artifacts after approval gates, plan patches, review merges, validation runs, and final decisions.
- Refresh `Current Codebase Snapshot` before each new implementation chunk using the current diff, touched files, validation logs, and relevant structural changes. Use AST or symbol-level summaries when simple diffs are not enough.
- Record every completed chunk commit in `Chunk Commit History` with commit hash, chunk id, acceptance criteria covered, validation summary, review disposition, and known issues if any.
- Record why a stage transition happened in `Decision Records`.
- Record structural tradeoffs, dead-code decisions, and future-work impact in `Decision Records`.
- Record unrelated cleanup, uncertain dead code, or non-blocking improvement ideas in `Follow-up Candidates` instead of expanding scope.
- Record repeated issues as `Rule Candidates`, never as permanent rules without user approval.

Master Orchestrator responsibilities:

- Workflow routing
- Approval gate control
- Conflict resolution
- Retry decision
- Stage transition
- Implementation autopilot enforcement after Gate 2 approval

Master Orchestrator forbidden actions:

- Direct implementation
- Direct code review findings outside merge/conflict resolution
- Carrying unbounded full context instead of State Store artifacts
- Skipping approval gates
- Asking for approval during implementation unless a Risk Gate is required

## Agent Contract Format

Every agent pass must have a bounded contract:

```yaml
agent:
scope:
inputs:
outputs:
forbidden_actions:
state_store_reads:
state_store_writes:
```

Do not run an agent without defining its role, input, output, and forbidden actions.

## Agent Roles

Spec Agent:

- Scope: translate the user request into the Requirement Contract.
- Inputs: user request, Known Constraints, relevant existing Decision Records.
- Outputs: Requirement Contract.
- Forbidden actions: implementation planning, code generation, architecture decision, file modification.
- State Store writes: Requirement Contract, open questions, Known Constraints.

Tech Lead Agent:

- Scope: inspect the repo and design the implementation plan after Gate 1 approval.
- Inputs: approved Requirement Contract, Known Constraints, relevant repository evidence.
- Outputs: Implementation Plan, Task Breakdown, Risk Report.
- Forbidden actions: code modification, patch generation, executor execution.
- State Store writes: Implementation Plan, Risk Report, Decision Records.
- Required work: repo inspection, architecture analysis, dependency analysis, implementation strategy, task breakdown, validation strategy, risk analysis, structural integrity check, future-workability check, dead-code cleanup plan.

Executor Agent:

- Scope: implement approved plan chunks incrementally.
- Inputs: approved Implementation Plan, current task chunk, Current Codebase Snapshot, Known Constraints, Risk Report.
- Outputs: incremental patch, chunk commit when the chunk reaches an accepted state, Implementation Notes, Validation Logs, Plan Patch Request when needed, updated Current Codebase Snapshot.
- Forbidden actions: large one-shot implementation, uncontrolled replanning, architecture changes without Plan Patch Loop routing and Risk Gate approval when required.
- State Store writes: Implementation Notes, Validation Logs, Current Codebase Snapshot, Chunk Commit History, Plan Patch History when applicable.
- Required work: remove dead code made obsolete by the chunk, avoid hidden coupling, and record material implementation tradeoffs.

Reviewer Swarm:

- Scope: independently review the diff after self-check succeeds.
- Inputs: diff, Requirement Contract, Implementation Plan, Current Codebase Snapshot, Validation Logs.
- Outputs: domain-specific Review Reports.
- Forbidden actions: implementation, architecture redesign, unrelated issue hunting, reviewer-to-reviewer debate.
- State Store writes: Review Reports.

Merge Reviewer:

- Scope: merge reviewer findings into a prioritized actionable report.
- Inputs: Review Reports, Requirement Contract, Implementation Plan, Current Codebase Snapshot, diff.
- Outputs: Final Review Report, Remaining Known Issues updates.
- Forbidden actions: new domain review outside merge work, implementation.
- State Store writes: merged Review Report, Remaining Known Issues, Decision Records.

Final Validation Agent:

- Scope: verify the final result against the approved contract.
- Inputs: Requirement Contract, Implementation Plan, final diff, Validation Logs, Review Reports, Remaining Known Issues.
- Outputs: Final Validation Result, Remaining Known Issues, Risk Summary.
- Forbidden actions: new feature scope, unapproved architectural change.
- State Store writes: Validation Logs, Risk Reports, Decision Records.
- Required work: verify acceptance criteria, structural integrity, dead-code cleanup scope, and future-workability impact.

## Approval Gates

Gate 1: Requirement Contract Approval

- Run immediately after the Spec Agent.
- Include `goals`, `non-goals`, `constraints`, `acceptance criteria`, `edge cases`, and `open questions`.
- Stop and ask for explicit user approval.
- Before approval, do not plan implementation, generate code, modify files, or make architecture decisions.

Requirement Contract shape:

```yaml
goals:
non_goals:
constraints:
acceptance_criteria:
edge_cases:
open_questions:
```

Gate 2: Implementation Plan Approval

- Run after the Tech Lead Agent creates the Implementation Plan.
- Include `implementation steps`, `files likely to change`, `architecture impact`, `risks`, `rollout concerns`, and `validation strategy`.
- Stop and ask for explicit user approval.
- Before approval, do not implement, generate patches, run executor-style implementation commands, or modify files.

Implementation Plan shape:

```yaml
implementation_steps:
files_likely_to_change:
architecture_impact:
risks:
rollout_concerns:
validation_strategy:
task_breakdown:
chunk_commit_boundaries:
structural_integrity_check:
dead_code_cleanup_plan:
future_workability_check:
```

Gate 3: Implementation Risk Approval

- After Gate 2 approval, do not ask for approval for ordinary implementation work, validation commands, review repairs, P2/P3/P4 optional fixes, plan refinements inside the approved Requirement Contract, or per-chunk commits.
- Run Gate 3 only when the next action is risky, destructive, or changes the approved Requirement Contract.
- Risky actions include deleting files or directories, `rm`, mass deletion, destructive overwrite, destructive git operations such as reset/clean/restore of user changes, force push, release/deploy/publish actions, modifying files outside the workspace, or exposing/removing secrets.
- Contract-changing actions include scope expansion, changing acceptance criteria, contradicting a non-goal, or making a public API or architecture change that is not covered by the approved Implementation Plan and materially changes user-facing or integration behavior.
- If Gate 3 is required, stop and ask for explicit approval before that action only.
- If Gate 3 is not required, continue without asking.

## Workflow

Use this stage order:

```text
request
-> Spec Agent
-> Gate 1 approval
-> Tech Lead Agent
-> Gate 2 approval
-> enter Implementation Autopilot
-> plan
-> refresh Current Codebase Snapshot
-> chunk implementation
-> self validation
   -> if pass: reviewer swarm
   -> if fail: self repair up to max_self_repair_attempts = 2 -> self validation
      -> if still fail and plan mismatch: Plan Patch Loop -> Tech Lead Agent -> approval if required -> refresh Current Codebase Snapshot -> resume chunk
      -> if still fail and no plan patch applies: Blocker Report -> stop for decision
-> reviewer swarm
-> merge reviewer
-> review loop decision with max_review_iterations = 2
   -> if normal repair or optional fix: repair -> self validation -> reviewer swarm
   -> if risky/contract-changing action: Gate 3 -> approved action or record Remaining Known Issues/Blocker
   -> if no repair required: next chunk/final decision
-> commit accepted chunk
-> next chunk or final validation
-> final validation
-> result inspection (Refinement Loop entry decision)
   -> if no improvement found: done
   -> if in-frame improvement and budget remains: self_authored_prompt -> (skip Gate 1/2 approval) -> extra chunk -> existing validation/review/commit -> rerun final validation -> inspect again
   -> if frame-exceeding improvement: Gate 3 escalation
   -> if budget exhausted or no-progress: record Remaining Known Issues -> done
```

At each stage transition, update the State Store and record the decision.

Do not leave dangling nodes. Any declined risky or contract-changing action must be recorded in `Remaining Known Issues` or a Blocker Report, linked to the relevant finding or plan patch, and merged back into the next-chunk or final-validation decision when safe.

Do not move to the next chunk until the accepted current chunk has either been committed or a documented commit blocker has been recorded.

## Incremental Execution

Executor Agent rules:

- Implement in small task chunks from the approved plan.
- Create incremental patches; never attempt large implementation in one pass.
- Follow the loop: plan chunk, implement chunk, self-validate, review, then continue.
- Keep each chunk tied to one or more acceptance criteria.
- Treat each chunk as a default git commit boundary.
- Commit each accepted chunk before starting the next chunk.
- Refresh executor context before every chunk from `Current Codebase Snapshot`, current diff, touched files, validation logs, decision records, and updated constraints.
- When prior chunks changed structure, APIs, or type surfaces, summarize those changes before starting the next chunk. Use AST or symbol-level summaries when plain diff context is too noisy or incomplete.
- Stop and create a Plan Patch Request when repo reality invalidates the approved plan.

## Sandbox Self-Correction Loop

Before Reviewer Swarm, and after every Repair, the Executor must run self-checks appropriate to the repo:

- lint
- typecheck
- test
- build
- runtime validation

If checks fail:

- Attempt automatic repair.
- Set `max_self_repair_attempts = 2`.
- Record each attempt in Validation Logs.
- After two failed attempts, stop the self-repair loop.
- If the failure indicates repo reality mismatch, dependency issue, impossible implementation, or architectural conflict, create a Plan Patch Request.
- If the failure is not a plan issue, create a Blocker Report with the failing commands, error summary, attempted repairs, and recommended next decision.
- Run Reviewer Swarm only after self-check succeeds.

Blocker Report shape:

```yaml
failure_type:
commands:
error_summary:
self_repair_attempts:
suspected_cause:
recommended_next_step:
plan_patch_required:
```

## Chunk Commit Policy

Default assumption: every accepted implementation chunk becomes one git commit.

Commit timing:

- Commit only after the chunk reaches an accepted state.
- A chunk is accepted when self validation passes, Reviewer Swarm and Merge Reviewer have completed, required P0/P1 repairs are resolved, Gate 3 decisions are recorded, and remaining advisory issues do not block acceptance criteria.
- Do not commit before Gate 2 approval.
- Do not commit immediately after raw implementation.
- Do not commit failed, blocked, unvalidated, or unreviewed chunks.
- Do not start the next chunk until the current accepted chunk is committed or a commit blocker is recorded.

Staging rules:

- Stage only files intentionally changed for the current chunk.
- Never stage unrelated user changes.
- If unrelated dirty changes overlap with files needed for the chunk, stop and ask for a decision.
- Include generated files or lockfiles only when they are required by the chunk.
- Never push by default.

Commit message rules:

- Use a concise message that identifies the chunk and intent.
- Prefer this format: `runner chunk <n>: <short purpose>`.
- Mention the acceptance criteria covered in the State Store, not necessarily in the commit subject.

Commit blocker handling:

- If the repository is not a git repo, git is unavailable, hooks fail, or commit creation is blocked, record a `Commit Blocker` in Decision Records.
- If hooks fail due to lint, type, test, build, or runtime errors, return to Self Validation or Blocker Report instead of bypassing hooks.
- If the user explicitly disables per-chunk commits for a task, record that override in Decision Records and continue with uncommitted chunks only for that task.

Commit Blocker shape:

```yaml
chunk_id:
reason:
commands:
status:
recommended_next_step:
```

## Reviewer Swarm

Reviewer Swarm is an independent set of parallel reviewers. Reviewers do not debate each other. If actual parallelism is unavailable, perform separate isolated review passes.

Reviewer rules:

- Review only the diff.
- Review only the assigned domain.
- Do not report unrelated issues.
- Do not redesign architecture.
- Do not modify implementation.
- Minimize style nitpicks.
- Tie every finding to file, line when available, impact, and action.

Reviewer types:

- `Correctness`: requirement fit, edge cases, state mismatch, async or race issues, null or undefined flows
- `Security`: XSS, SSRF, injection, auth bypass, secret leakage, unsafe trust boundaries
- `Type Safety`: type soundness, generic inference, unsafe casts, runtime type risks
- `Performance`: unnecessary renders, hydration cost, memory usage, bundle size, network cost
- `Accessibility`: keyboard navigation, ARIA, screen reader behavior, focus flow
- `Testing`: missing tests, flaky tests, uncovered edge cases, weak assertions
- `Maintainability`: over-abstraction, unnecessary complexity, long-term maintenance risk

Finding format:

```text
[P#] Title - file:line - issue - impact - action
```

## Review Loop Control

Use `max_review_iterations = 2`.

Severity:

- `P0`: security or correctness
- `P1`: runtime or type stability
- `P2`: performance
- `P3`: accessibility
- `P4`: style or readability

Rules:

- Fix P0 and P1 findings.
- Treat P2 based on impact and fix it without asking when it is inside the approved contract and not risky.
- Treat P3 and P4 as advisory unless they block acceptance criteria; fix them without asking when low-risk and cheap.
- After approved repair work, return to Self Validation before any next chunk or final validation.
- Re-enter Reviewer Swarm only after Self Validation passes.
- Count each repair-and-review cycle against `max_review_iterations = 2`.
- Mark repeated findings as duplicates.
- End the review loop when remaining items are duplicate, advisory, or low-impact nitpicks.
- Store advisory issues in Remaining Known Issues when not fixed.

Exit strategy:

- Allow workflow exit after repeated review findings.
- If P0 or P1 remains after max review iterations, stop with a Blocker Report unless the issue is explicitly accepted by the user as risk.
- If P2 or lower remains after max review iterations, record it as Remaining Known Issues when it does not block acceptance criteria.
- Do not chase infinite polish.
- Never let reviewers take over architecture.

## Merge Reviewer

Merge Reviewer responsibilities:

- Review deduplication
- Severity normalization
- Conflict resolution
- Hallucination filtering
- Actionable item extraction

Remove unclear, unsupported, duplicate, or non-actionable findings. Resolve conflicts using the Requirement Contract, code evidence, validation logs, and approved Implementation Plan.

Final Review Report shape:

```yaml
must_fix:
should_fix:
advisory:
duplicates:
discarded:
requires_gate_3_approval:
declined_risky_actions_to_known_issues:
```

If the user declines a Gate 3 risky or contract-changing action, do not stop at the review node unless the declined action blocks acceptance criteria. Add the declined item to `Remaining Known Issues`, record the decision, skip that action, and continue to the next-chunk or final-validation decision when safe.

## Plan Patch Loop

If the Executor discovers repo reality mismatch, dependency issue, impossible implementation, or architectural conflict:

1. Create a Plan Patch Request.
2. Stop uncontrolled implementation.
3. Route to Tech Lead Agent for review.
4. Update the Implementation Plan.
5. Continue without user approval when the patch stays inside the approved Requirement Contract and does not require a risky action.
6. Request user approval only when the patch changes the approved Requirement Contract, expands scope, contradicts a non-goal, materially changes public API or architecture outside the approved plan, or requires a risky action.
7. Resume Executor after the updated plan is recorded, or after approval when approval is required.

Executor must never make independent architecture changes.

Plan Patch Loop can be entered from chunk implementation, self validation, self repair exhaustion, repair validation, or reviewer findings. After any approved plan patch, refresh `Current Codebase Snapshot` before resuming execution.

Plan Patch Request shape:

```yaml
trigger:
evidence:
current_plan_conflict:
proposed_plan_patch:
risk_change:
approval_required:
```

## Implementation Autopilot

Implementation Autopilot starts immediately after Gate 2 approval.

In autopilot mode, continue without asking the user for approval for:

- Chunk implementation inside the approved Implementation Plan
- Self validation commands
- Self repair attempts within `max_self_repair_attempts`
- Reviewer Swarm and Merge Reviewer
- P0/P1 required repairs that do not require risky actions
- P2/P3/P4 fixes that are low-risk and inside the approved Requirement Contract
- Plan Patch updates that stay inside the approved Requirement Contract
- Per-chunk git staging and commit of intentionally changed files
- Final validation

Autopilot must stop for Gate 3 only when the next action is risky or contract-changing:

- File or directory deletion, `rm`, mass delete, destructive overwrite, or irreversible data loss
- Destructive git operations such as `reset --hard`, `clean`, restoring or overwriting user changes, force push, or history rewrite
- Push, deploy, publish, release, or external side effects
- Dependency installation or network actions that materially change the project environment when not already approved by the plan
- Modifying files outside the approved workspace or writable roots
- Secret exposure, secret deletion, permission changes, or credential handling
- Scope expansion, acceptance criteria change, non-goal contradiction, or public API/architecture change outside the approved plan

When unsure whether an action is risky, classify it as risky and ask.

Autopilot extends to the post-completion Refinement Loop: stay autonomous for in-frame improvements after Final Validation, and stop only when a Gate 3 risky or contract-changing action is required. See `Refinement Loop`.

## Structural Integrity And Collaboration Safety

Apply these checks during Tech Lead planning, each Executor chunk, Plan Patch review, Merge Reviewer decisions, and Final Validation:

- Verify the direction: confirm the implementation matches the repo's existing architecture, ownership boundaries, naming conventions, and abstraction level.
- Prefer the least surprising structure that solves the contract. Add abstractions only when they remove real complexity or match established local patterns.
- Remove dead code made obsolete by the current approved work, including unused branches, obsolete helpers, superseded tests, and stale references.
- Do not perform unrelated legacy cleanup. Put uncertain dead code or broad cleanup ideas in `Follow-up Candidates`.
- Avoid hidden coupling, ambiguous ownership, duplicated sources of truth, and structures that make the next task harder to start.
- Preserve future workability: the next contributor should be able to infer where related code belongs, what contract changed, and which tests guard it.
- No silent decisions: do not quietly choose a material tradeoff, new convention, or structural direction. Record it in `Decision Records` and mention it in progress or final output.
- During autopilot, "no silent decisions" means transparent recording and updates, not extra approval prompts. Ask only when a Risk Gate is required.

## Validation

Final Validation must verify:

- Requirement Contract
- Acceptance Criteria
- Structural integrity and direction
- Dead-code cleanup caused by the work
- Future-workability and collaboration safety
- Regression validation
- Runtime validation
- Test verification

Final output must include:

- `Final Diff`
- `Validation Result`
- `Chunk Commit History`
- `Remaining Known Issues`
- `Follow-up Candidates`
- `Risk Summary`
- `Review Summary`

If a validation command cannot run, record why and explain residual risk.

## Refinement Loop

The Refinement Loop is the top-level post-completion extension of Implementation Autopilot. It does not replace or modify the stage-level loops (Sandbox Self-Correction Loop, Review Loop Control, Plan Patch Loop); it wraps a fully completed traversal and autonomously closes in-frame gaps.

### Entry

- Precondition: the full workflow traversal must be complete — all planned chunks committed and a Final Validation Result produced. Never enter mid-workflow (mid-chunk or mid-review).
- Trigger: result inspection after Final Validation finds an improvement point or deficiency.
- Inspection sources: original Requirement Contract `acceptance_criteria` coverage, `Remaining Known Issues`, `Risk Summary`, deferred advisory review findings, Structural Integrity / future-workability gaps, and acceptance criteria only weakly satisfied by Final Validation.
- `Follow-up Candidates` (out-of-frame or uncertain items) are NOT autonomous-loop targets; exclude them from inspection or treat them only as Gate 3 candidates.

### Loop Contract

Record in Decision Records when starting the loop:

```yaml
loop_id: refinement_loop
trigger:              # the improvement/deficiency found (which acceptance_criterion / known issue)
frame_reference:      # approved Requirement Contract + Implementation Plan (frozen frame = loop invariant)
self_authored_prompt: # the next-iteration objective authored by the orchestrator (no Gate 1/2 user approval)
in_frame_assertion:   # explicit confirmation this iteration stays in frame (no goals/non-goals/architecture violation)
iteration_bound:      # max_refinement_iterations = 2 (hard cap, no infinite polish)
progress_metric:      # unresolved improvement count decreasing + satisfied acceptance_criteria increasing
exit_conditions:      # see Exit
state_writes:         # Decision Records, Validation Logs, Remaining Known Issues, Chunk Commit History, Follow-up Candidates
```

Per the no-silent-decisions rule, always record `self_authored_prompt` and the in-frame judgment in Decision Records and user-facing progress. Only the approval prompt is skipped, never the artifact.

### Iteration Flow (skip Gate 1/2, keep autopilot)

```text
completed workflow result (after final validation)
-> result inspection (identify improvement/deficiency)
   -> no improvement found: done -> Final Response
   -> frame-exceeding improvement required: Gate 3 (Risk Gate) stop -> user escalation
   -> in-frame improvement and iteration_bound not exhausted:
      -> author self_authored_prompt (skip Spec Agent / Gate 1 / Gate 2 user approval)
      -> validate self_authored_prompt against the frozen frame (in_frame_assertion)
      -> form the improvement into an additional chunk
      -> refresh Current Codebase Snapshot
      -> chunk implementation -> Self-Correction -> Reviewer Swarm -> Merge Reviewer -> (Gate 3 if needed) -> commit
      -> rerun Final Validation
      -> inspect result again (loop)
```

- Skipped: the user-approval stop of Gate 1 (Requirement Contract) and Gate 2 (Implementation Plan). Autopilot performs these autonomously.
- Kept (reused unchanged): Incremental Execution, Sandbox Self-Correction Loop, Reviewer Swarm, Merge Reviewer, Gate 3 (Risk Gate), Chunk Commit Policy, Structural Integrity checks, Final Validation.

### Exit

- `success_exit`: no in-frame improvement remains.
- `budget_exit`: `max_refinement_iterations = 2` reached; record remaining items in Remaining Known Issues + Risk Summary, then stop.
- `no_progress_exit`: if an iteration fails to reduce unresolved improvements or reintroduces the same item, stop before the bound and record `no_progress` in Decision Records.
- `escalation_exit`: a frame-exceeding improvement stops at Gate 3 for the user.

### Frame Invariant (top priority, linked to Gate 3)

- Frozen frame = approved Requirement Contract (`goals`/`non_goals`/`constraints`/`acceptance_criteria`) + approved Implementation Plan (`architecture_impact`, approach).
- Autonomous iterations are allowed only inside the frame: no new goal, no non-goal violation, no scope expansion, no public API/architecture change — identical to Gate 3's "contract-changing action" definition.
- `self_authored_prompt` targets only the identified improvement and introduces no new feature.
- Before each iteration, compare `self_authored_prompt` against the frame; on violation, go straight to Gate 3 (`escalation_exit`).
- Risky or destructive actions (`rm`, force push, deploy, etc.) stop at Gate 3 even when done for improvement.

### Cautions

- Start only after the full traversal completes — never replace stage-level loops, never enter mid-workflow.
- Skip only the Gate 1/2 approval prompts; still record `self_authored_prompt`, the in-frame judgment, and decisions (no-silent-decisions).
- The frame invariant is the top priority — even an attractive improvement cannot proceed autonomously when out of frame; route to Gate 3.
- No infinite polish — `max_refinement_iterations` hard cap plus no-progress early exit guarantee convergence (consistent with "Do not chase infinite polish").
- When "in-frame improvement" vs "out-of-frame change" is ambiguous, default to out-of-frame and escalate to Gate 3 (same principle as autopilot's "when unsure, classify as risky and ask").
- Each iteration ends with no dangling nodes — unresolved items to Remaining Known Issues, uncertain or out-of-frame ideas to Follow-up Candidates.
- The initial run still requires Gate 1 and Gate 2 user approval; approval skipping applies only to post-completion refinement iterations.

### Exception Handling

- Frame-exceeding improvement: when it requires changing goals/non-goals/architecture/public API/acceptance, stop at Gate 3 and present the proposed frame delta plus rationale. If the user wants it, proceed as a new workflow from Gate 1.
- Budget exhausted: after `max_refinement_iterations`, record remaining items in Remaining Known Issues + Risk Summary and stop (no silent infinite loop).
- No-progress: improvement count not decreasing or the same item reintroduced -> early exit.
- Regression introduced: if a refinement iteration breaks a previously satisfied acceptance criterion, the rerun Final Validation catches it; if it cannot be resolved in-frame, roll back that iteration or go to Gate 3.
- Conflicting improvement: if fixing one improvement violates another acceptance criterion, treat as unresolvable in-frame and go to Gate 3.
- Unresolved original `open_questions`: cannot be decided autonomously -> escalate.
- Self-authored prompt drift: if the frame comparison detects drift, block that iteration before execution.

## Memory And Rule Learning

Store repeated reviewer results or recurring mistakes as `Rule Candidates`.

Do not permanently update project guidelines, reviewer rules, or architecture conventions without explicit user approval. Present the candidate, evidence, benefit, and risk before promotion.

## Cost Optimization

When model or execution selection is available:

- Use high-capability models for Master Orchestrator, Tech Lead, and Merge Reviewer.
- Use lighter models for P3 and P4 advisory review when appropriate.
- Prefer deterministic tooling for lint, typecheck, tests, build, and runtime checks.
- Do not use the highest-capability model for every stage by default.

## Final Response

Keep the final user response concise and evidence-based. Include changed paths, chunk commits, validation commands, remaining known issues, and risk summary. Do not include internal role transcripts unless the user asks for them.
