---
name: refine-changeset
description: >-
  Update an Intent changeset and linked specs to match what was actually built
  in code. Gathers local diffs, compares against requirements, drafts a
  refinement message, and sends it via refine_changeset after user confirmation.
  Use when implementation diverged from spec, scope changed during coding, the
  user wants to sync specs from code, or after verify when gaps should update
  requirements rather than code. Do not use for merge-readiness — run
  verify-changeset (/intent-verify) first.
---

# Refine Changeset

Align the changeset and linked specs with local implementation. **Never edit `.intent/` files locally** — use `refine_changeset` after the user confirms the draft message.

This is not a completeness check. Run **verify-changeset** (`/intent-verify`) first when merge-readiness is unknown — refine only handles deviations where the code is right and the spec should change.

Apply shared patterns from the **intent-changeset** rule.

## Workflow

### Step 1: Resolve changeset and match repos

Resolve the changeset ID and match linked repos to the workspace per the **intent-changeset** rule.

### Step 2: Load current requirements

**From MCP** — call `get_changeset` for title, status, todos, and comments.

**From local `.intent/`** — read the changeset and linked specs per the **intent-changeset** rule. Note acceptance criteria, testing notes, Included/Not Included, and open todos.

### Step 3: Collect local changes

For each matched local repo:

```bash
git fetch origin
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
git diff origin/<default-branch>...HEAD --stat
git diff origin/<default-branch>...HEAD
```

Include uncommitted changes if present (`git status --short`, `git diff`). Flag linked repos with no local checkout or empty diff.

Review changed files and infer behavior from the diff — do not assume unimplemented features exist.

### Step 4: Find deviations

Compare the diff against acceptance criteria, testing notes, and relevant spec sections. Only note places where **code diverges from the spec** — behavior differs, scope was added or dropped, or wording no longer matches what was built.

Skip requirements that match the implementation. Do not list what was built correctly.

Prefer citing files and symbols from the diff. Be factual — the message is input to Intent's specify AI.

### Step 5: Draft refinement message

List deviations only — one bullet per divergence:

```markdown
- [requirement or spec section]: spec says …; code does … (files: …)
```

If there are no deviations, tell the user and stop — do not call `refine_changeset`.

If the user provided focus or instructions (e.g. "only acceptance criteria"), limit comparison to that scope.

### Step 6: Confirm with user

Present the changeset (title, short ID) and the draft deviation list. Ask the user to **confirm**, **edit**, or **cancel**. Do not call `refine_changeset` until they explicitly approve.

### Step 7: Send refinement

After approval, call:

```json
{
  "changesetId": "<uuid>",
  "message": "<approved draft>"
}
```

If the user edited the draft, use their version.

### Step 8: Report

Summarize:

1. That `refine_changeset` was sent and any assistant replies returned by MCP
2. Local `.intent/` syncs from Intent on a ~5 minute interval — re-read after sync or use MCP for latest
3. Suggested follow-up: `/intent-verify` to check alignment, or continue coding if requirements were clarified

Do not call `refine_changeset` without confirmation. Do not edit `.intent/` files locally.
