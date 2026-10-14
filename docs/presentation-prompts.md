# Football Insights Engine: lessons from Brian Benz's demo

Inspired by the structure of Brian Benz's (Microsoft) football analytics demo. Everything below is adapted to our project.

## 1. TL;DR

- **Same core:** code computes, the model words it; every number is checked; one retry; evidence IDs; keyless Azure; no data in the repo.
- **Different by design:** his app **answers questions** (user asks). Ours **commentates live** (the match pushes insights).
- **Worth stealing:** data checks + data version, honest limits, receipts on every insight, end-to-end traces, an evaluation gate for the model, Bicep with hardened keyless, a repeatable "add an insight" pattern, an "extend it" guide.
- **Skip for now:** AKS as a second platform, 3 separate services, Parquet/DuckDB in Blob, the Q&A "Ask" mode (parked for December).

## 2. Similarities and differences

| Topic | Brian's demo | Ours today | Take it? |
|---|---|---|---|
| Code computes, model explains | ✅ | ✅ | have it |
| Number check + one retry | hides the narrative | template line instead | have it |
| Evidence IDs | under every number | in the card, Why? pitch | **E** (show under every number) |
| Usage | user **asks** | match **pushes** | different by design |
| Receipts per answer | status pills, model, latency, tokens, cost, trace ID | ❌ | **E** |
| Honest limits | "What this data can't tell you" | ❌ | **F** |
| Keyless, key auth disabled | ✅ everywhere | planned, not hardened | **C** |
| Infrastructure as code | Bicep (+ YAML on AKS) | `az containerapp up` | **C** |
| No data in repo | guard script + hook + CI | `.gitignore` only | **A** |
| Schema contract + checksums | ✅ | ❌ | **A** |
| Data version | `cv-…` hash of inputs | ❌ | **B** |
| Telemetry | OpenTelemetry spans, GenAI conventions, no prompts logged | print logs | **D** |
| Model evaluation | gates over 34 cases × 3 runs; cheapest passing model serves | ❌ | **K** |
| Switch model | one setting | hard-coded name | **K** |
| "Add a feature" pattern | tool → register → card → test | test-first, no registry | **G** |
| Extend guide | HACKATHON-GUIDE + METHODS | ❌ | **L** |
| Rollout | revisions 0% → 50/50 → rollback | none | **H** (optional) |
| Measured results | p95, rollout, rollback, cost | ❌ | **D + K** |
| Platforms | AKS **and** Container Apps | Container Apps | skip |
| Services / store | 3 services; Parquet + DuckDB | 1 container; JSONL + summary JSON | skip |
| Data licence | CC0 / CC BY | StatsBomb (stricter) | n/a |

## 3. Plan additions (applied Oct 14, 2026, with the Analyst agent and French output cut; see docs/PLAN.md)

| # | Addition | Step | Time |
|---|---|---|---|
| A | Data guard: `data/README.md` (source, licence, SHA-256); `verify` checks schema contract + checksums before ingest (schema drift fails); pre-commit hook blocks `*.jsonl` and `data/raw` | w1-3 | +30 min |
| G | Detector registry pattern: detector → register → template line → hand-computed test | w3-2a | 0 (design) |
| B | Data version `dv-…` (hash of inputs) on every card, log line, half-time summary | w4-0 | +15 min |
| E | Receipts in the Why? panel: status pills (AI live / verified N numbers / template), model, latency, tokens, cost, copyable trace ID, evidence ID under every number; max-token cap | w4-7 | +15 min |
| K | Evaluation gate: `evals/cases.yaml` with ~20 fact cards × 3 runs; gates: verifier first-pass ≥ 90%, fallback ≤ 10%, p95 ≤ 1.2 s, 0 content-filter blocks; cheapest passing deployment serves; model chosen by `AI_MODEL_DEPLOYMENT` setting | new w5-6 | +1 h |
| F | "What this data can't tell you" panel | w5-5b | +15 min |
| C | Deploy with Bicep (`infra/main.bicep`): user-assigned managed identity, Foundry User + AcrPull, key auth off, registry admin off, IP allow-list, scale 0–2 | w6-2 | +1 h |
| D | Application Insights via OpenTelemetry: spans event → detect → word → verify → show; GenAI attributes (model, input/output tokens); data version + image digest on every span; never log prompts, responses or secrets | w6-3 | +15 min |
| L | `docs/EXTEND.md`: add an insight, bring your own data, swap the model; the two promises | w6-5 | +15 min |
| H | *Optional:* ship v1.1 as a revision, 0% → 50/50 → rollback, timed | w6-2b | +30 min |
| I | *Optional:* parity check, local Docker vs Azure give the same card list | w6-2 | +15 min |

