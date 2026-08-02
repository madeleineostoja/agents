---
name: plan
description: Produce a written implementation handoff plan that a future executor can use without relying on the original conversation. Use only when the user explicitly asks to plan something (e.g. "/plan", "plan this", "spec this out"), or after confirming that planning is the right next step rather than execution.
---

# Plan

Produce a human-readable plan artifact that captures the goal, constraints, decisions, tradeoffs, and work breakdown well enough for a future executor — human, agent, or automation — to act autonomously without re-deriving the original conversation.

A fresh executor has the repository and completed plan artifacts, but not the original conversation or approval exchange. They may inspect source code and choose ordinary implementation details. They must not need to reconstruct user intent, reopen approved product or architecture decisions, infer material scope boundaries, or repeat the planning process.

The artifact is the point. Do the heavy thinking here: clarify ambiguity, investigate the existing system, record decisions, identify edge cases, and write a decision-complete handoff.

A plan is a handoff document, not a permanent specification system. Avoid unnecessary process, taxonomy, or maintenance structure unless the work actually needs it.

Detailed plans serve executors, but users should not have to inspect all executor-facing detail to govern the outcome. Before writing the artifact, present a concise approval digest of the decisions it will encode. The digest is a conversational approval surface, not a separate permanent artifact; carry its approved content into the plan's human-readable entry point.

## Authoring process

### 1. Clarify

Resolve open requirements, scope boundaries, edge cases, and tradeoffs in conversation. Ask the user directly in prose for gaps that matter. **Do not finish the plan with unresolved user-facing questions.**

Clarification should resolve ambiguity in the current outcome, not expand it. Do not promote plausible future needs or theoretical edge cases into scope. A possibility becomes a requirement only when grounded in the user's stated goal, existing behaviour or API contract, repository architecture or convention, explicit acceptance criteria, a real trust boundary, a demonstrated failure, or a material correctness, safety, data-integrity, or operational risk.

If invoked without prior discussion, spend extra time here. If invoked after prior discussion, preserve the decisions already made instead of reopening them.

**Investigation tasks are the only allowed escape hatch.** Use them only when the answer genuinely requires implementation or runtime evidence — never as a way to defer a decision the user could make in conversation. Label them explicitly, give concrete acceptance, and make them produce a recorded decision or choose between already-described branches.

### 2. Investigate

Investigate enough to understand existing patterns, integration points, constraints, and whether the requested work already exists. Never assume functionality is absent — check. Use the normal discovery tools and delegation guidance; this phase does not require an Explore subagent.

When evaluating an implementation area, identify relevant existing project implementations, framework or platform capabilities, standard-library facilities, and installed dependencies. Record only capabilities and options that materially affect requirements, architecture, task boundaries, or the solution class.

Investigation discovers options and constraints; it does not create requirements. Stop when further discovery is unlikely to change requirements, architecture, task boundaries, or the solution class.

The output can be informal while investigating: notes, file pointers, discovered patterns, and constraints. Use these when writing the artifact.

### 3. Prepare an approval digest

Before writing or finalizing the detailed plan, prepare a concise, decision-complete, implementation-light summary of what the plan will encode. A substantial digest should normally fit within roughly 300–800 words. For small, local work, a short paragraph covering the outcome, scope, and non-goals is enough.

Use only the sections relevant to the work. Do not force empty or `N/A` sections. Cover material decisions from these areas:

- **Outcome** — what the completed work enables and how success will be recognized.
- **Experience and behaviour** — affected user, developer, or operator flows, states, errors, defaults, recovery, accessibility, and degraded operation.
- **Contract surface** — affected public APIs, internal extension interfaces, schemas, CLI commands, tools, events, configuration, persisted formats, migrations, compatibility commitments, or other relied-upon behaviour.
- **Solution design** — major system boundaries, dependencies, solution-class decisions, new concepts, and expensive-to-reverse choices.
- **Non-goals** — what the work deliberately does not solve.
- **Delivery map** — substantive milestones, dependencies, and proposed sequencing without implementation choreography.
- **Risks and assumptions** — material uncertainty, external dependencies, and facts not yet proven.

