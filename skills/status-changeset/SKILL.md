---
name: status-changeset
description: >-
  Show a quick status summary for an Intent changeset — metadata, todos, PRs,
  and local workspace state. Use when the user asks for changeset status, what's
  in progress, PR links, todo progress, or a quick overview before verify or close.
---

# Status Changeset

Quick dashboard for the current (or specified) changeset. Read-only — do not update todos, status, or specs. Apply shared patterns from the **intent-changeset** rule.

## Workflow

### Step 1: Resolve changeset

Use the first available source from the **intent-changeset** rule: user-provided ID, current branch prefix, or MCP listing.

### Step 2: Fetch remote state

Call MCP for data **not** in local `.intent/` files:

- `get_changeset` — title, status, type, todos (checked state), comment count or latest comment snippet
- `list_pull_requests` — PR title, state, and URL per linked repo
- `list_repositories` — resolve linked repo names/URLs from `repositoryIds`

Do **not** use MCP for acceptance criteria or spec text when `.intent/` exists locally (see Step 3).

### Step 3: Check local workspace

For each linked repo that matches a git root in the workspace:

```bash
git branch --show-current
git status --short
git fetch origin
git rev-list --left-right --count origin/<default-branch>...HEAD
```

Note repos linked to the changeset but missing locally. Skip repos not in the workspace.

If on an intent branch in a matched repo, **read** `.intent/changesets/<prefix>*` locally for the changeset summary and spec links — do not fetch the same content from MCP. Note the file's modification time as a sync hint (Intent syncs from the server ~every 5 minutes).

### Step 4: Report

Keep it scannable — one screen when possible:

1. **Header** — title, short ID, status, type
2. **Todos** — `checked/total` with unchecked items listed; omit checked items when there are many
3. **Pull requests** — linked PRs with URLs and state; call out repos with no PR
4. **Local workspace** — table: repo, current branch, uncommitted changes (yes/no), commits ahead/behind default branch
5. **Sync hint** — whether local `.intent/` was read and when it was last modified; prefer MCP for latest todos/comments
6. **Suggested next step** — one line (e.g. run `/intent-verify`, finish unchecked todos, open a PR for repo X)

Do not run full verification or start implementation. Point the user to `/intent-verify` for a completeness check or `/intent-start` to pick up different work.
