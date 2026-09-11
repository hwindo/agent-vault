# pi-vault

A [pi](https://pi.dev) package for working with an Obsidian knowledge vault: the vault holds the *why* (intent, rejected options, unresolved questions), the repository holds the *how*.

## Install

```bash
pi install git:github.com/hwindo/pi-vault@v0.1.0
```

## Contents

### Skill: `vault`

Loads when a session touches the vault: reading notes, asking what is currently in progress, recording a decision or lesson, or working inside the vault directory.

- **Read order** — `AI.md` → `NOW.md` → `Home.md` → project hub → smallest useful note.
- **Routing** — where an artifact belongs: ADR, `Inbox.md`, a `NOW.md` focus row, a project note, or a daily note.
- **Verify before acting** — a focus row is a claim, not a fact; check it against the source it names before trusting it.
- **Editing rules** — preserve frontmatter, wikilinks, embeds, tags, callouts, and block IDs; link rather than copy.

### Prompt templates

| Command | What it does |
|---|---|
| `/now [thread]` | Reconcile the focus board against repo reality, report the drift, then work one thread and rewrite the rows that changed. |
| `/retire <thread>` | Verify a thread's outcome, record it durably in the project note or ADR, remove its `NOW.md` row. |

## Configuration

The vault path (`~/Documents/windo-vault`) is written literally into `skills/vault/SKILL.md` and the two templates. Fork and replace it to point at your own vault.

The package assumes the vault's own layout: an `AI.md` entry point, a `NOW.md` focus board, `Home.md`, `Inbox.md`, project hubs, and `Meta/` notes. The vault's own `AGENTS.md` remains the authority on how its notes are edited.
