---
name: linear
description: Use the Linear CLI (`linear`) for Linear work — reading issues, projects, cycles, milestones, initiatives, labels, documents, and creating issues / comments / projects / PRs from the command line. ALWAYS prefer `linear` over web fetching or the Linear web UI. Trigger when the user references a Linear issue short ID (team prefix + number), pastes a `linear.app` URL, asks about their assigned issues / current cycle / project status, wants to turn an RFC or spec into tickets, or asks to file/comment on issues. The CLI infers the current issue from the git branch via `linear issue id`. Reads and most writes are fine; destructive `delete` / `archive` / `remove` commands and auth changes are forbidden.
---

# Using the linear CLI

`linear` is the canonical interface to Linear from a terminal. Prefer it over web fetches, the Linear web UI, or hand-rolled GraphQL — it handles auth and git-branch ↔ issue mapping for you.

## When to reach for linear

- The user mentions a Linear issue short ID (team prefix + number) — go straight to `linear issue view`
- The user pastes a `linear.app` URL
- The user asks "what am I working on", "my open issues", or about a project / cycle / milestone / initiative
- Turning an RFC, spec, or design doc into a batch of tickets — `linear issue create` is the primary path
- Leaving a comment, updating state, assigning, adding labels, attaching files / links, or filing follow-up issues
- Opening a PR for the current issue — `linear issue pr`
- The current git branch encodes a Linear issue — use `linear issue id` to resolve it

If the answer might be in Linear, try `linear` before grepping further or asking the user.

## Key principles

