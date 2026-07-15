---
name: start-changeset
description: >-
  Pick up a changeset from the Intent project and prepare the local workspace.
  Use when the user provides a changeset ID (UUID or short ID), asks to start
  working on a changeset, pick up a changeset, resume work on a changeset, or
  mentions a changeset by name. Also use when the user pastes what looks like a
  UUID or short hex identifier in the context of starting work.
---

# Start Changeset

Workflow to pick up an in-progress changeset and prepare the local workspace. Apply shared patterns from the **intent-changeset** rule (resolve ID, repo discovery, branch checkout, read `.intent/`).

## Workflow

### Step 1: Resolve project

If the user provided a changeset ID, skip to Step 3.

Otherwise call `list_projects` and ask the user to pick a project. Use the selected project's `id` as `projectId` for the next step.

### Step 2: List in-progress changesets

Call `list_changesets`:

```json
{
  "projectId": "<project-id>",
  "status": ["in_progress"]
}
```

The response is an object with an `items` array: `{ items: [...] }`. Extract changesets from `items`.

### Step 3: Ask user to pick one

If the user did not already provide a changeset ID, present changesets using AskQuestion (or conversationally if unavailable). Show title and type for each option.

### Step 4: Get changeset details and match repos

Call `get_changeset` with the selected `changesetId` for **metadata only**: linked `repositoryIds`, todos, status, and comments. Do not use the MCP response for acceptance criteria or spec text if local `.intent/` files will be available after checkout.

Match linked repos to the workspace per the **intent-changeset** rule (repo discovery).

### Step 5: Checkout branches

Check out intent branches in matched local repos per the **intent-changeset** rule (branch checkout).

**Critical:** track the **remote** branch (`git switch --track origin/intent/...`). Never run `git checkout -b`. If no remote intent branch exists for a repo, skip it and report that — do not create a local branch.

### Step 6: Read changeset and specs

**Read from local `.intent/` files** in the first repo where Step 5 succeeded. Use the Read/Grep tools on:

- `.intent/changesets/<first-8-chars>*` — requirements, acceptance criteria, testing notes
- Linked files under `.intent/specs/` from the changeset's `## Specs` section

Only fall back to MCP (`get_artifact`, `list_artifacts`) if `.intent/` is missing or you are not on an intent branch. Merge in todo/status/comment data from Step 4's `get_changeset` call.

### Step 7: Outline implementation

Present a **brief** implementation outline in chat covering:
- What each affected repo needs to do
- Key files likely involved (infer from spec + repo structure)
- Suggested order of implementation
- Open questions or ambiguities

Keep it to ~10-20 bullet points max. Do not start implementing yet.

For larger or more complex changesets — multiple repos, cross-cutting refactors, unclear scope, or many open questions — suggest the user switch to **Plan mode** before implementation. Plan mode is better suited to working through architecture, trade-offs, and a step-by-step approach. For smaller, well-scoped changes, the brief outline is enough to proceed in Agent mode.
