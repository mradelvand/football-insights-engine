---
name: next
description: Show CESC's next step from docs/PLAN.md, one step only, as Do / Done when / Why. Use when he types /next, asks "what's next" or "what should I do now", or starts a session.
---

# Next step

1. Read the last entry in `docs/devlog.md` and note its "Next:" line.
2. Read `docs/PLAN.md`. Find the first step that is `- [ ]`, is not tagged `optional`, and has no `STUCK` line under it.
3. Show ONLY that step, in exactly this shape, with nothing before it:

   **<id> · <title>** · <time> · <by hand | with Claude>

   **Do**
   1. ...

   **Command** (only if the step has one, in a code block)

   **Done when:** ...

   **Why:** ... (2 lines max)

4. Then one line, depending on the tag:
   - by hand: "This one's yours. Say done when it's finished, or /stuck if it fights back."
   - with Claude: "Coach me or pair on it?" (skip the question if CLAUDE.md sets a default mode and he hasn't said otherwise this session).
5. If today (`date +%F`) is after the end date of that step's week, add one line: how many required steps are overdue, and "Run /status for the cut list."
6. If no required step is open, say whether only STUCK steps remain or the plan is finished, and suggest /status.

Never show more than one step. Never start the step before he answers.