**Time:** required +3 h 45 → plan ≈ **43 h 20**. About 41 h of work remain over 5.5 weeks: ~7.3 h/week, ~9 h/week with the learning buffer. **That's over your 5–8 h.**
Suggested trade: cut the **second (Analyst) agent** (w5-2a, 1 h 15) and **French output**. They're already #1–2 on the cut list, and the eval gate (K) is a stronger portfolio story than a second agent.

**Parking lot additions:** AKS second platform; 3 services; Parquet/DuckDB; season "giant-killers" table (biggest Elo upsets of 2015/16); penalty takers vs xG; late-drama table by club; head-to-head across the season.

## 4. Slide prompts (18)

**Shared style (paste first):**
```text
Dark presentation slide, 16:9, near-black background, small green uppercase section label top-left,
large light-grey bold title, rounded dark cards with thin borders, coloured pill tags, thin arrows,
icon-led, crisp small text, generous spacing, small timer badge top-right, slide number bottom-right.
Footer left: "github.com/mradelvand/football-insights-engine". No club crests or Premier League logos.
Where a number shows "__", leave it as a blank placeholder to fill in later.
```

**1 · Architecture**
```text
Label "ARCHITECTURE". Title "One container, one live engine, zero keys".
Dashed box "Azure Container Apps · one image, deployed with Bicep": card "app" [green PUBLIC] "Streamlit: live score, stats, insight feed, Why? pitch"
→ "in-process" → card "engine" [yellow INTERNAL] "Replay, stats, detector registry, priority queue, verifier".
Dashed blue "keyless" arrow → right card "Microsoft Foundry: Narrator agent writes slot wording".
Left card "Browser: allow-listed IPs" → app. Bottom-left dashed card "StatsBomb open data: fetched at start-up, never committed" → engine.
Right card "Season baselines: summary JSON, version dv-…" → engine. Bottom band "Application Insights · OpenTelemetry".
```

**2 · Code vs. model**
```text
Label "ARCHITECTURE". Title "Code computes. The model words it."
Left card, green edge, Python icon "Deterministic code: tested Python over StatsBomb events": ticks "Every number, clock, xG, Elo and rank" ·
"Fact cards with evidence IDs" · "Season stats from earlier matches only" · "Test-first with pytest".
Right card, blue edge, sparkle icon "Microsoft Foundry model: words only": arrows "Words the top fact card" · "Writes slots like {player_surname}" ·
"Analyst or casual-fan tone"; red line "Never types a number, never sees raw data".
Bottom green band, shield: "Verifier: every number, name and comparison must come from the card. One retry or 1.2 s, then a template line."
```

**3 · Event to insight**
```text
Label "ARCHITECTURE". Title "From match event to verified insight".
Six numbered cards, U-shape: 1 Event [engine] "index 2418, 64:49 → 65'" · 2 Detect [engine] "goal_in_context: Vardy, penalty, xG 0.78" ·
3 Queue [engine] "priority 10, goals skip the queue" · 4 Word [Foundry, blue] "JSON with slots + evidence IDs" ·
5 Check [engine] "numbers, names, comparisons" (dashed yellow arrow "one retry") · 6 Show [app] "overlay, Why? pitch, receipts, trace ID".
Yellow stat tiles: "1 insight/min max" · "1.2 s AI budget" · "1 retry" · "0 API keys".
```

**4 · Keyless security**
```text
Label "ARCHITECTURE". Title "Keyless everywhere with Microsoft Entra ID".
Column "In the Codespace": "az login: personal account" → "DefaultAzureCredential" → "Same code on Azure, unchanged".
Column (pink) "Azure Container Apps": "User-assigned managed identity" → "Foundry User on Foundry, AcrPull on the registry" → "Tokens refresh automatically".
Blue band "Entra token → Microsoft Foundry · Container Registry · Application Insights".
Red footer "Key auth disabled on Foundry and Application Insights; registry admin user off; all set in Bicep."
```

