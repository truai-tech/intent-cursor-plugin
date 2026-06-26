---
name: intent-start
description: Pick up an Intent changeset and prepare the local workspace. Optional changeset ID as the first argument.
---

# Intent Start

Pick up an in-progress changeset and prepare the local workspace.

Follow the **start-changeset** skill exactly. Apply shared patterns from the **intent-changeset** rule.

## Arguments

- **`$1` (optional)** — Changeset UUID or short ID. When provided, use it directly and skip project/changeset selection (Steps 1–3 in the skill).

## Output

After checkout and reading specs, present the brief implementation outline from the skill. Do not start implementing unless the user asks.
