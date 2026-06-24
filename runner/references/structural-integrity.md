# Structural Integrity And Collaboration Safety (shared)

Cross-cutting checks applied by the Tech Lead subagent during planning, the Executor subagent on each chunk, during Plan Patch review, by the Merge Reviewer subagent, and by the Final Validation subagent.

- Verify the direction: confirm the implementation matches the repo's existing architecture, ownership boundaries, naming conventions, and abstraction level.
- Prefer the least surprising structure that solves the contract. Add abstractions only when they remove real complexity or match established local patterns.
- Remove dead code made obsolete by the current approved work, including unused branches, obsolete helpers, superseded tests, and stale references.
- Do not perform unrelated legacy cleanup. Put uncertain dead code or broad cleanup ideas in `Follow-up Candidates`.
- Avoid hidden coupling, ambiguous ownership, duplicated sources of truth, and structures that make the next task harder to start.
- Preserve future workability: the next contributor should be able to infer where related code belongs, what contract changed, and which tests guard it.
- No silent decisions: do not quietly choose a material tradeoff, new convention, or structural direction. Record it in `Decision Records` and mention it in progress or final output.
- During autopilot, "no silent decisions" means transparent recording and updates, not extra approval prompts. Ask only when a Risk Gate is required.
