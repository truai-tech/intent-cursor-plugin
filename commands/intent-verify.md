---
name: intent-verify
description: Verify the local implementation against the active Intent changeset, specs, and acceptance criteria.
---

# Intent Verify

Check whether the local implementation satisfies the changeset, linked specs, and acceptance criteria.

Follow the **verify-changeset** skill exactly. Apply shared patterns from the **intent-changeset** rule.

## Resolve changeset

Prefer auto-detection from the current branch (`intent/<id>-<slug>`). If not on an intent branch and no ID was provided, resolve per the **intent-changeset** rule and ask the user to pick one.

## Output

Present the structured verification report from the skill. Be direct about gaps. Do not start fixing unless the user asks.
