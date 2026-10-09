# Data: StatsBomb open data

Data provided by **StatsBomb**. Their logo (from statsbomb.com/media-pack) must appear on anything published: the README and the app.

## What we use (verified Oct 9, 2026)

- **Competition:** Premier League 2015/16, `competition_id` 2, `season_id` 27. 380 matches, 20 clubs. Full season of events ≈ 1.13 GB.
- **Demo match:** Leicester City 3–1 Everton, 2016-05-07, `match_id` 3754033, matchweek 37, 3,269 events. 356 matches were played before that date.
- **Pitch:** 120 × 80. Locations are from the acting team's view, attacking toward x = 120.
- **IDs and order:** `id` is a UUID string; `index` is the readable running order (sort by it). `period` is the half. Second-half minutes start at 45, and timestamps restart each half.
- **Extras:** shots carry `freeze_frame` (player positions at the shot) on 41 of 42 shots in the demo match; lineups exist. No 360 data for this season.

## Licence summary (LICENSE.pdf, updated 8 Sep 2023)

- **Allowed:** analysis, research, publishing conclusions.
- **Not allowed:** redistributing the data in any form, including converted copies; commercial use of the data or analysis; anything defamatory or damaging to any individual.
- **Required:** the StatsBomb logo on anything published (README and app).

## What that means in this repo

- `data/raw/` and every converted event file (`data/*.jsonl`) stay git-ignored and out of public Docker images.
- Commit only small aggregate summaries (team and player totals, league ranks by date).
- The AI only ever receives fact cards, never raw event files.
- Players are described only through facts in the card (bias guard = licence rule).
- Season baselines use only matches played before the replayed match.
- Not for commercial use.
