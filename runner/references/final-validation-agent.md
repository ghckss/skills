# Final Validation Subagent

Verifies the final result against the approved contract. Runs after the last chunk; its output drives the Refinement Loop entry decision.

## Contract

```yaml
agent: final_validation
scope: verify the final result against the approved contract
inputs: Requirement Contract, Implementation Plan, final diff, Validation Logs, Review Reports, Remaining Known Issues
outputs: Final Validation Result, Remaining Known Issues, Risk Summary
forbidden_actions: new feature scope, unapproved architectural change
state_store_reads: Requirement Contract, Implementation Plan, Validation Logs, Review Reports, Remaining Known Issues
state_store_writes: Validation Logs, Risk Reports, Decision Records
```

## Required work

Verify acceptance criteria, structural integrity, dead-code cleanup scope, and future-workability impact.

## Must verify

- Requirement Contract
- Acceptance Criteria
- Structural integrity and direction
- Dead-code cleanup caused by the work
- Future-workability and collaboration safety
- Regression validation
- Runtime validation
- Test verification

## Final output must include

- `Final Diff`
- `Validation Result`
- `Chunk Commit History`
- `Remaining Known Issues`
- `Follow-up Candidates`
- `Risk Summary`
- `Review Summary`

If a validation command cannot run, record why and explain residual risk.

## Shared references

- `references/state-store.md`
- `references/structural-integrity.md`

## Hand-off

Return results to the Master Orchestrator, which runs the Refinement Loop entry decision (see Refinement Loop in `SKILL.md`).
