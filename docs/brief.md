# AI Match Commentator: project brief

**In one line:** an AI match commentator that replays a real Premier League match one event at a time and puts short insights on screen about what UK fans already talk about (chance quality, set pieces, pressing, late drama), with every number, name and comparison checked before anyone sees it.

**The problem.** Broadcasters get data for every pass, shot and tackle within seconds. Turning it into "why this moment matters" while the match is live still takes human analysts. AI can write those sentences, but it sometimes invents numbers, and a wrong stat can't go on air.

**The idea.**
1. Code works out the facts, each with a comparison: this match, the opponent, or the season so far. It's deterministic and tested.
2. Code picks what's worth saying, using priorities and cooldowns.
3. AI only writes the wording, with slots that code fills with the real values.
4. Code checks every number, name and comparison. If the check fails twice, or the AI takes longer than 1.2 seconds, a template line goes out instead.

If you know monitoring, it's the same shape:

    telemetry → metrics → alert rules + dedup → alert message → validation → dashboard
    events    → stats   → detectors + queue  → AI sentence   → verifier   → studio screen

**The data.** StatsBomb open data, Premier League 2015/16: 380 matches, the season Leicester won the league. The demo match is Leicester City 3–1 Everton, 7 May 2016. Season comparisons use only matches played before the replayed one. Data credit: StatsBomb, with their logo in this README and in the app. No raw or converted event files are in this repo, and the project is not for commercial use. Details in docs/DATA.md.

**What it talks about**

| Insight | Fires when | Priority |
|---|---|---|
| Goal in context | Any goal | 10 |
| Late drama | A goal or big chance from 85' | 10 |
| Big chance | A shot with xG of 0.30 or more | 9 |
| Set-piece threat | A team's 3rd set-piece shot in 15 minutes | 8 |
| Upset watch | The team that wasn't the Elo favourite takes the lead | 7 |
| Pressing surge | PPDA halves against the previous 10 minutes | 6 |
| Momentum swing | One team's threatening actions overtake the other's over 5 minutes | 6 |
| Defensive wall | A player reaches 10 defensive actions | 5 |

Before kickoff and at half-time, a **pre-match context** card adds the form guide: Elo rank, the Elo favourite, last 5 results, unbeaten runs and the earlier meeting this season. Elo uses only matches played before the replayed one.

(PPDA = how many passes the other team makes before your team wins or contests the ball. Lower means harder pressing.)

**Selection, wording and checks**
- Queue: merge cards about the same event; show the highest priority; at most one insight per match minute; a 5-minute cooldown per player and per topic. Goals skip the queue and the cooldown.
- Narrator: returns JSON wording with named slots, e.g. `{player_surname}`, `{xg_pct}`, `{goals_season_ord}`. Code fills the values. The model never types numbers.
- Verifier (plain code) rejects text that has: a number or number word ("fourth", "twice", "hat-trick") that didn't come from a slot; a name that isn't in the card (surname alone is allowed); a comparison ("most", "first since", "season high") with no `compare` field behind it, including any win probability drawn from Elo (Elo gives an expected score, so "Elo favourites" is fine but "a 76% chance of winning" is not); a word about a player's body or intelligence ("powerful", "pacy", "clever") with no card field behind it (also a StatsBomb licence rule).
- Fallback: one template line per card type. The AI gets 1.2 seconds of real time and one retry, then the template goes out. Logged as `source: ai` or `source: template`.

**One insight, end to end (real data)**

    Event 2418, 64:49, shown as 65'   Jamie Vardy scores a penalty (xG 0.78) after Everton's foul (event 2416)
    Fact card   {type: goal, priority: 10, clock: "65'", team: "Leicester City", player: "Jamie Vardy",
                 facts: {score: "3-0", shot_type: "Penalty", xg: 0.78, player_league_goals_season: 24},
                 compare: {basis: "season to date, earlier matches only"}, evidence_ids: [2416, 2418]}
    AI wording  "{player_surname} scores from the spot: his {player_league_goals_season_ord} league goal of the season."
    On screen   "Vardy scores from the spot: his 24th league goal of the season."
    Verifier    Vardy ✓  24th ✓  no other numbers, names or comparisons ✓
    Fallback    GOAL 65' · Jamie Vardy (penalty) · Leicester City 3–0 · 24th league goal this season

**The pipeline**

| Stage | In → out | Built with | AI? |
|---|---|---|---|
| 1. Fetch | StatsBomb files → data/raw/ (git-ignored) | Python | No |
| 2. Ingest | StatsBomb events → our Event format → one event at a time, "live" at 20× speed | Python adapter + generator | No |
| 3. Interpret | Rolling stats (5 min, 15 min, half, match) → fact cards with a comparison → priority queue | Python + pytest | No |
| 4. Explain | Top card → wording with slots → filled and verified, or a template line | Microsoft Foundry Narrator agent (one read-only tool) + code verifier; an eval gate picks the model | Only here |
| 5. Render | Insight → timed overlay JSON with the match clock | Python | No |
| 6. Personalize | Same facts → analyst or casual-fan wording, in English | Narrator + Streamlit page | Via the Narrator |

**The platform**
- The development machine is one file (the dev container), so it can be rebuilt identically any time.
- The app runs in Docker on Azure Container Apps.
- It has its own managed identity with the Foundry User role, so there are no API keys anywhere in this public repo.
- Logs feed Log Analytics: time from event to screen, AI usage per insight, verifier pass rate and template fallback rate.
- A budget alert guards the cost.

**Milestones**
- Nov 1, 2026: a local web page that replays the match with live stats and template insights. No AI yet.
- Nov 22, 2026: AI wording, running on Azure, plus a video under 2 minutes and a blog post.

**What it is not:** an official entry to the Microsoft × Premier League "Inside the Game" hackathon (Québec residents can't enter), a commercial product (the StatsBomb licence forbids it), a replacement for commentators (it's closer to the stats desk that feeds them), or a machine-learning research project. It's a portfolio piece aimed at platform and data-engineering roles in sports tech.
