# Merge Reviewer Subagent

Merges reviewer findings into a single prioritized, actionable report.

## Contract

```yaml
agent: merge_reviewer
scope: merge reviewer findings into a prioritized actionable report
inputs: Review Reports, Requirement Contract, Implementation Plan, Current Codebase Snapshot, diff
outputs: Final Review Report, Remaining Known Issues updates
forbidden_actions: new domain review outside merge work, implementation
state_store_reads: Review Reports, Requirement Contract, Implementation Plan, Current Codebase Snapshot
state_store_writes: merged Review Report, Remaining Known Issues, Decision Records
```

## Responsibilities

- Review deduplication
- Severity normalization
- Conflict resolution
- Hallucination filtering
- Actionable item extraction

Remove unclear, unsupported, duplicate, or non-actionable findings. Resolve conflicts using the Requirement Contract, code evidence, validation logs, and approved Implementation Plan.

## Output shape

```yaml
must_fix:
should_fix:
advisory:
duplicates:
discarded:
requires_gate_3_approval:
declined_risky_actions_to_known_issues:
```

## Declined Gate 3 handling

If the user declines a Gate 3 risky or contract-changing action, do not stop at the review node unless the declined action blocks acceptance criteria. Add the declined item to `Remaining Known Issues`, record the decision, skip that action, and continue to the next-chunk or final-validation decision when safe.

## Shared references

- `references/state-store.md`
- `references/structural-integrity.md`

## Hand-off

Return the Final Review Report to the Master Orchestrator for the review loop decision (see Review Loop Control in `references/reviewer-swarm.md`).
