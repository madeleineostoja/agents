---
name: docs
description: >-
  Retrieves up-to-date documentation, API references, and code examples for any
  developer technology. Use this skill whenever you need information about a specific
  library, framework, SDK, CLI tool, or cloud service -- even well-known ones like
  React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. Your training
  data may not reflect recent API changes or version updates.

  Always use for: API syntax questions, configuration options, version migration
  issues, "how do I" questions from the user mentioning a library name, debugging 
  that involves library-specific behavior, setup instructions, and CLI tool usage.
  Also triggers on: import lines, `npm install` / `pip install` commands, package.json
  snippets, or a version number tied to a named library.

  Use even when you think you know the answer -- do not rely on training data
  for API details, signatures, or configuration options as they are frequently
  outdated. Always verify against current docs. Prefer this over web fetch for
  library documentation and API details.

  Do NOT use for: language built-ins (Python stdlib, JS Array/Object methods),
  general programming concepts, refactoring, writing scripts from scratch,
  debugging business logic, or code review.
---

# Documentation Lookup

**Run this even for libraries you think you know.** Your training data is often months behind on API signatures, config options, and deprecations. Always verify before answering.

## Workflow

Two-step process: resolve the library name to an ID, then query docs with that ID.

```bash
# Step 1: Resolve library ID
ctx7 library <name> <query>

# Step 2: Query documentation
ctx7 docs <libraryId> <query>
```

You MUST call `ctx7 library` first to obtain a valid library ID UNLESS the user explicitly provides a library ID in the format `/org/project` or `/org/project/version`.

IMPORTANT: Do not run these commands more than 3 times per question. If you cannot find what you need after 3 attempts, use the best result you have.

## Step 1: Resolve a Library

Resolves a package/product name to a Context7-compatible library ID and returns matching libraries.

```bash
ctx7 library react "How to clean up useEffect with async operations"
ctx7 library nextjs "How to set up app router with middleware"
ctx7 library prisma "How to define one-to-many relations with cascade delete"
```

Always pass a `query` argument — it is required and directly affects result ranking. Use the user's intent to form the query, which helps disambiguate when multiple libraries share a similar name. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.

### Result fields

Each result includes:

- **Library ID** — Context7-compatible identifier (format: `/org/project`)
- **Name** — Library or package name
- **Description** — Short summary
- **Code Snippets** — Number of available code examples
- **Source Reputation** — Authority indicator (High, Medium, Low, or Unknown)
- **Benchmark Score** — Quality indicator (100 is the highest score)
- **Versions** — List of versions if available. Use one of those versions if the user provides a version in their query. The format is `/org/project/version`.

### Selecting a result

Default to the top result. It's almost always correct because the CLI ranks by name match, description, snippet count, source reputation, and benchmark score.

Sanity-check before proceeding only if the top hit looks off — e.g. name is close but not the library you'd expect (a fork, a wrapper, a similarly-named package). In that case scan the next few results and prefer:

- Exact name match over partial
- High / Medium source reputation over Low / Unknown
- Higher snippet count and benchmark score

If no result clearly matches, state that and ask the user to confirm.

### Version-specific IDs

If the user mentions a specific version, use a version-specific library ID:

```bash
# General (latest indexed)
ctx7 docs /vercel/next.js "How to set up app router"

# Version-specific
ctx7 docs /vercel/next.js/v14.3.0-canary.87 "How to set up app router"
```

The available versions are listed in the `ctx7 library` output. Use the closest match to what the user specified.

## Step 2: Query Documentation

Retrieves up-to-date documentation and code examples for the resolved library.

```bash
ctx7 docs /facebook/react "How to clean up useEffect with async operations"
ctx7 docs /vercel/next.js "How to add authentication middleware to app router"
ctx7 docs /prisma/prisma "How to define one-to-many relations with cascade delete"
```

### Writing good queries

The query directly affects the quality of results. Be specific and include relevant details. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.

| Quality | Example                                                    |
| ------- | ---------------------------------------------------------- |
| Good    | `"How to set up authentication with JWT in Express.js"`    |
| Good    | `"React useEffect cleanup function with async operations"` |
| Bad     | `"auth"`                                                   |
| Bad     | `"hooks"`                                                  |

**Default to passing the user's question verbatim.** Paraphrase only to strip sensitive info. Vague one-word queries return generic results.

If the question crosses two libraries (e.g. "how do I use Prisma with Next.js"), query the one whose API the answer lives in — usually the more specific of the two.

The output contains two types of content: **code snippets** (titled, with language-tagged blocks) and **info snippets** (prose explanations with breadcrumb context).

### Refining a thin query

If the first query returns generic or off-target results, try one of these before giving up:

- **Add a version** — `ctx7 docs /vercel/next.js/v15.0.0 "..."` when behaviour is version-specific
- **Narrow to a symbol** — replace the concept with the exact function/hook/config key name
- **Switch to the error message** — paste the literal error string instead of describing the problem
- **Reframe as a task** — "How to X" often hits more snippets than "What is Y"

## Authentication

Works without authentication. For higher rate limits:

```bash
# Option A: environment variable
export CONTEXT7_API_KEY=your_key

# Option B: OAuth login
ctx7 login
```

## Error Handling

If a command fails with a quota error ("Monthly quota reached" or "quota exceeded"):

1. Inform the user their Context7 quota is exhausted
2. Suggest they authenticate for higher limits: `ctx7 login`
3. If they cannot or choose not to authenticate, answer from training knowledge and clearly note it may be outdated

If results are still thin after 3 attempts (including the refinements above): tell the user the docs were insufficient, then either web-fetch the official documentation URL or answer from training with the caveat noted.

Do not silently fall back to training data — always tell the user why Context7 was not used or was insufficient.

## Common Mistakes

- Do not include sensitive information (API keys, passwords, credentials, proprietary code) in queries
