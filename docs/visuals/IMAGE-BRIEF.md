# Image brief: overview + 18 slides

One file to hand to another Claude session. It holds the kickoff prompt, the facts the images must use, a spec for each image, the file names, and the checklist used to verify the result.

**How to use it**
1. Open a new Claude session that can write HTML and run a headless browser (Claude Code, or claude.ai with code execution).
2. Paste **section 1 (kickoff prompt)**, then paste this whole file below it (or, in Claude Code inside this repo, just paste section 1: it reads this file itself).
3. Bring the PNGs back to the project session. They get checked against **section 6** and uploaded to the paths in **section 5**.

---

## 1. Kickoff prompt (copy from here to the end of the code block)

```text
You are producing presentation graphics for a football analytics portfolio project.
The full brief is in docs/visuals/IMAGE-BRIEF.md (pasted below if you can't read the repo).

Make 19 images: the README overview (section 3) and slides 01–18 (section 4).

HOW
- Build each image as a self-contained HTML page with inline CSS, using the design system in section 2.
  Exact text matters more than artistry: every label, number and name must match the brief and the facts in section 2.
- Render each page to PNG with a headless browser (Playwright/Chromium) at the size in section 5, device scale factor 1.
- Use only the facts in section 2. If a number isn't there, use the placeholder "__". Never invent statistics.
- No club crests, no Premier League logos, no player photos. Where the brief says "StatsBomb logo", draw an empty dashed box with that label.
- Icons: simple line icons drawn in inline SVG (no external icon fonts).

DELIVER
- The PNGs with the exact file names in section 5, plus the HTML source of each in a folder named src/.
- A short table: file name, size in pixels, anything you couldn't fit or changed.
- Do not commit or push to any repository. The project owner reviews the images first.

Work in this order: shared CSS → overview → slides 01–18. After the overview, stop and show it before doing the slides.
```

---

## 2. Design system and facts

### Style (every image)
- Dark, broadcast-studio look. Background `#0b0f0e`. Cards `#121a17`, border `#22302a`, 12 px radius.
- Text `#e6ece9`, muted `#8fa39a`. Accents: green `#2fd17a` (primary, "pass", code), yellow `#f5c84b` (numbers, milestones), blue `#4aa3ff` (AI / Foundry), pink `#d9709c` (Azure Container Apps), red `#ff6b6b` (rejects).
- Fonts: "Barlow Condensed" bold for titles, "Inter" for body, "JetBrains Mono" for code and IDs (Google Fonts).
- Slides: small green uppercase section label top-left, large title, rounded cards, coloured pill tags, thin arrows, generous spacing, slide number bottom-right, footer left `github.com/mradelvand/football-insights-engine`.
- Footer on the overview and slide 01: "Independent portfolio project inspired by the Microsoft × Premier League 'Inside the Game' brief. Not an official entry. Data: StatsBomb open data, not for commercial use."

