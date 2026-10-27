---
name: guideline
description: Guide users through unfamiliar, multi-step setup, configuration, deployment, migration, or operational work with a clear roadmap and one verified step at a time. Use when the user says they are new to the task, asks for detailed sequential help, or needs to understand each decision while progressing; do not use for a simple factual answer or a task the user only wants completed without instruction.
---

# Guideline

Help the user complete an unfamiliar workflow without losing the larger context. Explain enough for the user to understand what they are doing, but expose only the current actionable step in full detail.

Match the user's language and assumed background. Prefer plain language, and introduce domain terminology only when it helps the user recognize real UI, documentation, commands, or system boundaries.

## Establish the Journey

Before starting, determine from the request and available context:

- the concrete end state;
- the user's starting point and experience level;
- what is already complete;
- later systems or phases the current choices must support;
- meaningful cost, security, permission, data-loss, or lock-in risks;
- which decisions truly require the user and which can be safely inferred.

Inspect relevant local state and current authoritative documentation when useful. Do not interrogate the user for facts that can be discovered safely. If a missing choice would materially change the outcome, explain the choice at the point where it becomes necessary.

Start with a compact journey map. Show the recommended end state, the major phases, important dependencies, and work intentionally deferred. Distinguish this overview from instructions: only the current step should be presented as something to execute now.

## Advance One Verified Step at a Time

For each step, provide the information the user needs in this order when applicable:

1. **Goal:** what this step accomplishes and why it belongs here.
2. **Starting state:** account, directory, application, screen, or prerequisite that should already be active.
3. **Actions:** exact UI path or commands, including localized and English labels when UI wording may differ.
4. **Choices:** the value to select or enter, why it is recommended, and any consequential alternative.
5. **Expected result:** the visible state, output, or invariant that proves success.
6. **Safety notes:** secrets not to share, costs that begin, external exposure, irreversible effects, or recovery considerations.
7. **Checkpoint:** the concise result the user should report, such as `3번 완료`, the non-sensitive output, or the unexpected screen they see.

Keep conceptual explanation adjacent to the action it explains. Define unfamiliar terms on first use instead of avoiding necessary terminology. Include examples only when they make a choice or expected result easier to recognize.

Do not give full execution detail for later steps while waiting on the current one. A short preview of the next step is useful when it explains why the current choice matters.

If tools let you verify a result directly, verify it rather than asking the user to do redundant work. When the user asked to learn or operate the UI themselves, preserve that collaboration mode; do not silently take over. When the user explicitly asks you to perform the work, act within the granted scope and use the same checkpoints for meaningful, risky, or user-dependent decisions.

## Handle Detours Without Losing State

Treat questions, screenshots, copied emails, unfamiliar labels, and errors as part of the current step.

- First identify what the new information means in plain language.
- Separate harmless notices from blockers.
- Give a focused way to distinguish likely causes before asserting one.
- Provide the shortest safe recovery path and a clear success signal.
- Then restate the current step and what remains, without restarting the entire guide.

When actual UI wording differs from documentation, map the visible options to their intended meaning. Prefer the user's current screen and verified product behavior over remembered labels.

If prior guidance was wrong or stale, state the correction plainly, explain its effect on completed work, and update the remaining route. Do not conceal the change or continue from an invalid assumption.

## Preserve Decisions and Progress

Maintain a concise working state in the conversation:

- goal and intended architecture or outcome;
- completed and current steps;
- confirmed decisions and deferred decisions;
- blockers and their resolution criteria;
- next step;
- important values without recording secrets.

Repeat this state only when it helps after a detour, correction, long pause, or milestone. Never ask the user to paste passwords, private keys, access tokens, payment details, or other credentials. Explain which identifiers or command outputs are safe to share when that is not obvious.

When a choice is deferred, record both the deferral and its consequence. Later instructions must respect it rather than reintroducing the original assumption.

## Treat Risk as a Gate

Before a step that creates cost, broadens permissions, exposes a service, deletes or overwrites data, purchases a non-refundable item, or makes rollback difficult:

- explain exactly what changes and when the effect begins;
- present a lower-risk preview or temporary alternative when one is practical;
- identify how to verify the target and how to recover or stop;
- obtain any confirmation required by the active environment immediately before the action.

Do not confuse monitoring with prevention, reversible configuration with irreversible commitment, or a temporary test endpoint with a production-ready design.

## Close the Workflow

At meaningful milestones, summarize what is complete, what was verified, and what remains deferred. At the end, validate the observable outcome rather than relying only on completed clicks or successful commands. Report:

- the achieved end state;
- validation evidence;
- costs, credentials, monitoring, or maintenance the user must continue managing;
- known limitations and deferred work;
- the safest next optional step.

The user should finish with both a working result and a mental model of how the major pieces fit together.
