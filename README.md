# agent-vault

Obsidian knowledge vault workflow for coding agents. One repo, three harnesses: [pi](https://pi.dev), Claude Code, and Codex.

The vault holds the **why** — intent, rejected options, unresolved questions, traps already paid for. The repository holds the **how**. This package teaches an agent to read the vault in the right order, route what it learns to the right note, and keep the focus board honest.

## Install

```bash
# pi
pi install git:github.com/hwindo/agent-vault@v0.1.0

# Claude Code
claude plugin marketplace add hwindo/agent-vault
claude plugin install agent-vault@agent-vault

# Codex
codex plugin marketplace add hwindo/agent-vault
codex plugin add agent-vault@agent-vault
```

## Contents

### Skill: `vault`

Loads when a session touches the vault: reading notes, asking what is currently in progress, recording a decision or lesson, or working inside the vault directory.

- **Read order** — `AI.md` → `NOW.md` → `Home.md` → project hub → smallest useful note.
- **Routing** — where an artifact belongs: an ADR, `Inbox.md`, a `NOW.md` focus row, a project note, or a daily note.
- **Verify before acting** — a focus row is a claim, not a fact; check it against the source it names before trusting it.
- **Editing rules** — preserve frontmatter, wikilinks, embeds, tags, callouts, and block IDs; link rather than copy.

### Commands: `/now` and `/retire`

| Command | What it does |
|---|---|
| `/now [thread]` | Reconcile the focus board against repo reality, report the drift with evidence, then work one thread and rewrite the rows that changed. |
| `/retire <thread>` | Verify a thread's outcome, record it durably in the project note or ADR, remove its `NOW.md` row. Stops if an outcome is unevidenced. |

Claude Code and pi expose both as slash commands. Codex ships the skill; there, ask for the focus loop in prose.

## Layout

```
├── skills/vault/SKILL.md              # the skill (all three harnesses)
├── commands/now.md, retire.md         # slash commands
├── package.json                       # pi package manifest
├── .claude-plugin/                    # Claude Code plugin + marketplace
├── .codex-plugin/                     # Codex plugin
└── .agents/plugins/marketplace.json   # Codex marketplace
```

## Configuration

The vault path (`~/Documents/windo-vault`) is written literally into `skills/vault/SKILL.md` and both commands. Fork the repo and replace it to point at your own vault.

The package assumes the vault's layout: an `AI.md` entry point, a `NOW.md` focus board, `Home.md`, `Inbox.md`, project hubs, and `Meta/` notes. The vault's own `AGENTS.md` remains the authority on how its notes are edited.
