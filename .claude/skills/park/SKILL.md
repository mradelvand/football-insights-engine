---
name: park
description: Save a side idea to docs/parking-lot.md so it doesn't derail the current step. Use when CESC types /park or drifts into a new idea mid-task ("what if we also…", "later I want…").
argument-hint: [idea]
---

# Park an idea

1. Take the idea from `$ARGUMENTS`. If it's empty, ask for it in one line.
2. Append `- YYYY-MM-DD: <idea>` to `docs/parking-lot.md` (today's date from `date +%F`).
3. Reply in one line: "Parked. Back to <id> · <title>." using the current open step from `docs/PLAN.md`.

Don't discuss, plan or start the idea.
