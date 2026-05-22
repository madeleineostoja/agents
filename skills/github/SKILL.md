---
name: github
description: Use the GitHub CLI (gh) for all read-only GitHub work — reading files from a repo, repo/issue/code search, PR review, Actions runs. ALWAYS prefer gh over web fetching, and prefer `gh api repos/.../contents/...` over cloning when you just need to read a handful of files. Trigger when the user names a repo/issue/PR, pastes a github.com URL (including `/tree/` or `/blob/` paths), wants to debug a library via its issues, find real-world usage via code search, inspect Actions, or review a PR. NEVER use for mutations.
---

# Using the gh CLI

`gh` is the canonical interface to GitHub. Prefer it over web fetches, screen scraping, or unauthenticated API calls — it handles auth, pagination, and rate limits for you.

## When to reach for gh

- The user names a GitHub repo, issue, PR, or pastes a `github.com/...` URL
- Debugging a library — check its issue tracker for known bugs before going deeper into source
- Looking for real-world usage examples of an API, function, or config option
- Reviewing a PR, comments, checks, or commits
- Reviewing Actions runs

If the answer might be in GitHub data, try `gh` before grepping further or asking the user.

## Reading files vs. cloning

When the user points at a repo or subtree (e.g. `github.com/owner/repo/tree/main/packages/x`) and you need to read source:

- **≤5 named files, or a single subtree you can browse top-down → use `gh api` contents.** No local state, no cleanup.
- **Broad grep/glob across many files, or you need to run the code → `gh repo clone --depth 1` into `$(mktemp -d)`.**

**Do NOT `git clone` a public repo into /tmp just to read a handful of files.** That's the wrong default — use the contents API:

```bash
# List a directory
gh api repos/owner/repo/contents/packages/coding-agent

# Read a file (contents are base64-encoded)
gh api repos/owner/repo/contents/packages/coding-agent/package.json -q .content | base64 -d

# Browse a whole subtree in one call
gh api repos/owner/repo/git/trees/HEAD:packages/coding-agent
```

If you do clone, always use `--depth 1` and clone into `$(mktemp -d)`, not a hardcoded `/tmp` path.

## Output formatting

Default `gh` output is human-formatted and verbose. For programmatic consumption prefer:

- `--json <fields> -q '<jq expr>'` to get structured, filtered output
- `--limit N` to cap large lists
- `gh <cmd> --help` to discover available `--json` fields

Example: `gh issue list -R owner/repo --json number,title,state,labels -q '.[] | select(.state=="OPEN")'`

**Prefer dedicated subcommands** (`gh repo view`, `gh pr view`, `gh search code`) over raw `gh api` when one fits — they have saner defaults and clearer output. Reach for `gh api` only when no subcommand covers the case (notably: reading file contents, fetching review threads, traversing git trees).

## Repo lookup and search

```bash
# List files at a path / view file contents at a ref (see also: Reading files vs. cloning)
gh api repos/owner/repo/contents/path/to/dir
gh api repos/owner/repo/contents/path/to/file.ts -q .content | base64 -d
gh api repos/owner/repo/git/trees/HEAD:path/to/dir   # full subtree in one call

# View repo metadata, README, topics, default branch
gh repo view owner/repo
gh repo view owner/repo --json description,defaultBranchRef,stargazerCount,licenseInfo

# Search for repos
gh search repos "vector database rust" --limit 10
gh search repos --owner=anthropics --language=python
```

## Issue lookup and search

Issue search is the fastest way to check for **known bugs** before diving into a library's source. Always check both open and closed issues — the closed ones often have the workaround.

```bash
# View a specific issue with comments
gh issue view 1234 -R owner/repo --comments

# Search across all of GitHub (broad — good for "is this a known problem?")
gh search issues "TypeError cannot read undefined" --repo=owner/repo --state=all
gh search issues "memory leak" --repo=facebook/react --state=closed --limit 20

# Filter within one repo
gh issue list -R owner/repo --search "panic in parser" --state all --limit 20
gh issue list -R owner/repo --label bug --state open

# Find issues mentioning a specific error/symbol
gh search issues "specific-error-string" --repo=owner/repo --state=all
```

**Debugging workflow:** when you hit an unfamiliar error from a library, search its issue tracker for the error message _first_. A 5-second `gh search issues` can save 30 minutes of source spelunking.

## Code search

Code search is **underutilised**. It's the best way to find real usage examples of an API, a config option, or a pattern across the entire ecosystem of open-source repos.

```bash
# Find usage examples of a function across all of GitHub
gh search code "useTransition" --language=tsx --limit 20

# Search within a single org or repo
gh search code "createServerClient" --owner=supabase --language=ts
gh search code "experimental_taintObjectReference" --repo=vercel/next.js

# Filter by filename or path
gh search code "rateLimiter" --filename="middleware.ts"
gh search code "Dockerfile" --extension=yml --path="**/ci/**"

# Find config examples
gh search code "compilerOptions" --filename=tsconfig.json --limit 10
```

## PR reviews

