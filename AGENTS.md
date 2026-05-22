## Rules

- Default to writing no code comments. Only add one when the "why" is non-obvious — a hidden constraint, subtle invariant, bug workaround, or surprising behaviour. Don't restate what the code does, don't label sections, don't annotate every change
- Always confirm implementation plans with the user before writing any code or requesting changes.

## Tool Preferences

The following non-standard CLI tools are available for you to use:

- Use `rg` instead of `grep` (example: `rg "pattern"` instead of `grep -r "pattern"`)
- Use `fd` instead of `find` (example: `fd "filename"` instead of `find . -name "filename"`)

## Commit preferences

Follow a lightweight Conventional Commit pattern for commit messages: `<type>: <brief description>`. Do NOT include a scope (no `(scope)` parenthesis). Add a body (after a blank line) ONLY if there is non-obvious context worth recording.