#### Propose plan shape

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

#### Propose plan location

- Detect whether the repo already has `plans/`, `docs/plans/`, `tmp/plans/`, or similar. Use existing conventions when present.
- Otherwise default to `plan-<slug>.md` for a single file, or `plans/<slug>/` for multi-file plans.
- Include the proposed shape and base path in the approval digest: the plan file for a single-file plan, or the plan directory for a multi-file plan.
- If writing the plan would overwrite existing files, explicitly identify that approving the digest would authorize it.

### 4. Obtain approval

Present the digest, proposed plan shape, and base path together. Ask the user to approve or correct them before writing any plan files.

Approval covers the outcome, scope, overall solution design, user, developer, and operator experience, relied-upon interfaces, material risks, and delivery shape. Detailed implementation decisions remain with the executor and do not require approval.

When prior conversation already contains explicit approval of material decisions, summarize them rather than reopening them, but still wait for approval of the digest and base path. Never write to an unconfirmed base path or overwrite existing plan files without explicit confirmation.

### 5. Write

Use the templates in `templates/` as starting points when helpful. Adapt headings to the work; do not force empty sections.

Treat the approved digest as the boundary for the detailed plan. Preserve its outcome and material decisions in the single plan file or multi-file index so that document remains the human-readable entry point. If further research would materially change the outcome, scope, solution design, user, developer, or operator experience, relied-upon interfaces, material risks, delivery shape, or base path, stop and present a revised digest for approval rather than silently embedding the change.

Keep contracts grounded in current requirements, existing behaviour, established architecture, and material risks. Do not turn plausible future needs or theoretical edge cases into acceptance criteria.

Where solution choice materially affects maintenance surface, record the intended solution class or its decision criteria: reuse existing project code, use a framework or platform capability, use an installed dependency, add a dependency, or implement locally. Require a concrete present-purpose rationale when the plan introduces a new subsystem, abstraction boundary, compatibility layer, configurable mechanism, or substantial custom implementation.

Design each checkbox task as one cohesive change that can be implemented, verified, reviewed, and committed in a focused work cycle. Assuming earlier tasks are complete, it should leave the repository in a coherent state with a meaningful outcome. A task file may add detail, but must not hide an oversized task.

#### Task sizing

Use a logical commit as the default unit of work, not a file, architectural layer, or implementation phase. Prefer a vertical slice containing the code, tests, migrations, and documentation needed for its outcome.

Split a task when any of these apply:

- It delivers multiple outcomes that could be accepted or committed independently.
- It crosses distinct subsystems or contracts that can be changed and verified separately.
- It combines preparatory refactoring, migration, or infrastructure work with behavior that does not need to land atomically.
- Its acceptance criteria form several unrelated clusters, or reviewing it requires switching between materially different concerns, risks, or rollout strategies.
- A capable executor is unlikely to implement and review it in one focused work cycle without losing context.

Do not create a separate checkbox for a tiny edit, helper, import update, routine test coverage, validation run, git operation, or other mechanical step. Fold these into the task they support. Tests, documentation, dependency changes, migrations, or investigation are standalone tasks only when they produce a substantive independently reviewable outcome of their own.

Keep a larger task intact only when splitting it would create an invalid intermediate state or obscure a single atomic contract change. State that coupling explicitly in the task so the executor and reviewer understand why it cannot be divided.

#### Rules for plans:

- Include an explicit checklist section for tasks; tasks MUST use the `- [ ]` markdown checkbox format.
- Include a heading for the task list, eg: `## Tasks`.
- Do not divide or group the task list with subheadings.
- Tasks can have sub-points as nested bullets indented below the `- [ ]` task.
- Do not use checkboxes (`- [ ]`) for non-task sections, eg: Acceptance Criteria.
- Order tasks by dependency, then priority.

#### Guidance for plans:

- Record observable acceptance criteria where they help autonomy.
- Include verification guidance where the executor would otherwise have to guess. Keep it proportional to changed behaviour and material risk; do not prescribe exhaustive matrices unless distinct failure modes or their impact justify them.
- Capture contracts and decisions over code choreography.
  - Good: `POST /widgets` with missing `name` returns 400 with `errors[0].field === "name"`. Validation uses the project's existing schema pattern.
  - Bad: In `widgets.ts`, add a `validateWidget(input)` function and call it at the top of the handler.