- **Run the command first.** The CLI infers the branch → issue, reads global auth, and uses the user's default team if one is configured. Supply `--team` / `--sort` only when (a) the user has named the team this session, (b) the issue ID prefix makes the team unambiguous, or (c) the CLI errors asking for them.
- **Prefer dedicated subcommands over `linear api`.** Reach for `linear api` only when no subcommand covers the case.
- **Use `linear issue id` to resolve the current issue from the branch** before asking the user "which issue?".
- **Use file flags for any multi-line markdown.** See [Markdown content](#markdown-content).

## Prerequisites

Assume `linear` is installed and authenticated. If auth fails, tell the user to run `linear auth login` — do not run it for them.

```bash
linear --version
linear auth whoami
```

For multi-workspace users, scope a single command with `--workspace <slug>`:

```bash
linear --workspace acme issue view ENG-123
```

### Asking for team and sort

Most listing and creation commands need `--team <KEY>`; `issue mine` / `issue query` also need `--sort` (unless searching). If `LINEAR_ISSUE_SORT` is exported, `--sort` is optional.

The first time you need either in a session:

1. If the team is obvious from context (issue ID prefix, branch-resolved issue, the user just named it), use it without asking.
2. Otherwise ask — concisely, both at once. Example: "Which team key should I use (e.g. `ENG`)? And sort order — `priority` or `manual`?"
3. Offer `linear team list` if they don't remember the key.
4. Reuse the answers for the rest of the session.

## Branch ↔ issue mapping

`linear issue id` reads the current git branch and returns the matching issue ID. Use it before asking the user. `issue commits` is **jj only** — for git users, use `git log --grep="$(linear issue id)"` instead.

```bash
linear issue id                  # short ID for the current branch
linear issue view                # view the current branch's issue
linear issue url                 # linear.app URL for current issue
linear issue title               # just the title
linear issue describe            # title + "Fixes <ID>" trailer, pipe-friendly
linear issue describe -r         # use "References <ID>" instead of "Fixes <ID>"
```

Pipe `describe` into a commit message:

```bash
git commit -m "$(linear issue describe)"
```

## Output formatting

Default output is human-formatted. For programmatic consumption:

- `-j` / `--json` — structured output. **Available on `issue query` and `issue view`, but NOT on `issue mine`**. Most `list` commands have it (project, label, document, initiative, project-update, initiative-update) — but `team list` and `cycle list` do not.
- `--limit N` — cap results on `mine` / `query` (`0` = unlimited)
- `-w` / `--web` or `-a` / `--app` on view commands to open in browser / Linear.app
- `--no-pager` — supported on `issue view`, `issue mine`, and `issue query`. Other commands reject it.

```bash
linear issue query --team <TEAM> --sort priority --json --limit 10 | jq '.[] | {identifier, title, state}'
```

## Issue investigation

Issue IDs are **short IDs** with a team prefix (e.g. `ENG-123`), not the internal UUID.

```bash
linear issue view <ID>           # full details + comments
linear issue view                # current branch's issue
linear issue view <ID> --json    # JSON
linear issue url <ID>            # linear.app URL
linear issue comment list <ID>   # comments only (supports --json)
```

`linear issue view` includes comments by default — pass `--no-comments` to skip, `--show-resolved-threads` to include resolved ones, `--no-download` to keep remote file URLs instead of downloading attachments.

## Searching and filtering issues

Two complementary commands:

- `linear issue mine` — your assigned issues (defaults to `unstarted` state, human output only)
- `linear issue query` — structured filters across the workspace, supports `--json`

```bash
linear issue mine --team <TEAM> --sort priority
linear issue mine --team <TEAM> --sort priority --state started --state unstarted
linear issue mine --team <TEAM> --sort priority --all-states --limit 0

linear issue query --team <TEAM> --search "rate limit"              # full-text
linear issue query --team <TEAM> --search "leak" --search-comments  # also search comment bodies
linear issue query --team <TEAM> --sort priority --state started --assignee <user>
linear issue query --team <TEAM> --sort priority --project "<project>" --label bug
linear issue query --team <TEAM> --sort priority --cycle active
linear issue query --team <TEAM> --sort priority --updated-after 2025-05-01 --json
linear issue query --team <TEAM> --sort priority --unassigned --state backlog
linear issue query --team <TEAM> --sort priority --include-archived
linear issue query --all-teams --search "<text>"
```

Gotchas:

- **`--search` disables `--sort`** on `query`. Pick one.
- **`mine` / `query` need a sort order when not searching.** Pass `--sort priority` (or `manual`), or export `LINEAR_ISSUE_SORT`.
- **`--milestone` requires `--project`** on both `mine` and `query`.
- **States for filters (`-s` on `mine` / `query`) are a fixed set:** `triage`, `backlog`, `unstarted`, `started`, `completed`, `canceled`. Human-facing names like "In Progress" do **not** work here.
- **States for `issue create -s` / `issue update -s` accept either** — the type name OR the human state name ("In Progress", "In Review"). The CLI resolves both.
- **`mine` has no `--search`** — use `query` for full-text.

## Creating issues (incl. RFC → tickets)

Single issue, simple:

```bash
linear issue create --no-interactive \
  --team <TEAM> \
  --title "Add retry on 429 in API client" \
  --project "<project>" \
  --label bug \
  --priority 2
```

Always pass `--no-interactive` in scripts and loops — otherwise the CLI may prompt and stall.

With markdown body, use a file (see [Markdown content](#markdown-content)):

```bash
desc=$(mktemp -t linear-desc.XXXXXX.md)
cat > "$desc" <<'EOF'
## Context

Pulled from the [Resilience RFC](https://...). Client currently throws on 429.

## Acceptance

- [ ] Respect `Retry-After` header
- [ ] Cap retries at 5
- [ ] Emit a structured log on retry exhaustion
EOF

linear issue create --no-interactive \
  --team <TEAM> \
  --title "Add retry on 429 in API client" \
  --description-file "$desc" \
  --project "<project>" --label bug --priority 2
```

Useful flags on `issue create` / `issue update`:

- `--team <KEY>` — required if no default is configured (create only)
- `--project <name|slug>`, `--milestone <name>` (requires `--project`), `--cycle <name|number|active>`
- `--label <name>` (repeatable), `--state <name|type>`, `--assignee self|<username>`
- `--priority 1..4` (1 = urgent, 4 = low), `--estimate <points>`, `--due-date YYYY-MM-DD`
- `--parent <ID>` for sub-issues (e.g. `ENG-123`)
- `--start` on `create` to immediately transition to started (also creates a branch)
- `--no-use-default-template` on `create` to skip the team's default template

Update an existing issue:

```bash
linear issue update <ID> --state "In Review" --assignee self --label needs-design
linear issue update <ID> --description-file "$desc"
```

### Worked example: RFC → tickets

1. Read the RFC.
2. Draft a list of titles, target team / project / labels / parent, and shared body fragments.
3. **Show the plan to the user and wait for explicit go-ahead** (see [Confirm before bulk operations](#confirm-before-bulk-operations)).
4. Loop:

   ```bash
   for t in "Add retry on 429" "Cap retry count at 5" "Log on retry exhaustion"; do
     linear issue create --no-interactive \
       --team ENG --project "Resilience" --label bug \
       --title "$t" --description-file /tmp/shared-context.md
     sleep 0.5   # gentle throttle for large batches
   done
   ```

For more than ~20 issues, add a short `sleep` between calls to stay under Linear's API rate limits.

### Other safe writes

```bash
linear issue start <ID>                           # transition + create local branch
linear issue start <ID> -b feature/custom-branch  # override branch name
linear issue start <ID> -f main                   # branch from a specific ref

linear issue comment add <ID> --body-file /tmp/comment.md
linear issue comment add <ID> --body-file /tmp/reply.md --parent <comment-id>  # threaded reply
linear issue comment add <ID> --body-file /tmp/c.md --attach screenshot.png    # attach file
linear issue comment update <comment-id> --body-file /tmp/updated.md

linear issue attach <ID> ./path/to/file.png       # upload a FILE
linear issue link <ID> https://example.com        # link a URL (with optional --title)

linear issue relation add <ID> blocks <other-ID>      # types: blocks, blocked-by, related, duplicate
linear issue relation add <ID> blocked-by <other-ID>

linear issue pr                                   # GitHub PR for current branch's issue
linear issue pr <ID> --draft --base main

linear project create --team <TEAM> --name "Q3 polish" --description "..."
linear project-update create <projectId> --body-file /tmp/status.md --health onTrack
linear milestone create --project "<project>" --name "Beta" --target-date 2025-08-01
linear initiative-update create <initiativeId> --body-file /tmp/timeline.md --health atRisk
linear label create --team <TEAM> --name "needs-design"
linear document create --title "Auth model" --content-file /tmp/doc.md --project "<project>"
linear document update <documentId> --content-file /tmp/doc.md
```

## Markdown content

When the body or description is more than a single line, **always write it to a tempfile and pass the file flag**. Inline strings break on shell escaping, lose newlines, and produce literal `\n` in the Linear UI.

File flag matrix — only these commands accept a file flag:

| Command                                      | Flag                           |
| -------------------------------------------- | ------------------------------ |
| `issue create` / `issue update`              | `--description-file <path>`    |
| `issue comment add` / `issue comment update` | `--body-file <path>`           |
| `project-update create`                      | `--body-file <path>`           |
| `initiative-update create`                   | `--body-file <path>`           |
| `document create` / `document update`        | `-f` / `--content-file <path>` |

Notably **`project update`, `milestone create/update`, `initiative create/update`, `team create`, `project create`, `label create` have no file flag** — only inline `-d/--description`. For long descriptions on those, either keep it terse or fall back to `linear api` with a mutation.

```bash
body=$(mktemp -t linear-body.XXXXXX.md)
cat > "$body" <<'EOF'
Confirmed the regression starts at v0.42.

- Repro: https://...
- Suspect commit: `abc1234`
EOF
linear issue comment add <ID> --body-file "$body"
```

## Teams, projects, cycles, milestones, initiatives, labels, documents

Quick listings. JSON support varies — listed below.

```bash
linear team list                              # no --json
linear team members <TEAM>                    # --all for inactive members
linear team id                                # configured team UUID

linear project list --team <TEAM> --json
linear project list --all-teams --status started
linear project view <project>
linear project-update list <project> --json --limit 50

linear cycle list --team <TEAM>               # no --json
linear cycle view <cycle> --team <TEAM>

linear milestone list --project "<project>"   # --project is required, no --json
linear initiative list --json                 # defaults to active only; --all-statuses for everything; --archived to include
linear initiative view <initiative> --json
linear initiative-update list <initiative>

linear label list --team <TEAM> --json
linear label list --workspace                 # workspace-scoped labels only (different from global --workspace <slug>!)
linear label list --all                       # workspace + team labels

linear document list --project "<project>" --json
linear document view <document> --raw         # raw markdown
linear document view <document> --json
```

`label list --workspace` (no value) means "workspace-level labels only", which collides visually with the global `--workspace <slug>` (which selects which Linear workspace). The global form requires a value, so the parser disambiguates.

## Raw GraphQL fallback

When no subcommand covers the case, drop to `linear api`.

Discover types and fields by writing the schema to a tempfile first:

```bash
schema=$(mktemp -t linear-schema.XXXXXX.graphql)
linear schema -o "$schema"
rg -i "cycle" "$schema"
rg -A 30 "^type Issue " "$schema"
```

Make a request. **Queries with non-null type markers (`String!`, `Int!`, `IssueFilter!`) must be passed via heredoc** — escaping `!` on the shell is unreliable. Simple queries without `!` can go inline.

Variable forms:

- `--variable key=value` for scalars (coerces booleans, numbers, null)
- `--variable key=@/path/to/file` to read a variable value from a file (useful for long markdown bodies on commands with no file flag)
- `--variables-json '{...}'` for objects / arrays

```bash
# Simple — inline is fine
linear api '{ viewer { id name email } }'

# With variables — heredoc required
linear api --variable teamId=abc123 <<'GRAPHQL'
query($teamId: String!) { team(id: $teamId) { name key } }
GRAPHQL

# Complex variables via JSON
linear api --variables-json '{"filter": {"state": {"name": {"eq": "In Progress"}}}}' <<'GRAPHQL'
query($filter: IssueFilter!) { issues(filter: $filter) { nodes { identifier title } } }
GRAPHQL

# Auto-paginate a single connection
linear api --paginate '{ issues(first: 100) { nodes { identifier } pageInfo { hasNextPage endCursor } } }'

# Silent mode (exit code only, for non-destructive mutations)
linear api --silent '{ viewer { id } }'

linear api '{ issues(first: 5) { nodes { identifier title } } }' | jq '.data.issues.nodes[].title'
```

For full HTTP control, `linear auth token` exposes the bearer token:

```bash
curl -s -X POST https://api.linear.app/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: $(linear auth token)" \
  -d '{"query":"{ viewer { id } }"}'
```

## Agent sessions

Linear tracks AI-agent activity per issue. If the user asks about an agent session:

```bash
linear issue agent-session list <ID>
linear issue agent-session view <sessionId>
```

## Confirm before bulk operations

Before creating more than ~3 issues in a single batch (e.g. RFC → tickets), show the user:

- The list of titles
- Target team, project, and any shared labels / parent / cycle
- Whether issues will be created in `triage` / `backlog` / `unstarted`

Wait for explicit go-ahead before running the loop. Don't silently spam a project.

## Forbidden mutations

This skill can read and create / update — but **never** run destructive commands. Destructive removals in Linear are irreversible from the CLI. Stop and confirm with the user before any of these:

- `linear issue delete`
- `linear issue comment delete`
- `linear issue relation delete`
- `linear team delete`
- `linear project delete`
- `linear milestone delete`
- `linear initiative delete`, `linear initiative archive`, `linear initiative remove-project` (all support `--bulk` — extra dangerous)
- `linear label delete`
- `linear document delete` (supports `--bulk`)
- `linear auth logout`, `linear auth default`, `linear auth migrate` — don't change the user's auth state
- `linear config` — writes a `.linear.toml` into the project that will be committed; defer to the user
- `linear team autolinks` writes — modifies external GitHub integration config
- `linear api` with a destructive GraphQL mutation (anything matching `*Delete` / `*Archive` / `*Remove`)

`linear initiative unarchive` is non-destructive and may be used if needed.

**If the user explicitly asks you to delete or archive something:** stop, confirm what exactly will be removed, and only proceed on explicit go-ahead. Default to refusing.

**If you're unsure whether a command mutates destructively:** run `linear <command> --help` and read it. Default to not running it.

## Common mistakes

One-line index of footguns covered above:

- Wrong issue ID format — use the short ID with team prefix (`ENG-123`), not the UUID.
- `--type` flag on `issue relation add` — doesn't exist; relation type is the second positional arg (`blocks`, `blocked-by`, `related`, `duplicate`).
- `linear issue attach <ID> <url>` — `attach` takes a filepath; use `linear issue link <ID> <url>` for URLs.
- `--json` on `issue mine`, `team list`, `cycle list`, `milestone list` — not supported; use `issue query` / fall back to `linear api`.
- `--description-file` on `project update` / `milestone update` / `initiative update` / `team create` / `project create` / `label create` — not supported; see the file flag matrix.
- Combining `--search` with `--sort` on `issue query` — `--search` disables sort.
- `--milestone` without `--project` — both required together.
- State names on `mine`/`query` filters — fixed set of 6 types only; "In Progress" is invalid here (but valid on `create`/`update`).
- Silently picking a team — ask the user once per session.
- Forgetting `--no-interactive` in scripted creates — the CLI may prompt and stall.
- Inline `-d` / `-b` for multi-line markdown — use the file flag.
- Forgetting `!` escapes in `linear api` — use a heredoc.
- `linear issue commits` on a git repo — it's jj-only; use `git log --grep` instead.
- Running `linear auth login` proactively — let the CLI prompt the user.
- Asking the user for the current issue — try `linear issue id` first.