**5 · The data rule**
```text
Label "THE DATA RULE". Title "The repository contains no match data".
Terminal: "$ git ls-files data" → "data/.gitkeep", "data/README.md", "data/season_baselines.json"; "$ python -m src.ingest.verify"; comment "# pre-commit hook blocks *.jsonl and data/raw".
Card "YOU DOWNLOAD THE DATA YOURSELF": "source, version, licence" · "SHA-256 checksums" · "fetched at start-up, never in the image".
Right card with an empty "StatsBomb logo" box: "StatsBomb open data · PL 2015/16 · 380 matches"; tag "Research use · logo required · no redistribution · non-commercial".
```

**6 · Verify, ingest, version**
```text
Label "DATA". Title "Verify, convert, and stamp a data version".
Four cards with arrows: "Your download: match + season files in data/raw" → "verify: schema contract + SHA-256; schema drift fails" →
"ingest: StatsBomb → Event format, TV clock, own goals; Elo, form, ranks by date" → "Data version dv-…: hash of the inputs; same inputs, same version".
Italic note: "Every card, log line and half-time summary carries the version."
```

**7 · Live demo moments**
```text
Label "THE FOUR MOMENTS". Yellow "LIVE DEMO" badge. Title "Live: replay the match".
Rows with a yellow tag, a blue tag and "WATCH FOR": M1 [5'] "Vardy scores, xG 0.37" | Big chance + Why? pitch ·
M2 [65'] "Vardy penalty" | "his 24th", earlier matches only · M3 [74' · Arsenal 1–2 Swansea] "the underdog goes ahead" | Upset watch fires once ·
M4 [red "Rejected"] "AI writes 'pacy Vardy, his fourth hat-trick'" | the verifier blocks it, the template goes out.
Footer: "Then switch to casual fan: same facts, new words."
```

**8 · Honest limits**
```text
Label "HONEST LIMITS". Title "When the data can't answer, the app says so".
Left app panel, pills "AI wording: live" · "Verified: 3 numbers" · "Outside what the data covers"; text "This replay can't say who ran most: 2015/16 has no tracking or 360 data."; stat "3,269 events in this match".
Right list "OUT OF SCOPE, BY DESIGN", red X icons: "Player speed or distance" · "Other seasons or leagues" · "Predictions and betting" · "A player's body or intelligence" · "Anything after the replayed date".
Box "WHAT IT WOULD TAKE": "New sources and new detectors; tracking data is usually commercial."
```

**9 · What the data said**
```text
Label "HALF-TIME" in yellow. Title "What the data said". Subtitle "Deterministic results · data dv-…". Faint centre circle in the background.
Five cards with big yellow numbers: "1,624: Leicester's Elo, 1st before the match" · "0.76: expected score at home to Everton" ·
"10: matches unbeaten" · "24th: Vardy's league goal at 65'" · "0.72: Arsenal's expected score before losing to Swansea".
Green italic: "'Favourite' is a rating, not a probability." Box "SECOND HALF: from Codespace to Azure".
```

**10 · Same intent, two files**
```text
Label "INFRASTRUCTURE AS CODE". Title "Same idea, two files".
Two code cards with traffic-light dots. Left "my dev machine · .devcontainer/devcontainer.json": Dockerfile build, azure-cli and docker-in-docker features, postCreate pip install.
Right "production · infra/main.bicep": snippets "identity: { type: 'UserAssigned' }", "ingress: { external: true, ipSecurityRestrictions: [ … ] }",
"scale: { minReplicas: 0, maxReplicas: 2 }", "disableLocalAuth: true".
Footer: "Rebuild either from Git, any time: the same machine, the same app."
```

**11 · Measured**
```text
Label "MEASURED". Title "Numbers from a full replay". Subtitle "Leicester 3–1 Everton · one replica · data dv-…".
Horizontal bars, "shorter is better": "Event to screen, p95: __ ms" · "AI wording, p95: __ s" · "First deployment: __ min" · "Rollback: __ s".
Right card "QUALITY": big numbers "__% verified first time" · "__% template fallback" · "0 wrong numbers on screen" · "__ tokens per insight".
```

