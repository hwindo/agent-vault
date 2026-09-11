---
name: vault
description: Navigate and edit the Obsidian knowledge vault at ~/Documents/windo-vault. Use when reading vault notes, asking what is currently in progress or what we are working on, recording a decision, lesson, or rationale, or when the working directory is inside the vault.
---

# Vault

`~/Documents/windo-vault` is the knowledge vault: the source of truth for **why** — intent, rejected options, unresolved questions, traps already paid for. The repository stays the source of truth for **how**.

## Read in this order

1. `AI.md` — entry point and operating rules. Read it before substantive work.
2. `NOW.md` — the focus board: what deserves attention before starting new work.
3. `Home.md` — routes to the project or cross-project pattern.
4. The project hub, then the smallest useful note: its area, a decision, open questions, or gotchas.

`Meta/AI Model Configuration.md` picks the model and reasoning depth for a task.

## Route what you learn

| Artifact | Lives in |
|---|---|
| Decision with rationale | The project's ADR |
| Loose thought, when filing would interrupt the work | `Inbox.md` |
| Active work, blocker, or deadline | A `NOW.md` focus row |
| Durable outcome | The project note or ADR — not `NOW.md` |
| Dated observation needing its own context | A daily note, linked from the project |

`NOW.md` keeps only active work, a decision deadline, or a blocker that changes what happens next. It is a focus board, not a task list and not a log: record the outcome somewhere durable, then retire the row.

## Verify before acting

A focus row is a claim, not a fact. Check it against the source it names — `git log`, `gh pr view`, deploy state, the plan doc — and report drift before acting on it.

## Editing notes

Preserve YAML frontmatter, `[[wikilinks]]`, embeds, tags, callouts, and block IDs; prefer relative vault paths and Obsidian-style links; leave `.obsidian/` alone unless the change is a requested settings change; ask before deleting, bulk-renaming, or reorganizing notes. Link to an existing note instead of copying it.