### Facts (the only numbers allowed)
- **Data:** StatsBomb open data, Premier League 2015/16, 380 matches. Licence: research use, StatsBomb logo required, no redistribution, non-commercial.
- **Demo match:** Leicester City 3–1 Everton, 7 May 2016, match_id 3754033, 3,269 events, 42 shots.
- **Goals:** 5' Vardy (event 240, xG 0.37) · 33' King (event 1324, xG 0.16) · 65' Vardy penalty (event 2418, xG 0.78, after Everton's foul, event 2416) · 88' Mirallas from a counter (event 3115, xG 0.35).
- **Season context:** Vardy had 22 league goals before 7 May 2016, so the 65' penalty is his **24th**.
- **Clock rule:** StatsBomb 64:49 is shown as 65'. Added time shows as 45+n / 90+n.
- **Elo before the match** (start 1500, home +60, K = 20, earlier matches only): Leicester 1,624 (1st), Tottenham 1,602 (2nd), Arsenal 1,571 (3rd), Everton 1,484 (11th). Leicester's expected score at home: 0.76. Elo is a rating, not a win probability.
- **Form, oldest to newest:** Leicester W W D W D (unbeaten in 10) · Everton D D D L W. Reverse fixture: 19 Dec 2015, Everton 2–3 Leicester.
- **Second demo match (upset):** Arsenal 1–2 Swansea City, 2 Mar 2016, match_id 3753976. Arsenal's expected score 0.72. Campbell 15' (xG 0.21) · Routledge 32', counter (xG 0.28) · Ashley Williams 74', free kick (event 3201, xG 0.50): Upset watch fires once, here.
- **Insight types (priority):** Goal in context 10 · Late drama (goal or xG ≥ 0.30 from 85') 10 · Big chance (xG ≥ 0.30) 9 · Set-piece threat (3rd set-piece shot in 15 min) 8 · Upset watch (Elo underdog takes the lead) 7 · Pressing surge (PPDA halves vs previous 10 min) 6 · Momentum swing 6 · Defensive wall (10 defensive actions) 5. Plus **Pre-match context** at kickoff and half-time (Elo rank, favourite, form, unbeaten runs, head-to-head).
- **Queue:** max 1 insight per match minute · 5-minute cooldown per player and topic · goals skip the queue.
- **AI rules:** the model writes wording with slots (`{player_surname}`, `{goals_season_ord}`); code fills values. Verifier is plain code. Budget 1.2 s, 1 retry, then a template line. 0 API keys.
- **Verifier rejects:** unslotted numbers or number words ("his fourth hat-trick") · names not in the card · comparisons with no backing ("season high") · win probabilities from Elo ("a 76% chance of winning") · body or intelligence words ("pacy", "clever").
- **Receipts per insight:** status (AI live / verified N numbers / template), model, latency, tokens, est. cost, trace ID, evidence ID under every number.
- **Eval gate:** ~20 fact cards × 3 runs. Gates: verified first time ≥ 90% · template fallback ≤ 10% · p95 ≤ 1.2 s · 0 content-filter blocks. The cheapest passing deployment serves; switch with `AI_MODEL_DEPLOYMENT`.
- **Data guard:** `data/README.md` with SHA-256 checksums; `verify` checks schema contract + checksums before ingest; pre-commit hook blocks `data/raw` and `*.jsonl`. Data version `dv-…` (hash of inputs) on every card. Committed: only `data/season_baselines.json`.
- **Platform:** GitHub Codespaces dev container (Python 3.12, Azure CLI, Docker, Claude Code) → Docker image → Azure Container Apps, deployed with **Bicep** (`infra/main.bicep`) → **user-assigned managed identity** with Foundry User (Microsoft Foundry) and AcrPull (Container Registry) → key auth off on Foundry and Application Insights, registry admin off, ingress IP allow-list, scale 0–2 → **Application Insights via OpenTelemetry** (spans per stage, GenAI attributes, data version, image digest; prompts and responses never logged) → budget alert USD 30/month.
- **Personalization:** analyst or casual-fan wording, English only.
- **Timeline:** Oct 8, 2026 start · **Nov 1** Milestone 1: local demo, template insights, no AI · **Nov 22** Milestone 2: Azure + AI wording + video under 2 min + blog.
- **Not in the product (don't draw):** a second "Analyst" agent, French output, AKS, multiple services, Parquet/DuckDB.

---

## 3. README overview (replaces `docs/images/overview.webp`)

```text
Size 2400 × 1350. Title: "Football Insights Engine — an AI match commentator you can trust" (second half in green).
Subtitle: "Replays a real Premier League match event by event. Code computes the facts, AI writes the words, code checks every word."

SECTION 1 · THE PIPELINE (full-width row of 6 cards with arrows, each with an icon and number):
01 FETCH + VERIFY: "StatsBomb open data, PL 2015/16 → data/raw (never committed) · SHA-256 + schema check · data version dv-…"
02 INGEST: "Events → our format → replayed one by one at 20× speed · TV clock (64:49 → 65')"
03 INTERPRET: "Live stats: possession, field tilt, PPDA, momentum, xG, Elo → fact cards → priority queue"
04 EXPLAIN (green border, pill "THE ONLY AI STAGE"): "Narrator agent writes wording with slots like {player_surname}; code fills the real values"
05 VERIFIER (pills "pass" / "reject"): "Plain-code checks: numbers, names, comparisons, bias words. Fail twice or > 1.2 s → template line"
06 RENDER + PERSONALIZE: "Timed on-screen overlay with receipts · analyst or casual-fan wording"
Dashed strip under the row: "IF YOU KNOW MONITORING  telemetry → metrics → alert rules → alert message → validation → dashboard".

SECTION 2 · ONE INSIGHT, END TO END (left, stacked rows with labels EVENT / FACT CARD / AI WORDING / VERIFIER / RECEIPTS):
EVENT "Event 2418 · 64:49 → shown as 65'" · FACT CARD chips "goal" "Jamie Vardy" "penalty" "xG 0.78" "24th league goal" "dv-…", note "built by code; season count uses earlier matches only" ·
AI WORDING mono "{player_surname} scores from the spot: his {goals_season_ord} league goal of the season" ·
VERIFIER chips "Vardy ✓" "24th ✓" → "on air" · RECEIPTS mono "AI live · verified 2 · __ s · __ tokens · $__ · trace 7f3a…".
Right of it: a TV lower-third mock "LEI 3-0 EVE 65'" + "GOAL · Vardy scores from the spot: his 24th league goal of the season."
and a "WHY?" pill opening a mini pitch with two dots (2416 foul won, 2418 penalty → goal).

SECTION 3 · WHAT IT TALKS ABOUT (centre, 2-column grid of 8 rows with icon, name, one-line rule, priority circle):
the 8 insight types from section 2 with their priorities. Below: card "PRE-MATCH CONTEXT · Leicester before Everton, 7 May 2016" with chips
"Elo #1 · 1,624" "Expected score 0.76" "Form W W D W D" "Unbeaten in 10" "Won 3–2 at Everton in December". Footer line "QUEUE max 1 insight per minute · 5-min cooldown · goals skip the queue".

SECTION 4 · WHAT THE VERIFIER REJECTS (right, red X rows): the 5 rejects from section 2 with one-line reasons,
then a green row "Fallback template always ready: GOAL 65' · Jamie Vardy (pen) · Leicester City 3-0 Everton · xG 0.78".

SECTION 5 · PLATFORM (bottom-left, icons with arrows):
"Codespaces dev container · infrastructure as code" → "Bicep · infra/main.bicep" → "Azure Container Apps · IP allow-list, scale 0–2" →
"User-assigned managed identity · Foundry User + AcrPull" → "Microsoft Foundry agent · no API keys anywhere" →
"Application Insights · OpenTelemetry traces, no prompts logged" → "Budget alert".
A small badge under it: "EVAL GATE ≥ 90% verified first time · ≤ 10% fallback · p95 ≤ 1.2 s → cheapest passing model serves".

SECTION 6 · DATA AND RULES (bottom-centre, icon list): "StatsBomb open data · 380 matches · Leicester's 2015/16 title season" ·
"Demo: Leicester City 3–1 Everton, 7 May 2016 · upset demo: Arsenal 1–2 Swansea" · "Season stats use only earlier matches" ·
"Checksums + schema check before ingest · data version on every card" · "No raw data in the repo (pre-commit hook)" · "Not for commercial use".

SECTION 7 · TIMELINE (bottom-right): OCT 8 Start → NOV 1 Milestone 1 "Local demo, template insights, no AI" → NOV 22 Milestone 2 "Azure + AI + 2-min video + blog".

Footer left: the disclaimer from section 2. Bottom-right: dashed box "StatsBomb logo".
```

---

## 4. Slides 01–18 (1920 × 1080 each)

**01 · Architecture**
```text
Label "ARCHITECTURE". Title "One container, one live engine, zero keys".
Dashed box "Azure Container Apps · one image, deployed with Bicep": card "app" [green PUBLIC] "Streamlit: live score, stats, insight feed, Why? pitch"
→ "in-process" → card "engine" [yellow INTERNAL] "Replay, stats, detector registry, priority queue, verifier".
Dashed blue "keyless" arrow → right card "Microsoft Foundry: Narrator agent writes slot wording".
Left card "Browser: allow-listed IPs" → app. Bottom-left dashed card "StatsBomb open data: fetched at start-up, never committed" → engine.
Right card "Season baselines: summary JSON, version dv-…" → engine. Bottom band "Application Insights · OpenTelemetry from every stage".
Footer: the disclaimer.
```

**02 · Code vs. model**
```text
Label "ARCHITECTURE". Title "Code computes. The model words it."
Left card, green edge, Python icon "Deterministic code: tested Python over StatsBomb events": ticks "Every number, clock, xG, Elo and rank" ·
"Fact cards with evidence IDs" · "Season stats from earlier matches only" · "Test-first with pytest".
Right card, blue edge, sparkle icon "Microsoft Foundry model: words only": arrows "Words the top fact card" · "Writes slots like {player_surname}" ·
"Analyst or casual-fan tone"; red line "Never types a number, never sees raw data".
Bottom green band, shield: "Verifier: every number, name and comparison must come from the card. One retry or 1.2 s, then a template line."
```

**03 · Event to insight**
```text
Label "ARCHITECTURE". Title "From match event to verified insight".
Six numbered cards, U-shape: 1 Event [engine] "event 2418, 64:49 → 65'" · 2 Detect [engine] "goal_in_context: Vardy, penalty, xG 0.78" ·
3 Queue [engine] "priority 10, goals skip the queue" · 4 Word [Foundry, blue] "JSON with slots + evidence IDs" ·
5 Check [engine] "numbers, names, comparisons" (dashed yellow arrow back to 4: "one retry") · 6 Show [app] "overlay, Why? pitch, receipts".
Yellow stat tiles: "1 insight/min max" · "1.2 s AI budget" · "1 retry" · "0 API keys".
```

**04 · Keyless security**
```text
Label "ARCHITECTURE". Title "Keyless everywhere with Microsoft Entra ID".
Column "In the Codespace": "az login: personal account" → "DefaultAzureCredential" → "Same code on Azure, unchanged".
Column (pink header) "Azure Container Apps": "User-assigned managed identity" → "Foundry User on Foundry, AcrPull on the registry" → "Tokens refresh automatically".
Blue band "Entra token → Microsoft Foundry · Container Registry · Application Insights".
Red footer "Key auth disabled on Foundry and Application Insights; registry admin user off; all set in Bicep."
```

**05 · The data rule**
```text
Label "THE DATA RULE". Title "The repository contains no match data".
Terminal card: "$ git ls-files data" → "data/.gitkeep", "data/README.md", "data/season_baselines.json"; "$ python -m src.ingest.verify"; comment "# pre-commit hook blocks *.jsonl and data/raw".
Card "YOU DOWNLOAD THE DATA YOURSELF": "source, version, licence" · "SHA-256 checksums" · "fetched at start-up, never in the image".
Right card with a dashed "StatsBomb logo" box: "StatsBomb open data · PL 2015/16 · 380 matches"; tag "Research use · logo required · no redistribution · non-commercial".
```

**06 · Verify, ingest, version**
```text
Label "DATA". Title "Verify, convert, and stamp a data version".
Four cards with arrows: "Your download: match + season files in data/raw" → "verify: schema contract + SHA-256; a missing field fails, a newer download only warns" →
"ingest: StatsBomb → Event format, TV clock, own goals; Elo, form, ranks by date" → "Data version dv-…: a hash of the inputs; same inputs, same version".
Italic note: "Every card, trace and half-time summary carries the version."
```

**07 · Live demo moments**
```text
Label "THE FOUR MOMENTS". Yellow "LIVE DEMO" badge. Title "Live: replay the match".
Rows with a yellow tag, a blue tag and a "WATCH FOR" column: M1 [5'] "Vardy scores, xG 0.37" | Big chance + Why? pitch ·
M2 [65'] "Vardy penalty" | "his 24th", earlier matches only · M3 [74' · Arsenal 1–2 Swansea] "the underdog goes ahead" | Upset watch fires once ·
M4 [red "Rejected"] "AI writes 'pacy Vardy, his fourth hat-trick'" | the verifier blocks it, the template goes out.
Footer: "Then switch to casual fan: same facts, new words."
```

**08 · Honest limits**
```text
Label "HONEST LIMITS". Title "When the data can't answer, the app says so".
Left app panel, pills "AI wording: live" · "Verified: 0 numbers" · "Outside what the data covers"; text "This replay can't say who ran the most: StatsBomb 2015/16 has no tracking or 360 data."; yellow stat "3,269 events in this match".
Right list "OUT OF SCOPE, BY DESIGN", red X icons: "Player speed or distance" · "Other seasons or leagues" · "Predictions and betting" · "A player's body or intelligence" · "Anything after the replayed date".
Box "WHAT IT WOULD TAKE": "New sources and new detectors; tracking data is usually commercial."
```

**09 · What the data said**
```text
Label "HALF-TIME" in yellow. Title "What the data said". Subtitle "Deterministic results · data dv-…". Faint centre circle and halfway line in the background.
Five cards with big yellow numbers: "1,624 · Leicester's Elo, 1st before the match" · "0.76 · expected score at home to Everton" ·
"10 · matches unbeaten" · "24th · Vardy's league goal at 65'" · "0.72 · Arsenal's expected score before losing to Swansea".
Green italic: "'Favourite' is a rating, not a probability." Box "SECOND HALF: from Codespace to Azure".
```

**10 · Same intent, two files**
```text
Label "INFRASTRUCTURE AS CODE". Title "Same idea, two files".
Two code cards with traffic-light dots. Left "my dev machine · .devcontainer/devcontainer.json": "build": { "dockerfile": "Dockerfile" }, features azure-cli and docker-in-docker, postCreateCommand pip install.
Right "production · infra/main.bicep": "identity: { type: 'UserAssigned' }", "ingress: { external: true, ipSecurityRestrictions: [ … ] }",
"scale: { minReplicas: 0, maxReplicas: 2 }", "disableLocalAuth: true".
Footer: "Rebuild either from Git, any time: the same machine, the same app."
```

**11 · Measured**
```text
Label "MEASURED". Title "Numbers from a full replay". Subtitle "Leicester 3–1 Everton · one replica · data dv-…".
Horizontal bars (all values "__", equal placeholder lengths), legend "shorter is better": "Event to screen, p95" · "AI wording, p95" · "First deployment" · "Rollback".
Right card "QUALITY": big "__%" verified first time · "__%" template fallback · "0" wrong numbers on screen · "__" tokens per insight.
```

**12 · Add an insight live**
```text
Label "DEVELOP AND SHIP". Yellow "LIVE DEMO" badge. Title "Add insight 9 live with Claude Code".
Left "THE PATTERN EVERY INSIGHT FOLLOWS", four yellow numbered circles: 1 "A detector · src/interpret/detectors/flank_overload.py" ·
2 "Register it · src/interpret/registry.py" · 3 "Its template line · src/render/templates.py" · 4 "A hand-computed test · tests/test_flank_overload.py".
Right terminal "Claude Code · the prompt, abridged": "Add a 'flank overload' insight following the pattern of big_chance. 1. Fire when one team makes 70%+ of its final-third entries down one flank over 10 minutes. 2. Register it at priority 5. 3. Add its template. 4. Test with hand-made events."
Bottom box: "If it deviates: switch to branch v2-flank".
```

**13 · Ship v1.1 safely**
```text
Label "DEVELOP AND SHIP". Title "Ship v1.1: revisions with traffic weights".
One pink lane "Azure Container Apps · revisions with traffic weights": "v1.0 serving 100%" → "v1.1 revision at 0%" → "split 50 / 50" → "roll back: v1.0 100% in __ s".
Star footer: "v1.1 adds flank overload: same pipeline, verified the same way."
```

**14 · Receipts**
```text
Label "OPERATE". Title "Every insight carries its receipts".
Left: app panel "65' · Vardy scores from the spot: his 24th league goal of the season." with pills "AI wording: live" · "Verified: 2 checks", grey mono line "model __ · __ s · __ in / __ out tokens · est. $__";
two number tiles "24th · season.vardy.league_goals" and "0.78 · event.2418.xg"; a mini pitch with two dots; a yellow-edged box "What this data can't tell you: …".
Four yellow numbered callouts linking to the right list: 1 "Status: live, verified or template" · 2 "An evidence ID under every number" ·
3 "A pitch drawn from event data, never from the model's text" · 4 "What this data can't tell you". Footer: "Every insight also shows a copyable trace ID."
```

**15 · One trace**
```text
Label "OPERATE". Title "One trace, from event to screen".
Left Gantt span chart, axis 0–1.2 s: "replay event 2418" (blue, full width) → "detect goal_in_context" (short, green) → "queue" (tiny) →
"chat narrator" (long, green) → "verify" (short, green) → "render overlay" (short, blue). Legend "app · engine". Note "timings illustrative".
Right card "ON EVERY SPAN": "app.image_digest", "app.data_version", "cloud.region"; "GENERATIVE AI CONVENTIONS": "gen_ai.request.model", "gen_ai.usage.input_tokens", "gen_ai.usage.output_tokens";
eye-slash icon "Not logged: prompts, responses or secrets". Bottom cards: "Why? panel: the trace ID, right away" · "Application Insights: durable, a few minutes behind".
```

**16 · Evaluation picks the model**
```text
Label "OPERATE". Title "Evaluation decides which model serves".
Left table "GATE | PASS IF | RESULT": "Verified first time | ≥ 90% | __%" · "Template fallback | ≤ 10% | __%" · "Limits respected | = 100% | __%" ·
"Blocked by content filters | 0 | __" · "Latency p95 | ≤ 1.2 s | __ s" (no ticks yet: results pending).
Right stat tiles: "60 insights · 20 cards × 3 runs" · "$__ per insight" · "__ s p50" · "__ tokens".
Bottom-left box "The rule, fixed before the run: the cheapest deployment that passes every gate serves."
Bottom-right box "Switch models with one setting: AI_MODEL_DEPLOYMENT. No rebuild."
```

**17 · Extend it**
```text
Label "BUILD ON IT". Title "Take it apart: four ways to extend it".
Four cards with icons: "Add an insight · docs/EXTEND.md": detector · register · template · hand-computed test ·
"Bring another match · data/README.md": keep it out of Git · schema contract · checksums · earlier-matches-only baselines ·
"Add a fan profile · src/personalize": same facts · new tone · verifier unchanged ·
"Swap the model · AI_MODEL_DEPLOYMENT": add it in Bicep · run the eval · then switch.
Bottom yellow shield line: "Keep the two promises: tested code owns every number, and no match data enters the repository."
```

**18 · Ideas to build on**
```text
Label "BUILD ON IT". Title "Ideas to build on, with the same data".
Eight icon cards in two rows: "Giant-killers · the season's biggest Elo upsets" · "Penalties · scored vs xG" · "Head-to-head · every meeting this season" ·
"Late-drama table · goals after 85', club by club" · "Set-piece specialists · shots from corners and free kicks" · "Pressing map · PPDA by matchweek" ·
"Title race · Elo by date for the top 4" · "Ask the commentator · questions answered by the same tools".
Yellow warning footer: "Tracking data, other leagues and commercial use need other data: check the licence and keep it out of the repository."
```

---

## 5. Output files

| Image | Size | File name | Goes to |
|---|---|---|---|
| Overview | 2400 × 1350 | `overview.png` | `docs/images/overview.png` (README top) |
| Slides 01–18 | 1920 × 1080 | `01-architecture.png`, `02-code-vs-model.png`, `03-event-to-insight.png`, `04-keyless.png`, `05-data-rule.png`, `06-verify-version.png`, `07-demo-moments.png`, `08-honest-limits.png`, `09-half-time.png`, `10-two-files.png`, `11-measured.png`, `12-add-insight.png`, `13-ship-v11.png`, `14-receipts.png`, `15-trace.png`, `16-eval-gate.png`, `17-extend.png`, `18-ideas.png` | `docs/images/slides/` + gallery in `docs/SLIDES.md` |
| HTML sources | n/a | `src/<same name>.html` | `docs/visuals/src/` |

---

## 6. Verification checklist (used before upload)

- [ ] Every number matches section 2, or is "__". No invented stats.
- [ ] Names spelled right: Jamie Vardy, King, Mirallas, Ashley Williams, Campbell, Routledge, Leicester City, Everton, Arsenal, Swansea City.
- [ ] Clock shows 65' (not 64'), 5', 33', 88', 74'.
- [ ] No win-probability wording ("76% chance"); Elo shown as rating or expected score.
- [ ] Nothing about a second Analyst agent, French, AKS, multiple services, Parquet/DuckDB.
- [ ] Bicep, user-assigned managed identity, Application Insights + OpenTelemetry, eval gate shown where the brief asks.
- [ ] No club crests, Premier League logos or player photos; StatsBomb logo is a dashed placeholder.
- [ ] Disclaimer present on the overview and slide 01.
- [ ] Text readable at 100% zoom; nothing cut off; sizes as in section 5.