**12 · Add an insight live**
```text
Label "DEVELOP AND SHIP". Yellow "LIVE DEMO" badge. Title "Add insight 9 live with Claude Code".
Left "THE PATTERN EVERY INSIGHT FOLLOWS", four numbered yellow circles: 1 "A detector: src/interpret/detectors/flank_overload.py" ·
2 "Register it: src/interpret/registry.py" · 3 "Its template line: src/render/templates.py" · 4 "Prove it with a hand-computed test: tests/test_flank_overload.py".
Right terminal "Claude Code · the prompt, abridged": "Add a 'flank overload' insight following the pattern of shot_cluster. 1. Fire when one team makes 70%+ of its final-third entries down one flank over 10 minutes. 2. Register it at priority 5. 3. Add its template. 4. Test with hand-made events."
Bottom: "If it deviates: switch to branch v2-flank".
```

**13 · Ship v1.1 safely (only if H)**
```text
Label "DEVELOP AND SHIP". Title "Ship v1.1: revisions with traffic weights".
One pink lane "Azure Container Apps": "v1.0 serving 100%" → "v1.1 revision created at 0%" → "traffic split 50/50" → "roll back: v1.0 100% in __ s".
Star footer: "v1.1 adds flank overload: same facts pipeline, verified the same way."
```

**14 · Receipts**
```text
Label "OPERATE". Title "Every insight carries its receipts".
Left: app screenshot-style panel "65' · Vardy scores from the spot: his 24th league goal of the season." with pills "AI wording: live" · "Verified: 2 checks" and a grey line "model __ · __ s · __ in / __ out tokens · est. $__";
under it two small number tiles "24th · season.vardy.league_goals" and "0.78 · event.2418.xg"; a mini pitch with two dots; a yellow-edged box "What this data can't tell you: …".
Four yellow numbered callouts connect to the right list: 1 "Status: live, verified or template" · 2 "An evidence ID under every number" ·
3 "A pitch drawn from event data, never from the model's text" · 4 "What this data can't tell you". Footer: "Every insight also shows a copyable trace ID."
```

**15 · One trace**
```text
Label "OPERATE". Title "One trace, from event to screen".
Left Gantt-style span chart, axis 0–1.2 s: "replay event 2418" (blue, full width) → "detect goal_in_context" (short green) → "queue" (tiny) →
"chat narrator" (long green) → "verify" (short green) → "render overlay" (short blue). Legend "engine · app". Note "timings illustrative".
Right card "ON EVERY SPAN": "app.image_digest", "app.data_version", "cloud.region"; "GENERATIVE AI CONVENTIONS": "gen_ai.request.model", "gen_ai.usage.input_tokens", "gen_ai.usage.output_tokens";
eye-slash icon "Not logged: prompts, responses or secrets". Bottom cards: "Why? panel: trace ID now" · "Application Insights: durable, a few minutes behind".
```

**16 · Evaluation picks the model**
```text
Label "OPERATE". Title "Evaluation decides which model serves".
Left table "GATE | PASS IF | RESULT": "Verified first time ≥ 90% | __%" · "Template fallback ≤ 10% | __%" · "Limits respected = 100% | __%" ·
"Blocked by content filters = 0 | __" · "Latency p95 ≤ 1.2 s | __ s", each with a green tick.
Right stat tiles: "60 insights: 20 cards × 3 runs" · "$__ per insight" · "__ s p50" · "__ tokens".
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
Eight icon cards in two rows: "Giant-killers: the season's biggest Elo upsets" · "Penalties: scored vs xG" · "Head-to-head: every meeting this season" ·
"Late drama table: goals after 85', club by club" · "Set-piece specialists: shots from corners and free kicks" · "Pressing map: PPDA by matchweek" ·
"Title race: Elo by date for the top 4" · "Ask the commentator: questions answered by the same tools".
Yellow warning footer: "Tracking data, other leagues and commercial use need other data: check the licence and keep it out of the repository."
```
