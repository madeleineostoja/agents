## Rules

- Default to writing no code comments. Only add one when the "why" is non-obvious — a hidden constraint, subtle invariant, bug workaround, or surprising behaviour. Don't restate what the code does, don't label sections, don't annotate every change
- Before non-trivial or multi-file edits, summarize intended changes and ask for confirmation unless the user clearly asked you to implement, fix, or modify files

## Tool preferences

The following non-standard CLI tools are available for you to use:

- Prefer `rg` instead of `grep` (example: `rg "pattern"` instead of `grep -r "pattern"`)
- Prefer `fd` instead of `find` (example: `fd "filename"` instead of `find . -name "filename"`)
- Infer the package manager from the project lockfile; if no lockfile exists, fall back to system defaults (eg: npm)

## Commit preferences

Follow a lightweight Conventional Commit pattern for commit messages: `<type>: <brief description>`. Avoid including a scope (ie: no `(scope)` parenthesis) unless it adds meaningful clarity (eg: in a monorepo of many packages). Add a body (after a blank line) ONLY if there is non-obvious context worth recording.

## Testing preferences

- Tests must verify behaviour, not restate implementation. Before adding a test, be able to identify the real bug or regression it would catch; if you can't, skip it.
- Test public contracts and observable outcomes, not internals; skip trivial formatters, stdlib wrappers, and pass-through helpers unless they encode important behaviour. Assert mock/fake internals only when that interaction is itself the contract.
- Skip exhaustive parameter sweeps across one code path — pick representative edge cases and one happy path.
- Prioritise (in order): error/rejection paths, state invariants, data integrity, cross-component contracts, safety/security boundaries, and non-trivial heuristics.
- When touching tests, consolidate or remove nearby low-value tests instead of adding more noise.

## Code convention preferences

- In TypeScript projects, prefer `type` over `interface` unless features of interfaces (eg: declaration merging) are strictly required

## Subagents

Use Explore subagents to keep main context lean during non-trivial discovery of either a codebase or external resources (eg: GitHub repos, web fetch). Eg: when relevant files are unclear, patterns span multiple files, or debugging requires tracing across modules. Skip for small/local edits or obvious files.

Use general-purpose implementation subagents only for well-scoped parallel work with clear boundaries.
