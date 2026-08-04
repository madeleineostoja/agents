## General rules

- Match the surrounding code’s comment density and style. Add comments when they explain a non-obvious constraint, invariant, workaround, or design tradeoff; do not narrate straightforward code.
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

- Tests must verify behaviour, not restate implementation. Before adding a test, be able to identify the real bug or regression it would catch; if you can't, skip it.
- Test public contracts and observable outcomes, not internals; skip trivial formatters, stdlib wrappers, and pass-through helpers unless they encode important behaviour. Assert mock/fake internals only when that interaction is itself the contract.
- Skip exhaustive parameter sweeps across one code path — pick representative edge cases and one happy path.
- Prioritise (in order): error/rejection paths, state invariants, data integrity, cross-component contracts, safety/security boundaries, and non-trivial heuristics.
- When touching tests, consolidate or remove nearby low-value tests instead of adding more noise.

## Code conventions

- In TypeScript projects, prefer `type` over `interface` unless features of interfaces (eg: declaration merging) are strictly required
