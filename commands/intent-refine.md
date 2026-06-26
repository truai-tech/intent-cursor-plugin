---
name: intent-refine
description: Update the changeset and linked specs to match local code via refine_changeset, after drafting and confirming the refinement message.
---

# Intent Refine

Align the changeset and linked specs with what was actually built locally.

Follow the **refine-changeset** skill exactly. Apply shared patterns from the **intent-changeset** rule.

## Resolve changeset

Prefer auto-detection from the current branch (`intent/<id>-<slug>`). If not on an intent branch and no ID was provided, resolve per the **intent-changeset** rule and ask the user to pick one.

## Arguments

- **`$1` (optional)** — Focus or extra instruction for the draft (e.g. "only update acceptance criteria", "document the API change in repo X").

## Output

Present the draft refinement message and wait for user confirmation before calling `refine_changeset`. Never edit `.intent/` files locally.
