---
description: Reconcile the vault focus board against repo reality, then work one thread
argument-hint: "[thread to work on]"
---
Run the focus loop on `~/Documents/windo-vault/NOW.md` (load the `vault` skill if it is not already loaded):

1. Read `NOW.md`.
2. Verify every row against the source it names — `git log`, `gh pr list`/`view`, deploy state, the plan doc. One check per claim, and say what the evidence was.
3. Report drift as a table: row, what it claims, what you found, verdict (holds / stale / done). Edit nothing yet.
4. Work one thread: `$ARGUMENTS` if given, otherwise ask which row after the report.
5. Rewrite the rows the work changed — retire the ones whose outcome now lives in the project note, and give the rest a next concrete outcome or blocker. Keep it a focus board, not a log.
