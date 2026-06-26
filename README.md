# Intent Cursor Plugin

Cursor plugin for [Intent](https://onintent.build) — pick up changesets, work on intent branches, verify against specs, and refine requirements from code.

For teams using Intent and Cursor. Requires an Intent account, indexed repositories, and an API token.

---

## What is Intent?

[Intent](https://onintent.build) is a product development platform that keeps **specs, changesets, and code** aligned as work moves from idea to merge.

- **Changesets** track a unit of work — a feature, bug fix, or refactor — with acceptance criteria, testing notes, and todos.
- **Specs** describe how things should work, linked to changesets and versioned over time.
- **Repositories** are indexed and linked to changesets; Intent syncs changeset and spec content into a `.intent/` directory in each repo on a regular interval.
- **Intent branches** (`intent/<id>-<slug>`) tie local git work back to a changeset, with PRs tracked per repo.

Intent is the source of truth for requirements. This plugin brings that context into Cursor so the agent knows *what* to build, can check *whether it's done*, and can update specs when implementation diverges — without leaving the IDE.

---

## What this plugin does

The plugin connects Cursor to Intent via MCP and packages the workflows your team needs day to day:

- **MCP integration** — read changesets, specs, repos, and PRs; write updates through Intent's API (with confirmation)
- **Slash commands** — explicit entry points for start, status, verify, and refine
- **Rules** — always-on conventions (branch naming, repo discovery, never edit `.intent/` locally)
- **Skills** — detailed agent workflows invoked by commands or context

```text
Intent UI  →  changeset + specs  →  .intent/ in repo  →  Cursor agent
                                       ↑                        ↓
                                 refine_changeset  ←──  /intent-refine
```

---

## Prerequisites

- An [Intent](https://onintent.build) account with at least one project and indexed repositories
- Repos linked to changesets, with `.intent/` content synced from Intent
- Cursor with plugin and MCP support
- An Intent API token (see Setup)

**Workspace expectations:**

- One or more git repos from the changeset checked out in your Cursor workspace
- Intent branches named `intent/<8-char-id>-<slug>` (created automatically when you start work on a changeset)

---

## Installation

1. Install the plugin from the **Cursor Marketplace**, or load it locally (see [Local development](#local-development)).
2. Set your Intent API token:

   ```bash
   export INTENT_API_TOKEN="your-token-here"
   ```

   Create a token in Intent settings.

3. Open **Cursor Settings → Customize** and enable the Intent plugin.
4. Confirm the **intent-mcp-server** MCP server is enabled in Customize.

---

## Quick start

A typical session:

1. **`/intent-start`** — pick up an in-progress changeset, checkout intent branches, read specs, get an implementation outline
2. **Implement** — the agent follows plugin rules automatically; edit source files, not `.intent/`
3. **`/intent-status`** — quick check: todos, PR links, local branch state
4. **`/intent-verify`** — full completeness check against acceptance criteria and specs
5. **`/intent-refine`** — if code diverged from spec and the spec should catch up (requires confirmation)
6. **Commit and merge** — standard git workflow; merge PRs linked to the changeset

**Example invocations:**

```text
/intent-start c6d75b30
/intent-verify
/intent-refine only acceptance criteria
```

---

## Commands

| Command | When to use | Args |
| --- | --- | --- |
| `/intent-start` | Pick up work on a changeset | `[id]` — optional UUID or short ID |
| `/intent-status` | Quick dashboard before diving in | `[id]` — optional |
| `/intent-verify` | Check merge-readiness | — |
| `/intent-refine` | Sync spec from local code | `[focus]` — optional scope hint |

### `/intent-start`

Lists in-progress changesets (or uses the ID you pass), checks out intent branches in matched local repos, reads the changeset and linked specs, and presents a brief implementation outline. Does not start coding unless you ask.

For large or cross-cutting changesets, the agent may suggest Plan mode before implementation.

### `/intent-status`

Read-only snapshot: changeset metadata, todo progress, linked PRs, local branch and diff state, and a sync hint for `.intent/` files. Use this for a quick "where am I?" — not a full audit.

### `/intent-verify`

Compares local code against acceptance criteria, testing notes, and linked specs. Returns a structured checklist (Met / Partial / Missing / Untestable), test results if run, gaps, and an overall verdict. Read-only — does not write to Intent.

### `/intent-refine`

Finds places where code diverges from the spec, drafts a deviation list, and asks you to confirm before sending it to Intent via `refine_changeset`. Not a completeness check — run verify first when merge-readiness is unknown.

---

## Verify vs refine

**Verify** asks *is the work done?* — the spec is the judge. You get a full checklist, test results, and a merge-readiness verdict.

**Refine** asks *update the spec to match what I built* — code is the judge for mismatches. Only deviations are sent to Intent.

| Situation | Use |
| --- | --- |
| "Am I done? Ready to merge?" | `/intent-verify` |
| "I built it differently — update Intent" | `/intent-refine` |
| Verify found gaps that are spec-side | `/intent-refine` on those deviations |
| Verify found gaps that are code-side | Keep coding |
| Spec already matches code | Refine stops; verify should show mostly Met |

After verify, if a gap means the spec is wrong rather than the code, run refine — not the other way around.

---

## Core concepts

### Changesets

A changeset is a tracked unit of work with a title, status, type, acceptance criteria, testing notes, todos, and links to specs and repositories. Status flows through draft → in_progress → completed (and related states). The plugin focuses on **in_progress** work for pickup; verify and refine assume you are on or near an intent branch.

### Intent branches

Branches follow `intent/<changeset-id-prefix>-<slugified-name>`, e.g. `intent/c6d75b30-conversation-context-summarization`. The first 8 characters after `intent/` identify the changeset. The same prefix matches files under `.intent/changesets/`.

### `.intent/` in your repo

Intent syncs changeset and spec markdown into `.intent/changesets/` and `.intent/specs/` in linked repos roughly every five minutes. **Intent is the source of truth** — never create, edit, or delete these files locally. To update specs or changeset content, use `/intent-refine` or ask the agent to call `refine_changeset` after your confirmation.

When on an intent branch, prefer reading `.intent/` locally for specs; use MCP for todos, comments, and metadata.

### Multi-repo workspaces

A changeset can span multiple repositories. The plugin matches linked repo URLs to git roots in your workspace and only operates on repos that are both linked and present locally. Others are skipped silently — this is normal when repos are split across workspaces or machines.

---

## MCP tools

The plugin connects to `https://api.onintent.build/mcp/` using your `INTENT_API_TOKEN`.

### Read

| Tool | Purpose |
| --- | --- |
| `list_projects` | Resolve project ID |
| `list_changesets` | Filter changesets by status, type, project |
| `get_changeset` | Full detail: todos, comments, linked repos |
| `list_artifacts` / `get_artifact` | Specs when not on an intent branch |
| `list_repositories` | Map repo IDs to names and URLs |
| `list_pull_requests` | PRs linked to a changeset |
| `explore` | Natural-language codebase questions |
| `read_file` | Read a file outside the workspace |

### Write

The agent asks before calling write tools.

| Tool | Purpose |
| --- | --- |
| `refine_changeset` | Update specs/changeset via Intent's specify AI |
| `update_changeset` | Status and todos (todos are full replacement) |
| `add_comment` | Post a comment on a changeset |
| `link_repository` | Link a repo; creates branch/PR when in_progress |
| `preflight_check` | AI readiness check before closing |

Prefer local search and file reads when repos are checked out; use MCP for remote context and cross-repo questions.

---

## Rules, skills, and commands

Three layers work together:

| Layer | Role | Examples |
| --- | --- | --- |
| **Rules** | Always-on conventions for every agent turn | branch naming, repo discovery, don't edit `.intent/` |
| **Skills** | Detailed step-by-step workflows | `start-changeset`, `verify-changeset`, … |
| **Commands** | Manual slash entry points | `/intent-start`, `/intent-verify`, … |

Each command delegates to a skill:

| Skill | Command | Purpose |
| --- | --- | --- |
| `start-changeset` | `/intent-start` | Pick up work, checkout, outline |
| `status-changeset` | `/intent-status` | Quick dashboard |
| `verify-changeset` | `/intent-verify` | Completeness audit |
| `refine-from-code` | `/intent-refine` | Sync spec from code |

---

## Configuration

### Required

| Variable | Used by | Description |
| --- | --- | --- |
| `INTENT_API_TOKEN` | `mcp.json` | Bearer token for Intent MCP API |

Set in your shell profile or environment before starting Cursor.

---

## Troubleshooting

**MCP not connecting** — Check `INTENT_API_TOKEN` is set, the token is valid, and **intent-mcp-server** is enabled in Customize.

**Command not in palette** — Reload the window (**Developer: Reload Window**). Confirm the plugin is installed and enabled.

**Agent edits `.intent/` files** — It shouldn't. Rules forbid local edits; use `/intent-refine` to update specs through Intent.

**Stale specs after refine** — Local `.intent/` syncs on an interval. Re-read after a few minutes or use `get_changeset` via MCP for the latest todos and comments.

**Repo not found or skipped** — Repo may not be linked to the changeset, not checked out in this workspace, or missing an intent branch. Run `/intent-status` to see what's matched locally.

**Wrong changeset detected** — Pass the ID explicitly (`/intent-start <id>`) or checkout the intent branch first.

---

## Local development

Test the plugin before publishing:

1. Symlink or copy the plugin into Cursor's local plugins directory:

   ```bash
   ln -s /path/to/intent-cursor-plugin ~/.cursor/plugins/local/intent
   ```

2. Restart Cursor or run **Developer: Reload Window**.
3. Enable the plugin and MCP server in **Customize**.
4. Invoke commands with `/` in Agent chat.

### Repository layout

```text
intent-cursor-plugin/
├── .cursor-plugin/
│   └── plugin.json        # Plugin manifest (required)
├── assets/
│   └── logo.svg
├── mcp.json               # Intent MCP server configuration
├── commands/              # Slash commands
├── rules/                 # Cursor rules (.mdc)
├── skills/                # Agent skills (SKILL.md per directory)
└── README.md
```

Components in `rules/`, `skills/`, `commands/`, and `mcp.json` are auto-discovered. See the [Cursor plugin docs](https://cursor.com/docs/plugins) for manifest fields and submission.

---

## License

MIT — see [LICENSE](LICENSE).

## Links

- [Intent](https://onintent.build)
- [Plugin repository](https://github.com/truai-tech/intent-cursor-plugin)
- [Cursor plugins](https://cursor.com/docs/plugins)
