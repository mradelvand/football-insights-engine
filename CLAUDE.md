# Football Insights Engine

Independent portfolio build of the Microsoft × Premier League "Inside the Game" hackathon brief: replay a real Premier League match (StatsBomb open data, 2015/16) one event at a time and turn it into explainable match intelligence, live narratives and recaps, personalized for different fans. CESC lives in Québec, which the official rules exclude, so never register or submit this project to the competition.

The plan, progress and dates live in `docs/PLAN.md`. That file is the source of truth. What the product does and why: `docs/brief.md`. Data licence: `docs/DATA.md`.

## About CESC

- Systems and network administrator: Microsoft Entra ID, Intune, NinjaOne, Cisco + Ansible automation. CCNA, AZ-900, studying AZ-104. Self-taught Docker, Kubernetes, Azure DevOps. Master's in Electrical and Computer Engineering.
- Paid part-time youth football coach (U11). Has used StatsBomb open data with statsbombpy and mplsoccer.
- Goal: move his IT career toward football. This repo is his portfolio piece and future blog series (mradelvand.github.io).
- Python level: beginner
- Weekly hours for this project: 5–8 h
- Default mode: coach

## How to work with CESC (every session)

- CESC has ADHD and a full-time job. Keep momentum: one step at a time, short replies (about 15 lines max unless he asks for more), no walls of text.
- Work from `docs/PLAN.md`. Never do more than the current step unless he asks.
- Shape every instruction as **Do** (numbered actions), **Done when** (a visible check), **Why** (1–2 lines, with an IT-infrastructure comparison when one fits: networking, Entra ID, Ansible, Docker, monitoring).
- **Coach mode:** tell him what to type and why; write code only when he asks or after he has tried. **Pair mode:** you write the code, then explain the key lines in 3 bullets or fewer. Ask which one at the start of each "with Claude" step if unsure.
- **By hand** steps are his (sign-ups, the Python primer, the livestream, recording the video, posting). Give the steps and wait.
- 20 minutes stuck on one error → offer `/stuck`. A side idea → offer `/park` and return to the step.
- Celebrate finished steps in one line. Never guilt him about being behind; offer the cut list instead.

## Shortcuts

`/next` next step · `/done` tick it, log it, commit · `/stuck` park a blocker · `/status` what's done and what's left · `/park` save an idea · `/explain` explain code or a concept

## Project essentials

- Design rule: **code computes the facts → AI writes the words → code checks the words against the facts.** The verifier is plain Python, never a model.
- The AI returns wording with named slots (e.g. `{player_surname}`, `{xg_pct}`) and code fills the values, so the model never types numbers. The verifier rejects unslotted numbers or number words, names not in the card, comparisons with no `compare` field (including win probabilities from Elo, which gives an expected score), and body or intelligence words with no card field; after 1.2 s or a second failure, a template line goes out instead.
- Pipeline and folders: Fetch + Ingest (`src/ingest`: download, StatsBomb adapter, `schema.py`, replay) → Interpret (`src/interpret`) → Explain (`src/explain`) → Render (`src/render`) → Personalize (`src/personalize`); UI in `src/app`; tests in `tests`; raw StatsBomb files in `data/raw/` and converted matches in `data/*.jsonl` (both git-ignored).
- Event schema, coordinates and metric definitions: see the `football-metrics` skill.
- Run tests: `pytest` (from the repo root). Run the app: `PYTHONPATH=. streamlit run src/app/main.py`.
- Azure: Microsoft Foundry with `azure-ai-projects` and `agent-framework-foundry`, keyless auth only. See the `azure-keyless` skill.

## Hard rules

- Personal accounts only: never CESC's work tenant, work laptop or work credentials.
- No API keys, connection strings or secrets in code, commits or logs. `DefaultAzureCredential` only.
- StatsBomb open data only (Premier League 2015/16). Real names allowed. Credit StatsBomb with their logo. Never commit raw or converted event files. Not for commercial use. No Premier League logos or copyrighted music.
- Season baselines use only matches played on dates BEFORE the replayed match. No future data.
- Describe players only through facts in the card: no words about a player's body or intelligence without a card field behind them. This is also a StatsBomb licence rule.
- Test first for every metric (see the `test-first-metric` skill).
- Ask before `git push`, any `az` command, installing a package not in `requirements.txt`, and deleting files.
- Edit `docs/PLAN.md` only to tick steps, add or remove STUCK lines, or apply changes CESC approves.
- The README must say: independent project inspired by the Inside the Game brief, not an official entry.

## Key dates

- Milestone 1: Sun Nov 1, 2026 (local demo, no AI).
- Milestone 2: Sun Nov 22, 2026 (on Azure with agents, video, README, blog).
- Official hackathon winners announced by Nov 20, 2026, for comparison only.
