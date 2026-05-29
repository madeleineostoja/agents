## Rules

- Default to writing no code comments. Only add one when the "why" is non-obvious — a hidden constraint, subtle invariant, bug workaround, or surprising behaviour. Don't restate what the code does, don't label sections, don't annotate every change

## Tool Preferences

The following non-standard CLI tools are available for you to use:

- Use `rg` instead of `grep` (example: `rg "pattern"` instead of `grep -r "pattern"`)
- Use `fd` instead of `find` (example: `fd "filename"` instead of `find . -name "filename"`)

## Commit preferences

Follow a lightweight Conventional Commit pattern for commit messages: `<type>: <brief description>`. Avoid including a scope (ie: no `(scope)` parenthesis) unless it adds meaningful clarity (eg: in a monorepo of many packages). Add a body (after a blank line) ONLY if there is non-obvious context worth recording.

## Subagents

Use Explore subagents to keep main context lean during non-trivial discovery of either a codebase or external resources (eg: Github repos, web fetch). Eg: when relevant files are unclear, patterns span multiple files, or debugging requires tracing across modules. Skip for small/local edits or obvious files.

Use general-purpose implementation subagents only for well-scoped parallel work with clear boundaries;

Keep subagent outputs concise: files, patterns, change points, tests, risks. Default to one subagent unless areas are independent.

## Code convention preferences

- Prefer `type` over `interface` in Typescript
