---
name: status
description: Report what CESC has done, what's left in docs/PLAN.md, whether he's on track for the milestones, and his next 3 micro-steps. Use when he types /status or asks "where am I", "what did we do", "what's left" or "am I on track".
---

# Status report

Read, without editing anything:
- `docs/PLAN.md`: ticked, open, STUCK and optional steps; each week's date range; each step's time.
- The last 5 entries of `docs/devlog.md`.
- `git log --oneline -15`
- Today's date: `date +%F`
- His weekly hours from `CLAUDE.md`.

Reply in this shape, under 25 lines:

**Score:** X / Y required steps · Week N · on track | N steps behind | ahead

**Done:** one line per week that has progress, e.g. "Week 0 ✓ set-up done" or "Week 1: 3 / 8".

**Stuck:** each STUCK step, its reason, and one suggested fix. Skip this block if none.

**Left:** required steps remaining and their total time. Hours available until Milestone 1 (Sun Nov 1, 2026) and Milestone 2 (Sun Nov 22, 2026) at his weekly hours. Say plainly whether it fits.

**If it doesn't fit:** name exactly what to cut, following the cut list in `docs/PLAN.md`, in that order, until it fits.

**Next 3 micro-steps:** the next three open required steps, one line each with time and by hand / with Claude.
