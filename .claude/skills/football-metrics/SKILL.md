---
name: football-metrics
description: Event schema, StatsBomb conversion rules, pitch coordinates, match clock, metric definitions (possession, field tilt, PPDA, momentum, xG), Elo, form and head-to-head, insight types and the fact-card format for this project. Use when writing or reviewing code in src/ingest, src/interpret, src/explain, or tests for them.
user-invocable: false
---

# Football data conventions

Source: StatsBomb open data, Premier League 2015/16 (`competition_id` 2, `season_id` 27). Licence and repo rules: `docs/DATA.md`.

## Event schema (`src/ingest/schema.py`)

Dataclass `Event`: `id, sb_id, match_id, period, minute, second, team, player, type, x, y, end_x, end_y, outcome, play_pattern, body_part, shot_type, xg`.

- `id` = StatsBomb `index` (readable running order; sort by it). `sb_id` = the StatsBomb UUID.
- Types: `pass, corner, shot, tackle, interception, clearance, block, pressure, foul, own_goal`.
- Converted matches are JSON Lines in `data/match_<match_id>.jsonl` (git-ignored, never committed).

## StatsBomb → Event (`src/ingest/statsbomb_adapter.py`)

- Pass → `pass`; pass.type "Corner" → `corner`. pass.type "Free Kick" / "Throw-in" stay `pass` with that `play_pattern`.
- Shot → `shot`, with `xg` = `shot.statsbomb_xg`, `body_part` = `shot.body_part`, `shot_type` = `shot.type` ("Open Play", "Free Kick", "Penalty"), `outcome` = `shot.outcome` ("Goal", …).
- Duel with duel.type "Tackle" → `tackle`. Interception, Clearance, Block, Pressure → same name. Foul Committed → `foul`. Own Goal For → `own_goal`.
- `play_pattern` (Regular Play, From Corner, From Free Kick, From Throw In, From Counter, From Kick Off, From Goal Kick, From Keeper, Other) applies to the whole move, not just the restart.

## Coordinates

- Pitch is 120 × 80. Coordinates are **team-relative**: from the acting team's view, attacking toward x = 120.
- Final third: x > 80. Own defensive 60%: x < 72.
- The same spot is `x` for one team and `120 − x` for the other. Convert explicitly whenever you compare teams.

## Match clock

StatsBomb `minute` counts minutes already played; TV shows the minute in progress.
- Regular time: show `minute + 1`. 4:58 → `5'`.
- Period 1 and minute ≥ 45 → `45+n'`, n = minute − 44.
- Period 2 and minute ≥ 90 → `90+n'`, n = minute − 89.
- Timestamps restart each half; second-half minutes start at 45.

## Score, xG, set pieces

- **Score:** shots with outcome "Goal" plus `own_goal` events (Own Goal For, credited to the team that benefits).
- **xG:** `shot.statsbomb_xg` only. No home-made xG.
- **Set-piece goal/shot:** shot_type "Free Kick", or play_pattern "From Corner" / "From Free Kick". Penalties are their own category. Throw-in goals are counted separately, not as set pieces.

## Metrics (pure functions: events in, numbers out)

- **Possession %:** each team's share of on-ball events in the last 5 match minutes.
- **Field tilt:** each team's share of all final-third passes (x > 80, in the passer's coordinates).
- **PPDA:** opponent passes with x < 72 (their own defensive 60%, in their coordinates) ÷ our tackles + interceptions with x > 48 (in our coordinates). Lower = harder pressing. With zero defensive actions, return `None`; detectors skip `None`.
- **Momentum:** rolling count, per team, of final-third entries + shots over the last 5 minutes.
- **Season baselines:** only matches played on dates BEFORE the replayed match. No future data.

## Elo, form and head-to-head (season baselines, w4-0)

- Every club starts at 1500 before matchweek 1 of 2015/16.
- Process earlier matches in order of `match_date`, then `kick_off`, then `match_id`. Only dates BEFORE the replayed match count; same-day matches are excluded.
- Expected score for the home team: `E = 1 / (1 + 10 ** ((away_elo - (home_elo + 60)) / 400))`. The 60 is home advantage.
- Result S: home win 1, draw 0.5, loss 0. Update with K = 20: `home += 20 * (S - E)`, `away -= 20 * (S - E)`.
- **Elo favourite:** the home team if E ≥ 0.5, otherwise the away team.
- **Form:** last 5 league results per team, oldest to newest (W/D/L). **Unbeaten run:** consecutive matches without a loss (winless run likewise).
- **Head-to-head:** earlier meetings this season, with date and score.
- Saved per team, per date in the same small season summary file as the other baselines.
- **Wording rule:** Elo gives an expected score, not a win probability. Text may say "Elo favourites" or "ranked 1st on Elo"; it must never say "a 76% chance of winning". The verifier rejects it as an unbacked comparison.