- State implementation constraints only when required by the contract, existing architecture, or material risk. Leave the executor free to choose a simpler implementation that satisfies the same observable contract.
- Keep source pointers as breadcrumbs, not step-by-step implementation instructions.

### 6. Self-check

Before presenting, perform one artifact-bounded handoff pass. Start from the plan's human-readable entry point and read all completed plan artifacts as a fresh executor. Test whether they transfer the approved direction; do not restart repository investigation, reconsider approved decisions, or develop an alternative plan. Follow a source reference only when needed to resolve a concrete concern raised by the artifact.

Check:

- No unresolved user-facing questions remain.
- The plan matches the approved digest, and its human-readable entry point preserves the approved outcome and material decisions.
- No approved scope boundary, experience or behaviour, relied-upon interface, solution-design decision, non-goal, material risk, or delivery shape was silently changed after approval.
- Scope, out-of-scope items, and important decisions are captured.
- Every requirement and acceptance criterion is grounded in the current requirements, an existing repository contract, or a material risk.
- No task or planned mechanism exists solely for speculative flexibility, compatibility, or configuration.
- Every planned subsystem, abstraction boundary, or substantial custom implementation has a concrete present purpose and rationale.
- Existing project and ecosystem capabilities were considered where discovery was warranted.
- Verification guidance is proportional to changed behaviour and material risk.
- The plan does not force unnecessary machinery, and a simpler implementation remains valid when it satisfies the same contract and constraints.
- A fresh executor would know where to start.
- Referenced local files exist.
- Task/checklist structure is present (using `- [ ]` checkbox format) and every task is a deliverable.
- Each task fits one focused implementation/review cycle and logical commit, unless an explicitly documented atomicity constraint prevents splitting it.
- No task is merely a mechanical, validation-only, or bookkeeping step.
- Supporting docs are not accidentally written as task contracts.

Fix obvious transfer gaps that remain within the approved digest. If a fix would materially change the approved outcome, scope, solution design, experience, contract surface, risks, delivery shape, or base path, stop and present a revised digest for approval. Do not begin another general review cycle after making corrections.

### 7. Run risk-triggered independent review

Do not delegate a cold read by default. Run one independent Review only when at least one of these applies:

- The plan changes an existing public, persisted, or compatibility-sensitive contract.
- It includes destructive or difficult-to-reverse migration, rollout, or cutover work.
- It crosses an authorization, security, privacy, or other trust boundary.
- It coordinates independently delivered workstreams whose sequencing or shared contracts could leave the system inconsistent.
- The user explicitly requests independent review.

Plan length, task count, a multi-file shape, or the number of implementation files do not independently trigger review.

Use a Review subagent, never Explore. Give it the completed plan path or paths, but not the original conversation, approval exchange, or a separate digest. The review is plan-led and source-verified:

1. Read all supplied plan artifacts before inspecting source.
2. Assess handoff completeness, internal consistency, task dependencies, contract coverage, and whether material decisions are improperly left to the executor.
3. Perform only minimal, targeted source checks needed to verify a concrete claim or resolve a specific handoff concern raised by the plan. Every source lookup must answer an already-identified review question. If that would require broad subsystem exploration, report the uncertainty instead of starting another investigation phase.
4. Do not reconsider approved product or architecture decisions, search for alternative solutions, expand scope, rewrite the plan, or produce a replacement plan.
5. Return either `Ready` or a concise list of concrete blockers. Tie each blocker to an affected plan location and include any source evidence used.

Apply valid transfer fixes once when they remain within the approved digest. If review exposes a required material change, present a revised digest for user approval. Do not send the corrected artifact through another independent review cycle.

### 8. Present

End with a concise summary, not a full dump unless requested.

Include:

- Plan file path(s) created.
- The approved outcome.
- Task count and notable supporting files, if any.
- Known non-blocking risks, assumptions, or follow-up decisions, if present.

Offer to show or revise specific files if requested.
