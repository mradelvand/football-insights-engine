---
name: football-metrics
description: Event schema, pitch coordinates, metric definitions (possession, field tilt, PPDA, momentum, simple xG), detectors and the fact-card format for this project. Use when writing or reviewing code in src/sim, src/ingest, src/interpret, src/explain, or tests for them.
user-invocable: false
---

# Football data conventions

## Event schema (`src/sim/schema.py`)

Dataclass `Event`: `id, match_id, minute, second, team, player, type, x, y, end_x, end_y, outcome`.
Types in v0: `kickoff, pass, shot, goal, tackle, interception`. Matches are stored as JSON Lines (one event per line) in `data/match_XXX.jsonl`.

## Coordinates

- Pitch is 105 × 68 m.
- Coordinates are **team-relative**: from the acting team's view, attacking toward x = 105.
- Final third: x > 70. Own defensive 60%: x < 63.
- The same spot is `x` for one team and `105 − x` for the other. Convert explicitly whenever you compare teams.

## Metrics (pure functions: events in, numbers out)

- **Possession %:** each team's share of on-ball events in the last 5 match minutes.
- **Field tilt:** each team's share of all final-third passes (x > 70, in the passer's coordinates).
- **PPDA:** opponent passes in their own defensive 60% (x < 63 in the opponent's events) ÷ our tackles + interceptions in that same zone (x > 42 in our events). Lower = harder pressing. With zero defensive actions, return `None`; detectors skip `None`.
- **Momentum:** rolling count, per team, of threatening events (final-third entries and shots) over the last 5 minutes.
- **Simple xG:** goal probability that falls with distance to goal and with a narrower shooting angle. Close, central shots score highest; shots from 25 m+ stay at a few percent.

## Fact cards

```python
{"type": "momentum_swing", "minute": 63, "team": "Riverside Rovers",
 "metrics": {"momentum_for": 7, "momentum_against": 2},
 "evidence_ids": ["e0412", "e0415", "e0419"]}
```

- `evidence_ids` are the `Event.id` values behind the card.
- Every number in `metrics` must be recomputable from those events or the match state. The verifier relies on this.

## Detectors (`src/interpret/detectors.py`)

- **Momentum swing:** one team's momentum overtakes the other's by a threshold.
- **Shot cluster:** 3+ shots by one team within 5 minutes.
- **Pressing surge:** a team's PPDA drops sharply compared with the previous 5-minute window.
- Later (parking lot): flank overload, player involvement spike, set-piece threat.

## Simulator sanity targets (one 90-minute match)

Hundreds of passes, 10–30 shots, 0–6 goals, pass completion in a believable range. Use `random.seed(42)` so a match can be replayed exactly. Clubs and players are always fictional.
