# agent-vault

A knowledge-vault workflow for coding agents — **pi**, **Claude Code**, **Codex**, and **opencode** — plus a delegation loop that splits a task into plan, implement, verify, and review.

The vault holds the **why**: intent, rejected options, unresolved questions, traps already paid for. The repository holds the **how**. This package teaches an agent to read the vault in the right order, keep its focus board honest, and hand work between models you choose per run.

## Install

```bash
# pi
pi install git:github.com/hwindo/agent-vault@v0.3.0

# Claude Code
claude plugin marketplace add hwindo/agent-vault
claude plugin install agent-vault@agent-vault

# Codex
codex plugin marketplace add hwindo/agent-vault --ref v0.3.0
codex plugin add agent-vault@agent-vault

# opencode — no package manager for skills, so link a checkout
git clone https://github.com/hwindo/agent-vault ~/Developments/agent-vault  # skip if it exists
mkdir -p ~/.config/opencode/skills ~/.config/opencode/commands
ln -sfn ~/Developments/agent-vault/skills/vault ~/.config/opencode/skills/vault
for c in now retire delegate verify; do
  ln -sfn ~/Developments/agent-vault/commands/$c.md ~/.config/opencode/commands/$c.md
done
```

Restart the harness afterwards (Claude Code requires it).

## What you get

| Resource | Loads |
|---|---|
| `vault` skill | Automatically, when a session touches the vault |
| `/now [thread]` | When you invoke it |
| `/retire <thread>` | When you invoke it |
| `/verify [plan or task]` | When an agent has finished work and evidence still needs checking |
| `/delegate [plan=· implement=· review=] <task>` | When you invoke it |

Codex ships the skill; the commands are Claude Code, pi, and opencode slash commands. In Codex, describe what you want and the same instructions come from the skill.

## Prerequisites

