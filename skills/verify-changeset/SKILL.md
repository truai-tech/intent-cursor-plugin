---
name: verify-changeset
description: >-
  Verify that a changeset is fully implemented by comparing local code against
  the changeset, linked specs, and acceptance criteria. Use when the user asks
  to verify, review completeness, check if work is done, validate a changeset,
  or prepare to close or merge a changeset. Also use on intent branches before
  marking a changeset complete. For spec-side gaps, use refine-changeset
  (/intent-refine) — not this skill.
---

# Verify Changeset

Check whether the local implementation satisfies the changeset, its linked specs, and acceptance criteria. Apply shared patterns from the **intent-changeset** rule (resolve ID, repo discovery, read `.intent/`).

## Workflow

### Step 1: Resolve changeset and match repos

Resolve the changeset ID and match linked repos to the workspace per the **intent-changeset** rule.

### Step 2: Load requirements

**Local `.intent/` first** — when on an intent branch and `.intent/` exists, read the changeset and linked specs from disk per the **intent-changeset** rule. Build the verification checklist from local files:

- Each acceptance criterion (bullet)
- Each testing note (bullet)
- Relevant spec sections (Included/Not Included, How It Works, acceptance-style requirements)

**MCP second** — call `get_changeset` only for todos (checked state), status, and comments. Add unchecked todos to the checklist. Do **not** fetch spec content via `get_artifact` or `list_artifacts` when those specs are already linked locally under `.intent/specs/`.

### Step 3: Collect local changes

For each matched local repo:

```bash
git fetch origin
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
git diff origin/<default-branch>...HEAD --stat
git diff origin/<default-branch>...HEAD
```

Review the diff and changed files. Note repos linked to the changeset that have **no local checkout or no diff** — flag as potentially missing work.

Use local search and file reads first. Use `explore` for cross-repo questions when local search is insufficient. Only use `read_file` when the file is not in the workspace.

### Step 4: Verify each requirement

For every checklist item, inspect the diff and relevant source files. Classify each as:

| Status | Meaning |
| --- | --- |
| **Met** | Implementation clearly satisfies the requirement |
| **Partial** | Started but incomplete, or behavior differs from spec |
| **Missing** | No evidence in diff or codebase |
| **Untestable** | Cannot verify without running the app or manual QA |

Do not mark items Met without evidence. Prefer citing specific files and symbols.

### Step 5: Check tests

For each testing note and spec test requirement:

- Search the diff for new or updated test files
- Run the project's test command if known (check `package.json`, `Makefile`, etc.)
- Report pass/fail/skip with reason

If tests were not run, say so explicitly — do not assume they pass.

### Step 6: Report

Present a structured summary:

1. **Changeset** — title, ID, branch(es), repos reviewed
2. **Overall verdict** — Complete / Incomplete / Needs review
3. **Checklist** — table or list of each requirement with status and evidence
4. **Gaps** — missing repos, unchecked todos, missing tests, spec items marked Not Included that appear implemented (scope creep) or Included that are absent
5. **Suggested next steps** — ordered list of what to fix before closing the changeset

For each gap, ask the user whether to **fix in code** or **update the spec/changeset**. For spec-side updates, point to **refine-changeset** (`/intent-refine`) — do not call `refine_changeset` from verify. A gap may mean incomplete implementation or outdated requirements.

| Gap status | Likely next step |
| --- | --- |
| **Missing** | Write more code |
| **Met** | Nothing |
| **Partial** / spec differs | Fix code, or `/intent-refine` if the code is correct |
| **Untestable** | Manual QA or run the app |

Be direct about gaps. Do not start fixing unless the user asks.
