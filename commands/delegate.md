---
description: Run plan → implement → verify → review with the actors you choose, then close the loop
argument-hint: "[plan=<actor> implement=<actor> review=<actor>] <task>"
---
Run the delegation loop for: $ARGUMENTS

With no task given, ask for one before starting.

## Choose the actors

Parse `plan=`, `implement=`, `review=` from the arguments. Omit a role and ask for it before starting — never infer it from the task size.

| Actor | Runs as |
|---|---|
| `codex:sol`, `codex:luna` | Codex in a sibling herdr pane: `--kind codex -- -m gpt-6.2-sol` (or `gpt-6-luna`); a bare `sol` is not a valid model id |
| `claude:opus`, `claude:sonnet`, `claude:haiku` | Claude Code in a sibling herdr pane: `--kind claude -- --model <alias>` |
| `pi` | this session |
| `pi:<model>` | a pi subagent on that model, if the subagent extension is installed; otherwise say so and offer to install it |

House defaults: **plan `claude:opus` · implement `claude:sonnet` · review `codex:sol`**. When this session is already running Opus, it is the planner itself (`pi`) instead of starting a fresh Opus pane. Utility work — summarising, renaming, extraction, boilerplate, commit messages — goes to `codex:luna`, `claude:haiku`, or local `qwen3:8b`, and never occupies a loop role.

## The loop

1. **Plan** — the planner writes `docs/plans/<slug>.md`: the question, evidence inspected **with file and line numbers**, working assumptions, rejected paths, the decision still needed, and a bounded step list where each step names its acceptance check. It writes no code. Evidence without an anchor is not evidence; the implementer must never have to re-derive the exploration.
2. **Implement** — hand the implementer only the plan path, plus the contract: commit small and feature-scoped, never push, then append `## Result` to the plan — what changed, commit shas, tests run, and what it could not do.
3. **Verify** — check the Result against the tree yourself before spending a reviewer: `git log`, the diff, and rerun the tests. A delegate's claim is not evidence. Append `## Verified` to the plan with the exact commands, files, and runtime or UI checks that passed. If something could not be checked, record it explicitly instead of calling the work verified.
4. **Review** — hand the reviewer the plan path and the sha range. It appends `## Review`: blocking findings only, each with file and line.
5. **Close** — a blocking finding returns to step 2. Once clean, record the outcome by the `vault` rules: the plan file is the repository's copy, the vault gets the rationale, and any active `NOW.md` row is updated or retired.

## Completion invariant

An implementer's `## Result` is a report, not proof. The delegation is not complete until the orchestrating session has independently verified the tree and appended:

```markdown
## Verified
- `git diff <base>..<head>` — reviewed; scope matches the plan
- `<test/build command>` — passed
- `<runtime, integration, or UI check>` — passed, or explicitly not run
- Remaining unverified: <honest gap, or none>
```

The reviewer must receive the verified sha range. A green test suite that was not rerun against the final tree, or a plan that only repeats the implementer's claim, does not satisfy this invariant.

## Delegating

Start each delegate in a sibling pane with `--no-focus` so this session stays usable, and give it a unique name. Read its work from the plan file, not the pane: long output can leave the alternate screen, and scrollback will not bring it back. Check `herdr agent list` before starting, so two agents are not handed the same file.
