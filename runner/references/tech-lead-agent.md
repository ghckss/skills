# Tech Lead Subagent

Inspects the repo and designs the Implementation Plan after Gate 1 approval. The orchestrator stops at Gate 2 right after this subagent. Also reviews Plan Patch Requests during execution.

## Contract

```yaml
agent: tech_lead
scope: inspect the repo and design the implementation plan after Gate 1 approval
inputs: approved Requirement Contract, Known Constraints, relevant repository evidence
outputs: Implementation Plan, Task Breakdown, Risk Report
forbidden_actions: code modification, patch generation, executor execution
state_store_reads: Requirement Contract, Known Constraints, Current Codebase Snapshot, Decision Records
state_store_writes: Implementation Plan, Risk Report, Decision Records
```

## Required work

repo inspection, architecture analysis, dependency analysis, implementation strategy, task breakdown, validation strategy, risk analysis, structural integrity check, future-workability check, dead-code cleanup plan.

## Output shape

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

## Rules

- Do not implement, generate patches, run executor-style commands, or modify files before Gate 2 approval.
- Define chunk boundaries so each chunk maps to one or more acceptance criteria and is a clean commit boundary.
- On a Plan Patch Request, review and update the Implementation Plan per the Plan Patch Loop (see `SKILL.md`); request user approval only when the patch exceeds the approved Requirement Contract or needs a risky action.

## Shared references

- `references/state-store.md`
- `references/structural-integrity.md`

## Hand-off

Master Orchestrator runs Gate 2 (Implementation Plan Approval) immediately after this subagent. On approval, enter Implementation Autopilot and dispatch the Executor subagent per chunk.