Test values (356 matches before 2016-05-07; Elo to the nearest whole number, ±1 is fine):
- Elo: Leicester City 1624 (1st), Tottenham Hotspur 1602 (2nd), Arsenal 1571 (3rd), Everton 1484 (11th).
- Expected score, Leicester at home to Everton: 0.76.
- Last 5, oldest to newest: Leicester W W D W D; Everton D D D L W.
- Leicester unbeaten in 10 before this match.
- Earlier meeting: 2015-12-19, Everton 2–3 Leicester City.

## Insight types (`src/interpret/detectors.py`)

Thresholds are starting values to tune.

| Insight | Fires when | Priority |
|---|---|---|
| Goal in context | Any goal (Shot outcome "Goal" or Own Goal For) | 10 |
| Late drama | A goal, or a shot with xG ≥ 0.30, shown at 85' or later in the second half | 10 |
| Big chance | A shot with xG ≥ 0.30 | 9 |
| Set-piece threat | A team's 3rd set-piece shot within 15 minutes | 8 |
| Pressing surge | A team's PPDA over the last 10 minutes is half or less of the 10 minutes before | 6 |
| Momentum swing | Over 5 minutes, one team's final-third entries + shots overtake the other's by 3 or more | 6 |
| Upset watch | The team that was NOT the Elo favourite takes the lead. Once per lead change | 7 |
| Defensive wall | A player reaches 10 defensive actions (tackles, interceptions, clearances, blocks) | 5 |

**Pre-match context** is shown at kickoff and half-time, outside the live queue. Facts: Elo rank for both teams, the Elo favourite, both teams' form, unbeaten or winless runs of 5 or more, earlier meeting this season. Example: "Leicester come in top of the Elo table and unbeaten in 10. They won the reverse fixture 3–2 at Everton in December."

Upset watch never fires in match 3754033 (Leicester, the favourites, lead from 5'). Test it with hand-made events where the underdog scores first, and with the second demo match below.

## Fact cards

Shape: `{card_id, type, priority, clock, team, player, facts: {...}, compare: {...}, evidence_ids: [event indexes]}`

```python
{"card_id": "c017", "type": "goal", "priority": 10, "clock": "65'",
 "team": "Leicester City", "player": "Jamie Vardy",
 "facts": {"score": "3-0", "shot_type": "Penalty", "xg": 0.78, "player_league_goals_season": 24},
 "compare": {"basis": "season to date, earlier matches only"},
 "evidence_ids": [2416, 2418]}
```

- `compare` is one of: this match so far, the opponent, or the season to date (earlier matches only).
- `evidence_ids` are the `Event.id` (StatsBomb index) values behind the card.
- Every number in `facts` must be recomputable from those events or the season summary. The verifier relies on this.

## Test moments in match 3754033 (Leicester 3–1 Everton)

| Shown | Index | Raw time | What | xG |
|---|---|---|---|---|
| 5' | 240 | 4:58 | Vardy goal (big chance) | 0.37 |
| 33' | 1324 | 32:12 | King goal | 0.16 |
| 65' | 2418 | 64:49 | Vardy penalty, after Everton foul at 2416 | 0.78 |
| 88' | 3115 | 87:27 | Mirallas goal from a counter (late drama) | 0.35 |

- Final score in the converted file: 3–1. 42 shots; 3,269 events.
- Season check: Vardy had 22 league goals before 2016-05-07, so the 65' penalty must be his 24th. If not, the "earlier matches only" rule is broken.

## Second demo match for Upset watch: 3753976, Arsenal 1–2 Swansea City (2016-03-02)

Pre-match Elo: Arsenal favourites, expected score 0.72 at home (the biggest upset of the season by Elo).

| Shown | Index | Goal | Upset watch |
|---|---|---|---|
| 15' | 733 | Campbell, Arsenal (xG 0.21) | no (favourite leads) |
| 32' | 1537 | Routledge, Swansea, from a counter (xG 0.28) | no (level) |
| 74' | 3201 | Ashley Williams, Swansea, from a free kick (xG 0.50) | **fires once** |
