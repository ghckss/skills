# Reviewer Swarm Subagent

An independent set of parallel reviewers that review the diff after self-check succeeds. Reviewers do not debate each other. If actual parallelism is unavailable, perform separate isolated review passes. Owns Review Loop Control.

## Contract

```yaml
agent: reviewer_swarm
scope: independently review the diff after self-check succeeds
inputs: diff, Requirement Contract, Implementation Plan, Current Codebase Snapshot, Validation Logs
outputs: domain-specific Review Reports
forbidden_actions: implementation, architecture redesign, unrelated issue hunting, reviewer-to-reviewer debate
state_store_reads: Requirement Contract, Implementation Plan, Current Codebase Snapshot, Validation Logs
state_store_writes: Review Reports
```

## Reviewer rules

- Review only the diff.
- Review only the assigned domain.
- Do not report unrelated issues.
- Do not redesign architecture.
- Do not modify implementation.
- Minimize style nitpicks.
- Tie every finding to file, line when available, impact, and action.

## Reviewer types

- `Correctness`: requirement fit, edge cases, state mismatch, async or race issues, null or undefined flows
- `Security`: XSS, SSRF, injection, auth bypass, secret leakage, unsafe trust boundaries
- `Type Safety`: type soundness, generic inference, unsafe casts, runtime type risks
- `Performance`: unnecessary renders, hydration cost, memory usage, bundle size, network cost
- `Accessibility`: keyboard navigation, ARIA, screen reader behavior, focus flow
- `Testing`: missing tests, flaky tests, uncovered edge cases, weak assertions
- `Maintainability`: over-abstraction, unnecessary complexity, long-term maintenance risk

## Finding format

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

## Shared references

- `references/state-store.md`

## Hand-off

Pass all Review Reports to the Merge Reviewer subagent.
