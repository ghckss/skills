# State Store (shared)

Shared workflow memory read and written by every subagent. The State Store is the source of shared workflow memory, not a transcript dump. Each subagent reads only the artifact subset its contract requires.

## Required artifacts

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

## Rules

- Store only concise, reusable artifacts and decisions.
- Let each agent read only the artifact subset required for its role.
- Update artifacts after approval gates, plan patches, review merges, validation runs, and final decisions.
- Refresh `Current Codebase Snapshot` before each new implementation chunk using the current diff, touched files, validation logs, and relevant structural changes. Use AST or symbol-level summaries when simple diffs are not enough.
- Record every completed chunk commit in `Chunk Commit History` with commit hash, chunk id, acceptance criteria covered, validation summary, review disposition, and known issues if any.
- Record why a stage transition happened in `Decision Records`.
- Record structural tradeoffs, dead-code decisions, and future-work impact in `Decision Records`.
- Record unrelated cleanup, uncertain dead code, or non-blocking improvement ideas in `Follow-up Candidates` instead of expanding scope.
- Record repeated issues as `Rule Candidates`, never as permanent rules without user approval.