- **Cross-harness roles need Herdr**, the terminal multiplexer these agents run in. Assigning a role to `claude:*` or `codex:*` starts that agent in a sibling Herdr pane, so the session must run inside Herdr (`HERDR_ENV=1`). Otherwise assign the roles to `pi` actors, which run in the current session.
- `pi:<model>` roles need the shipped `subagent` extension for an isolated context; without it they run inline.
- A vault layout described under [Configuration](#configuration).

## Quick start

```bash
/now                       # reconcile the focus board, pick a thread
/delegate plan=claude:opus implement=pi review=codex:sol add the export retry budget
/retire Restock export safety
```

## Choosing actors

Assign a role per run; an omitted role is **asked for**, never inferred from task size.

```bash
/delegate implement=pi:deepseek-v4-flash <task>          # reuse house defaults for plan and review
/delegate plan=codex:sol implement=codex:sol review=claude:opus <task>
```

| Actor | Runs as |
|---|---|
| `codex:sol`, `codex:luna` | Codex in a sibling Herdr pane (`-m <model>`) |
| `claude:opus`, `claude:sonnet`, `claude:haiku` | Claude Code in a sibling Herdr pane (`--model <alias>`) |
| `pi` | This session |
| `pi:<model>` | A pi subagent on that model, if the subagent extension is installed |

**House defaults:** plan `claude:opus` · implement `pi` · review `codex:sol`.

**Utility work** — summarising, renaming, extraction, boilerplate, commit messages — goes to `codex:luna`, `claude:haiku`, or a local model, and never occupies a loop role.

## The plan file

The handoff bus is `docs/plans/<slug>.md` in the repository being changed. Panes are for watching; anything that matters is written to this file, because a delegate's long output can leave the alternate screen and scrollback will not bring it back.

```markdown
# <Slug>

Status: Draft — <what is still open>

## Question
What this work has to answer.

## Evidence
What was inspected, with paths and line numbers.

## Assumptions and rejected paths
What is being taken on faith, and what was considered and dropped.

## Steps
1. <step> — acceptance: <the check that proves it>

## Result        ← appended by the implementer
What changed, commit shas, tests run, what could not be done.

## Verified      ← appended by the orchestrating session, before any reviewer sees the range
- `git diff <base>..<head>` — reviewed; scope matches the plan
- `<test/build command>` — passed
- `<runtime, integration, or UI check>` — passed, or explicitly not run
- Remaining unverified: <honest gap, or none>

## Review        ← appended by the reviewer
Blocking findings only, each with file and line.
```

**Commit authority:** the implementer commits small and feature-scoped and **never pushes**, so the reviewer and the orchestrator can check the same sha range independently.

## The loop

1. **Plan** — the planner writes the plan file and writes no code.
2. **Implement** — the implementer receives only the plan path, then appends `## Result`.
3. **Verify** — the orchestrating session checks the Result against the tree (`git log`, the diff, rerun the tests) *before* spending a reviewer, then appends `## Verified` with exact evidence. A delegate's claim is not evidence.
4. **Review** — the reviewer receives the plan path and the sha range, and appends `## Review`.
5. **Close** — a blocking finding returns to step 2. Once clean, the outcome is recorded by the vault rules: the plan file stays in the repository, the vault gets the rationale, and any active `NOW.md` row is updated or retired.

## Worked examples

### The delegation loop

Task: *"The outbound export worker loses jobs when the provider times out. Fix it."*

**1. Invoke the loop and choose the actors.**

```bash
/delegate plan=claude:opus implement=pi review=codex:sol outbound export retries
```

Claude Code and Codex start in sibling panes, so this session stays usable:

```bash
herdr pane split --current --direction right --cwd "$PWD" --no-focus   # → w1:p7
herdr agent start planner --kind claude --pane w1:p7 -- --model opus
herdr agent prompt planner "Write docs/plans/outbound-export-retries.md per the plan contract." --wait --timeout 900000
```

**2. The planner writes the plan and no code:**

```markdown
# Outbound export retries

Status: Draft — retry budget still open

## Question
Does a provider timeout discard the job, and what budget should a retry get?

## Evidence
- `src/outbound/outbound.processor.ts:88` — the catch logs and returns, so Bull
  records the job as complete
- `docs/plans/qianyi-reliability.md` — the shared classifier calls timeouts transient

## Steps
1. Rethrow transient provider failures so Bull retries — acceptance: a unit test
   fails before the change and passes after
2. Cap the budget at 3 attempts with backoff — acceptance: no test exceeds 3 runs
```

**3. Implement in this session** (`implement=pi`), which appends:

```markdown
## Result
- `src/outbound/outbound.processor.ts` — classify, rethrow, discard on permanent
- `src/outbound/outbound.processor.spec.ts` — 4 tests, passing
- Commits: `a1b2c3d`, `d4e5f6a`
- Could not verify: the retry path against real provider timings
```

**4. Verify before spending a reviewer.** Read the diff, rerun the suite, then hand Codex the sha range:

```bash
herdr agent start reviewer --kind codex --pane w1:p9 -- -m gpt-5.6-sol
herdr agent prompt reviewer "Review a1b2c3d..d4e5f6a against docs/plans/outbound-export-retries.md. Append ## Review with blocking findings only." --wait
```

**5. Close.** With no blocking findings, the plan stays in the repository, the vault records that retry classification became shared policy, and the affected `NOW.md` row gets its next outcome instead of its old one.

### Focus-board reconciliation (`/now`)

The reconciliation reports drift as a table before touching anything:

| Row | Claims | Found | Verdict |
|---|---|---|---|
| Voucher release | production deploy unevidenced | Railway deployment `900fcbc4` SUCCESS on `main@2d26e93` | done |
| List UX | review frontend #309 | #309 merged 2026-09-09; backend #107 still open | stale |
| `staging` divergence | 4 files re-landed | `nixpacks.toml`, `.gitignore`, `ListingController` still differ | holds |

## The vault skill

Loads when a session reads vault notes, asks what is in progress, records a decision or lesson, or works inside the vault.

- **Read order** — `AI.md` → `NOW.md` → `Home.md` → project hub → smallest useful note.
- **Routing** — an ADR for a decision with rationale; `Inbox.md` for a loose thought; a `NOW.md` row for active work, a blocker, or a deadline; the project note or ADR for a durable outcome; a daily note only for a dated observation needing its own context.
- **Verify before acting** — a focus row is a claim, not a fact; check it against the source it names.
- **Editing rules** — preserve frontmatter, wikilinks, embeds, tags, callouts, and block IDs; link rather than copy; ask before deleting, bulk-renaming, or reorganising notes.

`NOW.md` is a focus board, not a task list or a log: it keeps only active work, a decision deadline, or a blocker that changes what happens next.

## Layout

```
├── skills/vault/SKILL.md                    # the skill (all four harnesses)
├── commands/now.md, retire.md, delegate.md, verify.md  # slash commands
├── docs/plans/                              # plan files the loop writes
├── package.json                             # pi package manifest
├── .claude-plugin/                          # Claude Code plugin + marketplace
├── .codex-plugin/                           # Codex plugin
└── .agents/plugins/marketplace.json         # Codex marketplace
```

## Configuration

The vault path (`~/Documents/windo-vault`) is written literally into `skills/vault/SKILL.md` and the four commands. Fork the repo and replace it to point at your own vault.

The package assumes the vault's layout: an `AI.md` entry point, a `NOW.md` focus board, `Home.md`, `Inbox.md`, project hubs, and `Meta/` notes. The vault's own `AGENTS.md` remains the authority on how its notes are edited.

The delegation loop assumes the repository keeps plans in `docs/plans/`. Change the path in `commands/delegate.md` if yours differ.

## Updating

pi, Claude Code, and Codex pin a ref, so an update means repointing it; opencode follows the checkout it is linked to, so it moves on `git pull`:

```bash
pi install git:github.com/hwindo/agent-vault@v0.3.0
claude plugin update agent-vault@agent-vault          # after the marketplace refreshes
codex plugin marketplace remove agent-vault           # Codex will not cross a ref in place
codex plugin marketplace add hwindo/agent-vault --ref v0.3.0
codex plugin add agent-vault@agent-vault
```

## Uninstalling

```bash
pi remove git:github.com/hwindo/agent-vault
claude plugin uninstall agent-vault@agent-vault
codex plugin remove agent-vault@agent-vault
```
