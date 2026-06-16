---
name: plan
description: Produce a written implementation handoff plan that a future executor can use without relying on the original conversation. Use only when the user explicitly asks to plan something (e.g. "/plan", "plan this", "spec this out"), or after confirming that planning is the right next step rather than execution.
---

# Plan

Produce a human-readable plan artifact that captures the goal, constraints, decisions, tradeoffs, and work breakdown well enough for a future executor — human, agent, or automation — to act autonomously without re-deriving the original conversation.

The artifact is the point. Do the heavy thinking here: clarify ambiguity, explore the existing system, record decisions, identify edge cases, and write the plan so it reads cold.

A plan is a handoff document, not a permanent specification system. Avoid unnecessary process, taxonomy, or maintenance structure unless the work actually needs it.

## Process

### 1. Clarify

Resolve open requirements, scope boundaries, edge cases, and tradeoffs in conversation. Ask the user directly in prose for gaps that matter. **Do not finish the plan with unresolved user-facing questions.**

If invoked cold, spend extra time here. If invoked after prior discussion, preserve the decisions already made instead of reopening them.

**Investigation tasks are the only allowed escape hatch.** Use them only when the answer genuinely requires implementation or runtime evidence — never as a way to defer a decision the user could make in conversation. Label them explicitly, give concrete acceptance, and make them produce a recorded decision or choose between already-described branches.

### 2. Explore

Explore enough to understand existing patterns, integration points, constraints, and whether the requested work already exists. Never assume functionality is absent — check. Stop when more exploration is unlikely to change the plan.

The output can be informal while exploring: notes, file pointers, discovered patterns, and constraints. Use these when writing the artifact.

### 3. Choose shape

Use the lightest shape that preserves autonomy.

- **Single file** — one plan file with context, acceptance, file pointers, and a task list inline. Use for cohesive, contained work.
- **Multi-file plan** — an index file owns the overview and task list; task files and/or supporting docs carry the details. Use when shared context, distinct workstreams, or non-obvious design decisions would make a single file noisy.

Use multiple files when any of these apply:

- Extensive shared context across multiple tasks would otherwise be duplicated.
- The work introduces domain concepts, APIs, data models, workflows, or architecture that deserve their own design documents or explanations.
- Several independently understandable tasks need different acceptance criteria or implementation notes.
- The plan needs generic supporting context that is not itself a task contract.

Number of tasks alone is a weak signal. If genuinely ambiguous, ask the user.

#### Multi-file guidance

Distinguish task-specific files from supporting docs.

- **Task files** describe one implementation milestone. They can include objective, scope, acceptance, verification, and source context.
- **Supporting docs** explain shared context. Name and structure them around the actual subject matter, not a required taxonomy. They may describe background, constraints, decisions, API shape, data model, edge cases, integration notes, non-goals, or tradeoffs.

Do not make a supporting doc look like a task contract unless it really is one. If a supporting doc includes acceptance criteria, make clear whether they are global criteria for the whole plan or contextual constraints for downstream tasks.

A typical multi-file layout may look like this, but adapt names and files to the work:

```text
plans/<slug>/
  index.md
  <supporting-context>.md
  tasks/
    001-<task>.md
    002-<task>.md
```

### 4. Choose location

- Detect whether the repo already has `plans/`, `docs/plans/`, `tmp/plans/`, or similar. Use existing conventions when present.
- Otherwise default to `plan-<slug>.md` for a single file, or `plans/<slug>/` for multi-file plans.
- Confirm the path with the user before writing. Never write to an unconfirmed path.
- Never overwrite an existing plan artifact without explicit confirmation.

### 5. Write

Use the templates in `templates/` as starting points when helpful. Adapt headings to the work; do not force empty sections.

Default conventions for implementation handoff plans:

- Include an explicit task or checklist section for implementation work under `## Tasks`.
- Order tasks by dependency, then priority.
- Every task must be a meaningful, atomic implementation milestone, not a small mechanical step.
- Record observable acceptance criteria where they help autonomy.
- Include verification guidance where the executor would otherwise have to guess.
- Capture contracts and decisions over code choreography.
  - Good: `POST /widgets` with missing `name` returns 400 with `errors[0].field === "name"`. Validation uses the project's existing schema pattern.
  - Bad: In `widgets.ts`, add a `validateWidget(input)` function and call it at the top of the handler.
- In task lists, keep task titles as plain text. If a task has its own file, add a markdown link after the task
- Use normal markdown links for supporting docs unless the surrounding project already has a stronger convention.
- Keep source pointers as breadcrumbs, not step-by-step implementation instructions.
- If the artifact is a design note, investigation plan, or decision record rather than an implementation handoff, do not force `## Tasks`.

### 6. Self-check

Before presenting, read the artifact as if you were a fresh executor.

Check:

- No unresolved user-facing questions remain.
- Scope, out-of-scope items, and important decisions are captured.
- A fresh executor would know where to start.
- Referenced local files exist.
- Task/checklist structure is present (using `- [ ]` checkbox format)
- Supporting docs are not accidentally written as task contracts.

Fix obvious gaps before presenting.

### 7. Optional cold read

Cold reads are for executor autonomy, not mechanical markdown validation. They test whether someone without the original conversation can act from the artifact.

Run a cold read by default for non-trivial or multi-file plans. It matters most when:

- the work is cross-cutting, architecture-heavy, security-sensitive, destructive, concurrent, persistent, or public-API-facing;
- the authoring conversation had lots of decisions that may not be captured cleanly;
- the executor will need to work autonomously without asking follow-up questions;
- the user asks for a "handoff", "implementation-ready", "thorough", or "autonomous" plan.

Skip cold reads when the plan is small/local, or the latency/cost is disproportionate.

Usually decide whether to run a cold read yourself and report it in the final summary. Ask the user first when the cold read would be materially slow or expensive.

Prompt shape:

> Read the plan at `<path>` as a cold executor. Read-only validation only: do not edit, write, modify files, run tests, or execute the plan. You have no other context about this work. Assume the executor can do normal code exploration and run appropriate verification. Identify any blockers or material gaps that would prevent autonomous implementation. For each gap, include Location, Gap, and Why it matters. If there are no material gaps, say "Ready for autonomous execution, no material gaps".

Act on cold-read feedback with judgment. Re-run at most once after substantive edits; if meaningful gaps remain, surface them to the user instead of looping.

### 8. Present

End with a concise summary, not a full dump unless requested.

Include:

- Plan file path(s) created.
- One-sentence goal of the plan.
- Task count and notable supporting files, if any.
- Known non-blocking risks, assumptions, or follow-up decisions, if present.
- Whether a cold read was run and what changed because of it.

Offer to show or revise specific files and iterate until approved.
