## General rules

- Match the surrounding code’s comment density and style. Add comments when they explain a non-obvious constraint, invariant, workaround, or design tradeoff; do not narrate straightforward code.
- Keep all documentation and code comments stateless: describe behaviour, constraints, and rationale, never point-in-time history or status (PRs, issues, rollout progress, or “recently changed” narratives). Keep change history in version control.
- State the intended change before editing when scope, authorization, or the requested outcome is ambiguous

## Tools

The following non-standard CLI tools are available; prefer them over the defaults:

- Prefer `rg` instead of `grep` (example: `rg "pattern"` instead of `grep -r "pattern"`)
- Prefer `fd` instead of `find` (example: `fd "filename"` instead of `find . -name "filename"`)
- Infer the package manager from the project lockfile; if no lockfile exists, fall back to system defaults (eg: npm)

## Git commits

- NEVER force-add or commit a file ignored by Git, including files in ignored parent folders
- Follow a lightweight Conventional Commit pattern for commit messages: `<type>: <brief description>`
- Add a scope (ie: `<type>(scope):`) ONLY if it adds meaningful clarity, eg: when the commit targets one package in a monorepo of many packages
- Add a body (ie: after a blank line) only if there is non-obvious context not inferable from the main message

## Engineering economy

- Prefer coherent changes that satisfy current requirements and fits the repository's existing architecture
- Before adding a non-trivial custom mechanism, inspect relevant nearby code and check for an existing project implementation, framework or platform capability, standard-library facility, or suitable installed dependency; reuse one when it materially satisfies the requirements and local constraints
- Do not add speculative flexibility, compatibility, configuration, abstractions, or edge-case handling without a current requirement, established repository contract, real trust boundary, or other material demonstrated risk
- A mature dependency is acceptable when it removes a meaningful maintenance domain; prefer local code when the required semantics are small and the dependency burden would be disproportionate
- Keep verification proportional to changed behaviour and material risk, following the repository's existing testing and validation conventions

## Testing

- Do not add tests merely because code changed. Default to existing coverage and validation; add a test only for a meaningful regression they would miss. Be able to name that regression before writing the test.
- Prioritise material risks: data loss, broken invariants, security boundaries, failure handling, and cross-component contracts. Skip trivial wrappers, straightforward transformations, and speculative edge cases unless they carry such risk.
- Test observable behaviour, not implementation details. Use representative cases for distinct failure modes; avoid redundant coverage, exhaustive permutations, and broad snapshots of incidental structure.
- Count fixtures, mocks, helpers, and validation scripts as part of the test’s maintenance cost. Use minimal valid inputs and existing infrastructure. If a test needs elaborate scaffolding, seek a simpler test boundary; do not build a testing subsystem for a small change.
- Prefer strengthening or consolidating an existing test over adding another. Keep cleanup local, and stop when the meaningful risk is covered.

## Tool orchestration

- Prefer a direct tool call for a single operation when the tool is directly available.
- Use codemode to batch independent calls, chain dependent calls, process results before returning them, or access tools available only through codemode.
- Give codemode batches an explicit worst-case deadline using `timeout_ms` in the leading `// @options:` JSON comment to avoid indefinite stalls; no deadline is set by default. Size it for the actual work: the slowest parallel call or the combined sequential steps, plus headroom for retries and overhead. Deadline expiry cancels pending calls but does not undo completed effects.
- Do not wrap a single call in codemode merely to print its unchanged result.
- Multiple shell commands inside one Bash invocation still constitute one tool call; they do not by themselves justify codemode.

## Code conventions

- In TypeScript projects, prefer `type` over `interface` unless features of interfaces (eg: declaration merging) are strictly required
