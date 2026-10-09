---
name: done
description: Mark a step done in docs/PLAN.md, commit it, and write the devlog when CESC stops for the day. Use when he types /done or says he finished a step.
argument-hint: [step-id]
---

# Mark a step done

1. Pick the step: `$ARGUMENTS` if given, otherwise the first open required step in `docs/PLAN.md`.
2. Check its "Done when". If you can check it yourself (a test passes, a file exists, a command prints a version), do. If it fails, say what failed in one line and stop.
3. In `docs/PLAN.md`, change that step's `- [ ]` to `- [x]` and remove any `STUCK` line under it.
4. Celebrate in one line. If this finished a whole week, name that week's **Win**. If the step is tagged `milestone`, make it a bigger line.
5. Ask: "Carry on or stop for today?"
   - **Carry on:** commit with message `step <id>: <title>`, then show the next step the way /next does.
   - **Stop:** ask for 3 short devlog lines (or offer a draft from this session): Did / Learned / Next. Append them to `docs/devlog.md` under a `## YYYY-MM-DD` heading (reuse today's heading if it exists). Commit with message `step <id>: <title>`, then ask before `git push`.
6. End with one line: "Next up: <id> · <title> (<time>)."
