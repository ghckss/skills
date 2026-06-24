---
name: runner
description: "State-driven, contract-driven, approval-gated agentic software delivery workflow for coding tasks. Use when Codex is asked to run an orchestrated, multi-agent, swarm-review, incremental implementation, contract-first workflow with explicit pre-implementation approval gates, implementation autopilot after plan approval, post-completion refinement loop, per-chunk commits, structural integrity checks, dead-code cleanup, State Store artifacts, per-role subagent references, review loop control, self-correction, validation, and final risk reporting. 특히 마스터 오케스트레이터, 요구사항 계약, 구현 계획, 승인 게이트, implementation autopilot, 완료 후 개선 루프, chunk 단위 커밋, 구조 검증, dead code 제거, State Store, 역할별 서브에이전트, 리뷰 스웜, 회귀 검증 같은 형식의 작업 플로우를 요청할 때 사용."
---

# Runner

## Core Architecture

Run as a workflow-driven, contract-driven, state-driven, incremental execution system. Do not treat this as a loose multi-agent brainstorm.

The Master Orchestrator does not carry the whole context directly. It routes work through a central State Store and lets each role fetch only the artifacts it needs.

Each role is defined as a bounded subagent in `references/`. The Master Orchestrator stays in this file and dispatches the role subagents in order; each subagent reads only its own reference plus the shared references it needs. Use actual subagents (the harness Agent tool) only when the user explicitly asks for subagents, delegation, parallel agent work, or a review swarm and the tools are available. Otherwise, emulate each role sequentially with isolated passes and clear role labels, following the same reference contracts.

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

## Master Orchestrator

The Master Orchestrator lives in this file. It does not perform role work itself; it dispatches role subagents and controls the gates.

Responsibilities:

- Workflow routing
- Approval gate control
- Conflict resolution
- Retry decision
- Stage transition
- Implementation autopilot enforcement after Gate 2 approval

Forbidden actions:

- Direct implementation
- Direct code review findings outside merge/conflict resolution
- Carrying unbounded full context instead of State Store artifacts
- Skipping approval gates
- Asking for approval during implementation unless a Risk Gate is required

## Subagent Roster

Each role is a bounded subagent defined in its own reference file. Before acting as a role, read its reference and honor its contract; write only the State Store artifacts that contract allows. Shared references are read by multiple roles.

| Subagent | Reference | Dispatch (`subagent_type`) | Dispatched when | Scope |
|---|---|---|---|---|
| Spec | `references/spec-agent.md` | `runner-spec` | at request, before Gate 1 | user request -> Requirement Contract |
| Tech Lead | `references/tech-lead-agent.md` | `runner-tech-lead` | after Gate 1, before Gate 2; Plan Patch review | repo inspection -> Implementation Plan |
| Executor | `references/executor-agent.md` | `runner-executor` | per chunk, after Gate 2 (autopilot) | incremental implementation, self-correction, per-chunk commit |
| Reviewer Swarm | `references/reviewer-swarm.md` | `runner-reviewer-swarm` | after self-check passes | independent domain review + review loop control |
| Merge Reviewer | `references/merge-reviewer.md` | `runner-merge-reviewer` | after Reviewer Swarm | merge findings -> prioritized Final Review Report |
| Final Validation | `references/final-validation-agent.md` | `runner-final-validation` | after the last chunk | verify result against the approved contract |

When the harness supports real subagent dispatch (e.g. the Claude Agent tool), dispatch each role via its `subagent_type` wrapper above; the wrappers live in `.claude/agents/` and read these same references. For the Reviewer Swarm, dispatch one `runner-reviewer-swarm` instance per assigned domain in parallel. When real dispatch is unavailable, emulate each role sequentially with isolated passes and clear role labels, following the same reference contracts. The Master Orchestrator stays in the main session and is not a dispatched subagent.

Shared references:

- `references/state-store.md` — shared workflow memory (artifacts + rules), read/written by all roles.
- `references/structural-integrity.md` — cross-cutting structure and collaboration-safety checks for Tech Lead, Executor, Plan Patch review, Merge Reviewer, and Final Validation.

## Agent Contract Format

Every subagent pass must honor a bounded contract (each reference file fills this in):

```yaml
agent:
scope:
inputs:
outputs:
forbidden_actions:
state_store_reads:
state_store_writes:
```

Do not run a subagent without its role, input, output, and forbidden actions defined.

## Approval Gates

Gate 1: Requirement Contract Approval

- Run immediately after the Spec subagent.
- Include `goals`, `non-goals`, `constraints`, `acceptance criteria`, `edge cases`, and `open questions` (Requirement Contract shape in `references/spec-agent.md`).
- Stop and ask for explicit user approval.
- Before approval, do not plan implementation, generate code, modify files, or make architecture decisions.

Gate 2: Implementation Plan Approval

