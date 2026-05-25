---
name: sentry
description: Use the Sentry CLI (`sentry`) for all read-only Sentry work — viewing issues, events, traces, spans, logs, replays, releases, projects, dashboards, and exploring the Sentry API. ALWAYS prefer `sentry` over web fetching, the Sentry UI, or raw `curl` calls against the API. Trigger when the user references a Sentry issue short ID (e.g. `PROJ-123`, `WEB-G5`), names a Sentry org/project, pastes a `sentry.io` URL, asks about errors / exceptions / performance regressions / traces / spans / logs, or wants AI root-cause analysis ("explain", "Seer", "fix plan"). The CLI auto-detects org/project from `.sentryclirc`, `.env`, and source DSNs. NEVER use for mutations.
---

# Using the sentry CLI

`sentry` is the canonical interface to Sentry — it's explicitly built for AI agents. Prefer it over web fetches, the Sentry UI, or hand-rolled `curl` calls. It handles auth, org/project detection, and pagination for you.

## When to reach for sentry

- The user mentions a Sentry issue short ID (`PROJ-123`, `WEB-G5`) — go straight to `sentry issue view`
- The user asks about errors, exceptions, performance regressions, replays, or releases
- The user names a Sentry org/project or pastes a `sentry.io` URL
- Investigating traces, spans, or logs
- Want AI root-cause analysis or a fix plan (Seer)
- Need to hit a Sentry API endpoint — try `sentry api` before `curl`

If the answer might live in Sentry, try `sentry` before grepping further or asking the user.

## Key principles

- **Just run the command.** The CLI auto-detects org/project from `.sentryclirc`, scans for DSNs in `.env` / source, and falls back to directory names. Don't pre-resolve org/project or pre-authenticate — the CLI prompts when auth is missing.
- **Prefer dedicated commands over `sentry api`.** `sentry issue view`, `sentry trace view`, `sentry log list` etc. before constructing raw API calls. Reach for `sentry api` only when no subcommand covers the case.
- **Use `sentry schema` to explore the API.** Faster and more accurate than fetching OpenAPI specs externally.
- **`gh`-style conventions throughout.** `<noun> <verb>` commands, `--json` / `--fields` for machine output, `-w` / `--web` to open in browser, `-q` / `--query` for search, `-n` / `--limit` to cap results, `-t` / `--period` for time windows.

## Prerequisites

Assume `sentry` is already installed and authenticated (`sentry auth status` to verify). If it fails with auth errors, tell the user — do not run `sentry auth login` for them.

If the `sentry` binary isn't on PATH, substitute `npx sentry@latest` in every command below (e.g. `npx sentry@latest issue view PROJECT-123`).

```bash
# install (only if missing)
curl https://cli.sentry.dev/install -fsS | bash
# or: brew install getsentry/tools/sentry  |  npm i -g sentry
```

## Output formatting

Default output is human-formatted and verbose. For programmatic consumption:

- `--json` for structured output (pipe to `jq`)
- `--fields <a>,<b>,...` to select specific fields and shrink the payload — run `<cmd> --help` to discover available fields
- `--limit N` (alias `-n`) to cap result count
- `-w` / `--web` on view commands to open in the browser

```bash
sentry issue list --json --fields shortId,title,priority,level,status --limit 10
sentry issue list --query "is:unresolved" --limit 5
```

## Scoping to an org or project

Org and project are positional, `gh`-style — but **most commands auto-detect**. Only pass them when the CLI says it can't detect the target, or it picks the wrong one.

```bash
sentry issue list                          # auto-detect
sentry issue list my-org/my-project        # explicit
sentry span list my-org/my-project/abc123  # trace ID is the trailing positional
```

Time filtering with `--period` / `-t`: `1h`, `24h`, `7d`.

## Issue investigation

The most common workflow. Issue IDs are **short IDs** with a project prefix (`PROJECT-123`), not the numeric ID.

```bash
sentry issue list --query "is:unresolved" --limit 5   # find
sentry issue view PROJECT-123                         # details + events + tags
sentry issue explain PROJECT-123                      # Seer root-cause analysis
sentry issue plan PROJECT-123                         # Seer fix plan
```

