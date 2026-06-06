---
name: plan
description: Produce a written plan artifact that a separate executor can implement without further clarification. Use only when the user explicitly asks to plan something (e.g. "/plan", "plan this", "spec this out"), or after confirming with the user that planning is the right next step rather than execution.
---

# Plan

Produce a written plan that a fresh session can pick up and execute without re-deriving any of the reasoning.

**A plan is written for one executor to consume once.** Don't design for updates, versioning, or maintenance. The plan exists purely to transfer thinking from one context to another.

The artifact is the point. Do the heavy work here — exploration, edge cases, decisions, tradeoffs — so the executor reads cold and acts.

## Process

### 1. Clarify

Resolve every open requirement, scope boundary, edge case, and tradeoff in conversation. Ask the user directly in prose for any gaps. **Do not finish the skill with unresolved questions**.

If invoked cold, spend extra time here. If invoked after prior discussion, build on it.

**Investigation tasks are the only allowed escape hatch.** Use them only when the answer genuinely requires implementation or runtime evidence — never as a way to defer a decision the user could make in conversation. Label them explicitly, give concrete acceptance, and make them produce a recorded decision or choose between already-described branches.

### 2. Explore

Explore enough to know existing patterns, integration points, and that the thing you're about to plan doesn't already exist. Never assume functionality is absent — check. Stop when more exploration won't change the plan.

The output is informal: notes, file pointers, identified patterns. You'll use these when writing.

### 3. Choose shape

Two shapes:

- **Single file** — one plan file with context, acceptance, file pointers, and a checkbox task list inline. For cohesive, contained work.
- **Index + supporting plans** — an index plan file owns the checkbox task list. Each task references a supporting plan file (or shares one with siblings). Supporting files carry the detailed context, acceptance criteria, decisions, and implementation notes.

Use supporting plan files when **any** of these apply:

- Shared context across multiple tasks that would otherwise be duplicated
- Non-obvious requirements or edge cases worth documenting independent of any single task
- New domain concepts or abstractions that need defining once
- Rich acceptance criteria that don't reduce to "code does X"
- Cross-cutting decisions (data model, API contract, naming) multiple tasks must respect
- Independently implementable sub-topics — each gets its own supporting file

Number of tasks alone is a bad signal. If genuinely ambiguous, ask the user.

### 4. Choose location

- Detect: if the repo already has `plans/`, `docs/plans/`, `tmp/plans/` or similar, use it. Otherwise default to `plan-<slug>.md` for single-file, or `plans/<slug>/` for multi-file.
- Confirm the path with the user before writing. Never write to an unconfirmed path.
- Never overwrite an existing plan artifact without explicit confirmation.

### 5. Write

Use the templates below.

**Task sizing**: Each task is atomic — one meaningful, individually verifiable unit of work. Acceptance is observable without human judgment.

**`## Tasks` is mandatory**: The single-file plan, or the index file for multi-file, must contain a literal `## Tasks` heading with at least one top-level unchecked checkbox (`- [ ] ...`). Each top-level checkbox is one executable task; put plan links and other metadata as indented non-checkbox lines beneath it.

**Out of Scope and Decisions**: think through whether they apply, then write the section only if non-empty.

**Guidelines:**

- Specify contracts and decisions over code structure. Encode what's observable from outside the module (API shape, error semantics, data model, chosen libraries when the team has standardized) and decisions the implementer can't re-derive
  - Good example: `POST /widgets` with missing `name` returns 400 with `errors[0].field === 'name'`. Validation uses the project's existing zod schemas
  - Bad example: In `widgets.ts`, add a `validateWidget(input)` function using `z.object({...})` and call it at the top of the POST handler
- Every acceptance criterion is observable. Include verification approach (tests, manual checks) inline with the criterion.
- Reads cold: a fresh executor should not need to ask the original requester anything.

### 6. Validate

Validate in two phases.

**1. Structural**

- The `## Tasks` heading exists in the right file (single-file plan, or index file for multi-file).
- At least one top-level unchecked checkbox task: `- [ ] ...`.
- Executable tasks appear only under `## Tasks`.

Fix any failure before presenting.

**2. Semantic**
2a. Self-review: read the artifact through and fix obvious gaps.

2b. For non-trivial plans, delegate cold-read validation to a fresh agent. Prefer an Implement agent (best approximates the eventual executor) if available; otherwise use general-purpose. Ensure you set a high enough max turn limit on any subagents to actually assess the plans.

Prompt shape:

> Read the plan at `<path>` as a cold implementer. Read-only validation only: do not edit, write, modify files, run tests, or execute the plan. You have no other context about this work. Assume the implementer can do normal code exploration and run appropriate verification. For each task, determine whether there would be any blockers or material gaps for implementation. For each gap, include the Task, Gap, and Why it matters. If there are no gaps, say "Ready to implement, no material gaps".

Action reviewer comments with discretion. Re-run validation at most twice; if substantive gaps remain, surface them to the user instead of looping further.

### 7. Present

End with a concise summary message, not a full dump unless requested.

Include:

- Plan file path(s) created
- One-sentence goal of the plan
- Task count and notable supporting files
- Any known non-blocking risks, assumptions, or follow-up decisions, if present
- Whether cold-read validation was run and what changed because of it

Offer to show or revise specific files and iterate until approved.

## Templates

### Single file

```markdown
# {Title}

## Context

Why this is needed. Background. How it fits.

## Acceptance Criteria

- Observable, testable criterion (include verification — tests, manual checks, etc.)
- ...

## Out of Scope

(Include if there are notable exclusions; omit otherwise.)

## Decisions

(Include if non-trivial decisions were made and the reasoning matters to the executor; omit otherwise.)

## Implementation Notes

File pointers, relevant patterns. Breadcrumbs, not instructions.

## Tasks

- [ ] Task one
- [ ] Task two
- [ ] ...
```

### Index + supporting

Index file:

```markdown
# {Title}

One-paragraph summary of the overall work and goal.

## Out of Scope

(Include if there are notable exclusions; omit otherwise.)

## Tasks

- [ ] Task one
  - Plan: `<supporting-file>.md`
- [ ] Task two
  - Plan: `<supporting-file>.md`
- [ ] ...
```

Tasks in the index are ordered by dependency, then priority. Each task references the supporting plan file(s) that carry its context. When a supporting file is shared by multiple tasks, structure it so each task's relevant acceptance criteria are obvious without duplicating them in the index.

Each supporting plan file uses the single-file template minus the `## Tasks` section (tasks live in the index).
