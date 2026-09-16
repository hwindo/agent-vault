---
description: Independently verify completed agent work before calling it done
argument-hint: "[plan path or task]"
---
Run an independent completion check for: $ARGUMENTS

With no argument, check the work just completed in this session.

Do not trust the agent's summary as evidence. Re-read the request and, when present, the repository plan. Then:

1. Inspect `git status`, the final diff, and the relevant commit range. Confirm the change is scoped and matches the request or plan.
2. Rerun the relevant tests, lint, type checks, and build against the final tree. Prefer a regression test that fails against the unfixed code when practical.
3. Exercise the behavior that source checks cannot prove: an integration, browser, mobile, deployment, or other runtime path when the task needs it. If it cannot be exercised, say exactly why.
4. Report the exact evidence and every remaining gap. Never turn “not checked” into “passed.”
5. For a plan-based task, append `## Verified` to the plan using the evidence above. Do not overwrite `## Result` or `## Review`.

Only after this may the work be described as verified or handed to a reviewer.