`--query` uses **Sentry search syntax** (`is:unresolved`, `assigned:me`, `level:error`, `severity:error`), not free text.

## Traces, spans, logs, events

```bash
sentry trace list --period 1h --limit 5
sentry trace view <trace-id>            # span tree
sentry span list <trace-id>             # spans flat
sentry trace logs <trace-id>            # logs for a trace

sentry log list --follow                # stream logs in real time
sentry log list --query "severity:error"

sentry event view <event-id>            # raw event payload
```

## API access and schema discovery

When no dedicated command covers the case, `sentry api` mimics `curl` — auth handled for you.

```bash
sentry schema                                                  # browse all resources
sentry schema issues                                           # search
sentry schema "GET /api/0/organizations/{organization_id_or_slug}/issues/"

sentry api /api/0/organizations/my-org/                        # GET (default)
```

## Common mistakes

- **Wrong issue ID format.** Use `PROJECT-123` (short ID with project prefix), not the numeric ID `123456789`.
- **Pre-authenticating unnecessarily.** Don't run `sentry auth login` before each command — the CLI prompts when needed. Only re-auth if switching accounts.
- **Missing `--json` for piping.** Human output includes formatting that breaks downstream parsing.
- **Specifying org/project when not needed.** Let auto-detection work first. Only override if the CLI reports it can't detect the target or picks the wrong one.
- **Treating `--query` as free text.** It's Sentry search syntax (`is:`, `assigned:`, `level:`).
- **Fetching OpenAPI specs externally.** Use `sentry schema` + `sentry api` — they handle auth and endpoint resolution.
- **Reaching for `sentry api` when a CLI command would do.** `sentry issue list --json` already exposes `shortId`, `title`, `level`, `priority`, `status`, `permalink` etc. via `--fields`. Only fall back to `sentry api` for data the CLI doesn't surface.

## Exit codes

The CLI uses semantic exit code ranges:

| Range | Meaning | Action |
|-------|---------|--------|
| 0 | success | proceed |
| 10–19 | auth error | tell the user to run `sentry auth login` |
| 20–29 | input error | check command args and retry |
| 30–39 | API error | retry or report |
| 40–49 | feature unavailable | inform the user (plan / setting) |
| 50–59 | operation error | report |
| 60–69 | command-specific | check stderr |

## Forbidden mutations

This skill is read-only. **Never** invoke any `sentry` command that writes to Sentry. The full list of forbidden subcommand groups and verbs:

- `sentry auth login`, `sentry auth logout` — don't manage credentials for the user
- `sentry init` — writes config files into the project
- `sentry org` — any subcommand that creates / edits / deletes an org
- `sentry project create`, `sentry project edit`, `sentry project delete`
- `sentry team create`, `sentry team edit`, `sentry team delete`
- `sentry issue assign`, `sentry issue resolve`, `sentry issue ignore`, `sentry issue update`, `sentry issue delete`, `sentry issue merge`, `sentry issue bookmark`
- `sentry release create`, `sentry release set-commits`, `sentry release finalize`, `sentry release deploy`, `sentry release delete`, `sentry release files upload`
- `sentry sourcemap upload`, `sentry sourcemap inject` — these modify source files and upload artifacts
- `sentry repo add`, `sentry repo remove`
- `sentry dashboard create`, `sentry dashboard delete`, `sentry dashboard widget add`, `sentry dashboard widget edit`, `sentry dashboard widget delete`
- `sentry trial start`, `sentry trial extend`
- `sentry replay delete`
- `sentry api ... --method POST/PATCH/PUT/DELETE` (any non-GET HTTP method), and `--data` / `--header` flags when paired with a writing method
- Any `sentry` subcommand whose `--help` describes it as creating, deleting, editing, assigning, resolving, ignoring, finalizing, deploying, uploading, injecting, setting, or adding

**If the user asks you to perform a mutation** (e.g. "resolve this issue", "create a release", "delete this project", "upload a sourcemap"): stop and confirm explicitly before running anything.

**If you're unsure whether a command mutates:** run `sentry <command> --help` and read it. Default to not running it.