- Run after the Tech Lead subagent creates the Implementation Plan.
- Include `implementation steps`, `files likely to change`, `architecture impact`, `risks`, `rollout concerns`, and `validation strategy` (Implementation Plan shape in `references/tech-lead-agent.md`).
- Stop and ask for explicit user approval.
- Before approval, do not implement, generate patches, run executor-style implementation commands, or modify files.

Gate 3: Implementation Risk Approval

- After Gate 2 approval, do not ask for approval for ordinary implementation work, validation commands, review repairs, P2/P3/P4 optional fixes, plan refinements inside the approved Requirement Contract, or per-chunk commits.
- Run Gate 3 only when the next action is risky, destructive, or changes the approved Requirement Contract.
- Risky actions include deleting files or directories, `rm`, mass deletion, destructive overwrite, destructive git operations such as reset/clean/restore of user changes, force push, release/deploy/publish actions, modifying files outside the workspace, or exposing/removing secrets.
- Contract-changing actions include scope expansion, changing acceptance criteria, contradicting a non-goal, or making a public API or architecture change that is not covered by the approved Implementation Plan and materially changes user-facing or integration behavior.
- If Gate 3 is required, stop and ask for explicit approval before that action only.
- If Gate 3 is not required, continue without asking.

## Workflow

Use this stage order. Dispatch the named subagent from `references/` at each role step:

```text
request
-> Spec subagent (references/spec-agent.md)
-> Gate 1 approval
-> Tech Lead subagent (references/tech-lead-agent.md)
-> Gate 2 approval
-> enter Implementation Autopilot
-> plan
-> refresh Current Codebase Snapshot
-> Executor subagent: chunk implementation (references/executor-agent.md)
-> self validation
   -> if pass: reviewer swarm
   -> if fail: self repair up to max_self_repair_attempts = 2 -> self validation
      -> if still fail and plan mismatch: Plan Patch Loop -> Tech Lead subagent -> approval if required -> refresh Current Codebase Snapshot -> resume chunk
      -> if still fail and no plan patch applies: Blocker Report -> stop for decision
-> Reviewer Swarm subagent (references/reviewer-swarm.md)
-> Merge Reviewer subagent (references/merge-reviewer.md)
-> review loop decision with max_review_iterations = 2
   -> if normal repair or optional fix: repair -> self validation -> reviewer swarm
   -> if risky/contract-changing action: Gate 3 -> approved action or record Remaining Known Issues/Blocker
   -> if no repair required: next chunk/final decision
-> commit accepted chunk
-> next chunk or final validation
-> Final Validation subagent (references/final-validation-agent.md)
-> result inspection (Refinement Loop entry decision)
   -> if no improvement found: done
   -> if in-frame improvement and budget remains: self_authored_prompt -> (skip Gate 1/2 approval) -> extra chunk -> existing validation/review/commit -> rerun final validation -> inspect again
   -> if frame-exceeding improvement: Gate 3 escalation
   -> if budget exhausted or no-progress: record Remaining Known Issues -> done
```

At each stage transition, update the State Store and record the decision.

Do not leave dangling nodes. Any declined risky or contract-changing action must be recorded in `Remaining Known Issues` or a Blocker Report, linked to the relevant finding or plan patch, and merged back into the next-chunk or final-validation decision when safe.

Do not move to the next chunk until the accepted current chunk has either been committed or a documented commit blocker has been recorded.

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

## Plan Patch Loop

Cross-role protocol: the Executor subagent raises it, the Tech Lead subagent reviews it, the Master Orchestrator routes approval.

If the Executor discovers repo reality mismatch, dependency issue, impossible implementation, or architectural conflict:

1. Create a Plan Patch Request.
2. Stop uncontrolled implementation.
3. Route to the Tech Lead subagent for review.
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

## Refinement Loop

The Refinement Loop is the top-level post-completion extension of Implementation Autopilot. It does not replace or modify the stage-level loops (Sandbox Self-Correction Loop and Chunk Commit Policy in `references/executor-agent.md`, Review Loop Control in `references/reviewer-swarm.md`, Plan Patch Loop above); it wraps a fully completed traversal and autonomously closes in-frame gaps by dispatching the existing subagents again.

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
      -> author self_authored_prompt (skip Spec subagent / Gate 1 / Gate 2 user approval)
      -> validate self_authored_prompt against the frozen frame (in_frame_assertion)
      -> form the improvement into an additional chunk
      -> refresh Current Codebase Snapshot
      -> Executor subagent -> Self-Correction -> Reviewer Swarm subagent -> Merge Reviewer subagent -> (Gate 3 if needed) -> commit
      -> rerun Final Validation subagent
      -> inspect result again (loop)
```

- Skipped: the user-approval stop of Gate 1 (Requirement Contract) and Gate 2 (Implementation Plan). Autopilot performs these autonomously.
- Kept (reused unchanged): the Executor, Reviewer Swarm, Merge Reviewer, and Final Validation subagents and their internal loops, Gate 3 (Risk Gate), and Structural Integrity checks.

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
