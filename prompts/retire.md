---
description: Verify a focus thread's outcome, record it durably, and retire its NOW.md row
argument-hint: "<thread>"
---
Retire the `$ARGUMENTS` thread from `~/Documents/windo-vault/NOW.md`:

1. Verify the row's remaining claims against their sources. If any outcome is unevidenced, name what is missing and stop — a thread retires on evidence, not on intent.
2. Write the durable outcome into the project note or ADR the row links: what shipped, what was decided, what stays open. Link rather than copy.
3. Remove the row from `NOW.md` and bump its `updated` frontmatter.
4. Report what moved and where it now lives.
