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

Investigation tasks are allowed only when the answer genuinely requires implementation or runtime evidence. Label them explicitly, give concrete acceptance, and make them produce a recorded decision or choose between already-described branches.

If invoked cold, spend extra time here. If invoked after prior discussion, build on it.

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

Propose a default and confirm with the user. Detection order:

1. If the repo already has `plans/`, `docs/plans/`, `tmp/plans` or similar suggest that location
2. Otherwise: single file → `PLAN.md` (or `plan-<slug>.md` if `PLAN.md` exists) at repo root; multi-file → `plans/<slug>/` with the index inside

Before writing, check whether every target file or directory already exists. Never overwrite an existing plan artifact unless the user explicitly confirms replacement.

Always confirm before writing. Never write to an unconfirmed path.

### 5. Write

Use the templates below.

**Task sizing**: Each task is atomic — one meaningful, individually verifiable unit of work. Acceptance is observable without human judgment.

The executable task list must always live under a literal `## Tasks` heading. Each top-level unchecked checkbox in that section is an executable task; put plan links or other metadata as indented non-checkbox lines under the task.

**Always consider** Out of Scope and Decisions: think through whether they apply, then write the section only if non-empty.

**Quality bar**:

- Implementation-agnostic where possible: specify WHAT, not HOW. Implementation Notes is for breadcrumbs, not prescriptions.
  - Good: `POST /widgets` with missing `name` returns 400 with `errors[0].field === 'name'`
  - Bad: Add input validation to the widgets endpoint using zod
- Every acceptance criterion observable. Include verification approach (tests, manual checks) inline with the criterion.
- No ambiguity about what "done" means.
- Reads cold: a fresh executor should not need to ask the original requester anything.

### 6. Validate

**Always** self-review the artifact. Read it through, fix obvious gaps.

For non-trivial plans, also delegate cold-read validation to a fresh agent. If available, use an Implement agent because it best approximates the eventual executor; otherwise use a general purpose subagent.

Prompt shape:

> Read the plan at `<path>`. Read-only validation only: do not edit, write, modify files, run commands, run tests, or execute the plan. You have no other context about this work. For each task: describe what you'd do, and list anything you can't determine from the plan alone. Report gaps in under 300 words.

Use the gap list to patch the plan. Re-run validation at most twice; if substantive gaps remain, surface them to the user instead of looping further.

Self-review is hygiene; cold-read by a fresh agent is the only real check that the artifact reads cold.

### 7. Present

End with a concise summary message, not a full dump unless requested.

Include:

- Plan file path(s) created
- One-sentence goal of the plan
- Task count and notable supporting files
- Any unresolved risks or user decisions, if present
- Whether cold-read validation was run and what changed because of it

Offer to show or revise specific files and iterate until approved.

## Templates

### Single file

```markdown
# {Title}

## Context

Why this is needed. Background. How it fits.

## Acceptance Criteria

- [ ] Observable, testable criterion (include verification — tests, manual checks, etc.)
- [ ] ...

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

## Guidelines

- **Resolve, don't defer** — every question gets answered in Clarify. Only implementation/runtime unknowns may become labeled investigation tasks with concrete acceptance and a recorded decision output.
- **Verify before claiming absence** — never assume something isn't implemented; check.
- **Always consider Out of Scope and Decisions** — write the sections only if non-empty, but always think through whether they apply.
- **Plan files encode decisions** — everything the executor needs lives in the artifact.
- **The index is for ordering** — task ordering by dependency lives in the index; supporting files are the source of truth for context.