```bash
# View PR metadata and description
gh pr view 1234 -R owner/repo
gh pr view 1234 -R owner/repo --json title,body,state,mergeable,reviewDecision,files

# Diff
gh pr diff 1234 -R owner/repo
gh pr diff 1234 -R owner/repo --name-only         # just changed files
gh pr diff 1234 -R owner/repo -- path/to/file.ts  # restrict to a path

# Comments and review threads
gh pr view 1234 -R owner/repo --comments
gh api repos/owner/repo/pulls/1234/comments       # inline review comments
gh api repos/owner/repo/pulls/1234/reviews        # review summaries

# Commits and checks
gh pr view 1234 -R owner/repo --json commits
gh pr checks 1234 -R owner/repo

# Search PRs
gh search prs "refactor auth" --repo=owner/repo --state=all
gh pr list -R owner/repo --search "is:merged author:@me" --limit 10
```

**For PR review tasks:** start with `gh pr view --json` for the metadata, then `gh pr diff` for the actual change, then `gh api .../comments` for the discussion. Don't paste raw `gh pr diff` output unless asked — summarise and cite specific files/lines.

## GitHub Actions

```bash
# List recent workflow runs (filter by workflow, branch, status, actor, event)
gh run list -R owner/repo --limit 20
gh run list -R owner/repo --workflow=ci.yml --branch=main --status=failure
gh run list -R owner/repo --user=@me --event=pull_request

# View a specific run — summary of jobs and their conclusions
gh run view <run-id> -R owner/repo
gh run view <run-id> -R owner/repo --json status,conclusion,jobs,headBranch,headSha

# Logs — full or just the failed steps (much smaller, usually what you want)
gh run view <run-id> -R owner/repo --log-failed
gh run view <run-id> -R owner/repo --log              # full logs (big)
gh run view <run-id> -R owner/repo --job=<job-id> --log

# Find the failing run for the current PR
gh pr checks 1234 -R owner/repo                       # which checks failed
gh run list -R owner/repo --branch=<pr-branch> --limit 5

# List workflows defined in the repo
gh workflow list -R owner/repo
gh workflow view ci.yml -R owner/repo                 # shows the YAML and recent runs
```

**Debugging workflow:** `gh pr checks` → identify the failing check → `gh run view --log-failed` for the targeted error. Avoid `--log` (full logs) unless `--log-failed` doesn't surface the cause — full logs are huge and burn context.

## Authentication and rate limits

- Assume `gh` is already authenticated (`gh auth status` to verify). If it fails with auth errors, tell the user — do not try to log them in.
- Hitting rate limits? Use `--json` + `-q` to fetch only what you need, and `--limit` to cap pagination.

## Forbidden mutations

This skill is read-only. **Never** invoke any `gh` command that writes to GitHub. The full list of forbidden verbs/flags:

- `gh repo create`, `gh repo delete`, `gh repo fork`, `gh repo rename`, `gh repo edit`, `gh repo archive`, `gh repo unarchive`, `gh repo set-default`
- `gh issue create`, `gh issue close`, `gh issue reopen`, `gh issue comment`, `gh issue edit`, `gh issue delete`, `gh issue lock`, `gh issue unlock`, `gh issue transfer`, `gh issue pin`, `gh issue unpin`, `gh issue develop`
- `gh pr create`, `gh pr close`, `gh pr reopen`, `gh pr comment`, `gh pr edit`, `gh pr merge`, `gh pr ready`, `gh pr review` (when `--approve`/`--request-changes`/`--comment`)
- `gh release create`, `gh release delete`, `gh release edit`, `gh release upload`, `gh release download` (writes locally, but usually unintended)
- `gh gist create`, `gh gist edit`, `gh gist delete`, `gh gist clone`
- `gh workflow run`, `gh workflow enable`, `gh workflow disable`, `gh run cancel`, `gh run rerun`, `gh run delete`
- `gh label create`, `gh label edit`, `gh label delete`, `gh label clone`
- `gh secret set`, `gh secret delete`, `gh variable set`, `gh variable delete`
- `gh ssh-key add`, `gh ssh-key delete`, `gh gpg-key add`, `gh gpg-key delete`
- `gh project create`, `gh project edit`, `gh project delete`, `gh project item-add`, `gh project item-edit`, `gh project item-delete`, `gh project field-create`, `gh project field-delete`
- `gh api ... -X POST/PATCH/PUT/DELETE` (any non-GET HTTP method) — and `--method`/`-X` flags with those verbs
- `gh auth login`, `gh auth logout`, `gh auth refresh`, `gh auth setup-git`, `gh config set`, `gh alias set`, `gh alias delete`, `gh extension install`, `gh extension remove`, `gh extension upgrade`
- Any `gh` subcommand whose `--help` describes it as creating, deleting, editing, closing, merging, commenting, approving, requesting, uploading, setting, adding, or removing

**If the user asks you to perform a mutation** (e.g. "open an issue", "comment on this PR", "merge it"): stop and confirm explicitly before running anything.

**If you're unsure whether a command mutates:** run `gh <command> --help` and read it. Default to not running it.
