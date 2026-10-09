---
name: stuck
description: Help CESC with a blocking step briefly, then park it and move on. Use when he types /stuck, says he's stuck or frustrated, or has spent 20+ minutes on one error.
argument-hint: [what's wrong or the error text]
---

# Stuck

1. Pick the step: the one he names, otherwise the first open required step in `docs/PLAN.md`.
2. Get the problem from `$ARGUMENTS`. If it's empty, ask one question: "Paste the error or tell me what's happening."
3. Say the most likely cause in one plain sentence.
4. Offer at most 3 fixes, smallest first, one line each. Offer to do the first one with him.
5. If he'd rather park it (or the fixes don't work): add a line under that step in `docs/PLAN.md`:
   `  - STUCK (YYYY-MM-DD): <one-line reason>`
   Keep its box `- [ ]`.
6. Say "Parked. It'll wait." and show the next step the way /next does.

Keep the whole reply under 15 lines. No blame, no lecture.
