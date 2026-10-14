# Football Insights Engine — Game Plan

Independent portfolio build of the Microsoft × Premier League "Inside the Game" brief (Québec residents can't officially enter), on real StatsBomb open data (Premier League 2015/16). 57 steps, Oct 8 → Nov 22, 2026. What it does: `docs/brief.md`. Data licence: `docs/DATA.md`.

- **Milestone 1 · Sun Nov 1, 2026:** local demo, no AI. Leicester 3–1 Everton replays in the browser with live stats and template insights.
- **Milestone 2 · Sun Nov 22, 2026:** deployed on Azure with AI agents, a video under 2 minutes, README and blog post.
- **Time:** about 39 h 35 min of required steps. With a 25% learning buffer that's about 7.5 h a week, the top of a 5–8 h range. If week 1 runs late, start cutting early.

## How to read this file

- `- [ ]` to do · `- [x]` done. A stuck step keeps `- [ ]` and gets a line `  - STUCK (YYYY-MM-DD): reason`.
- **by hand** = you do it yourself (website sign-ups, typing practice, recording). **with Claude** = do it in Claude Code, in coach or pair mode.
- Steps tagged **optional** can be skipped. **milestone** steps close a phase.
- Every session: `/next` → do 1–3 steps → `/done` → stop after a step, never in the middle.

**Behind by 2+ hours in a week? Cut in this order:** 1) French output, 2) the second agent, 3) KQL logs (w6-3), 4) Azure deploy (demo locally in the video). **Never cut:** the metric tests, the verifier, the template fallback, the video.

**Stuck?** 20 minutes on one error → `/stuck`. Can't start → do only the first 5 minutes of the next step. New idea → `/park`.

## Week 0 · Set up, install nothing

Dates: Thu Oct 8 – Sun Oct 11 (2026-10-08 → 2026-10-11) · Total: ≈2 h 45 min · **Win:** Your repo is live and Python runs in your browser.

- [x] **w0-1 · Write your goal in one sentence** · 10 min · by hand
  - Do:
    1. Open any notes app.
    2. Write: *"By Nov 22 I publish a working football insight engine on real StatsBomb data, with a video under 2 minutes and a blog post. Local demo by Nov 1."*
    3. Pin it where you'll see it.
  - Done when: The sentence is saved and pinned.
  - Why: When a shiny new idea appears, ask: "Does this help the goal?" If not, run `/park` and keep going.
- [x] **w0-2 · Register for the free Oct 14 livestream** · 5 min · by hand
  - Do:
    1. Open the event page.
    2. Sign in with your **personal** Microsoft account.
    3. Click **Register**.
    4. Add Wed Oct 14, 12:00–13:00 to your calendar.
  - Done when: Confirmation email received and the hour is blocked.
  - Why: Microsoft walks through a reference architecture for this exact problem. It's open to everyone; it is not the competition.
  - Link: [Open the event page](https://developer.microsoft.com/en-us/reactor/events/27536/)
- [x] **w0-3 · Create a public GitHub repo** · 20 min · by hand
  - Do:
    1. Name: `football-insights-engine`
    2. Choose **Public**.
    3. Tick **Add a README**.
    4. .gitignore template: **Python**. License: **MIT**.
    5. Click **Create repository**.
  - Done when: The repo shows README, LICENSE and .gitignore.
  - Why: Public means it's your portfolio. The Python .gitignore keeps junk and secrets out. MIT lets others learn from it.
  - Link: [Open GitHub: new repo](https://github.com/new)
- [x] **w0-4 · Open the repo in Codespaces** · 15 min · by hand
  - Do:
    1. On the repo page, click the green **Code** button.
    2. Open the **Codespaces** tab.
    3. Click **Create codespace on main**.
    4. In the terminal at the bottom, run the command below.
  - Command:
    ```bash
    python --version
    ```
  - Done when: It prints Python 3.10 or higher.
  - Why: Codespaces is VS Code in your browser, running on a cloud machine. Nothing to install, nothing on your work laptop. Think of it as a jump box. An idle codespace stops by itself.
- [x] **w0-4b · Install Claude Code and paste the setup prompt** · 15 min · by hand
  - Do:
    1. In the codespace terminal, run the command below to install Claude Code.
    2. Open a new terminal (**Terminal → New Terminal**), run `claude` and log in through the browser.
    3. Paste the whole setup prompt. When Claude asks to create files, allow edits for this session.
    4. Answer its questions one at a time.
  - Command:
    ```bash
    curl -fsSL https://claude.ai/install.sh | bash
    ```
  - Done when: Claude has created the workspace and shown you the file tree and its checks.
  - Why: One prompt builds the whole workspace: this plan, Claude's project memory, your shortcuts and the dev container. Claude Code needs a paid Claude plan (Pro, Max, Team or Enterprise).
- [x] **w0-6 · Check your Python setup** · 10 min · with Claude
  - Do:
    1. Run the two commands below.
  - Command:
    ```bash
    python -c "import pandas, mplsoccer, streamlit"
    pytest
    ```
  - Done when: The first command prints nothing. pytest says "no tests ran", which is fine for now.
  - Why: The dev container is your isolated environment, like a dedicated VM per project, so you don't need a separate venv inside it. `requirements.txt` lists the packages and the container installs them on every rebuild.
- [x] **w0-7 · Check the structure and make the first commit** · 10 min · with Claude
  - Do:
    1. Look at the folder tree: `.claude`, `.devcontainer`, `docs`, `src` (one folder per stage), `tests`, `data`.
    2. Open `pyproject.toml` and read the `[tool.pytest.ini_options]` part.
    3. Commit and push with the command below.
  - Command:
    ```bash
    git add . && git commit -m "Add Claude Code workspace and plan" && git push
    ```
  - Done when: On GitHub you see .claude, .devcontainer, docs, src, tests and data.
  - Why: One folder per pipeline stage, so the repo looks like the diagram. Git ignores empty folders, so `tests` and `data` hold a `.gitkeep` file. `__init__.py` makes the src folders importable, and `pythonpath = ["."]` in pyproject.toml lets pytest find them, so `from src.ingest.schema import Event` works.
- [x] **w0-7b · Rebuild the codespace and restart Claude Code** · 15 min · with Claude
  - Do:
    1. Type `/exit` to close Claude Code.
    2. Open the Command Palette (**Ctrl+Shift+P**, or **⌘⇧P** on a Mac) → **Codespaces: Rebuild Container**. Wait a few minutes.
    3. In the new terminal, run `claude` (log in again if asked), then type `/status` and `/next`.
  - Command:
    ```bash
    claude
    ```
  - Done when: `/status` shows your progress and `/next` shows your next step.
  - Why: `.devcontainer/devcontainer.json` describes your whole dev machine in one file: Python, your packages, Azure CLI, Docker and Claude Code. Rebuild it any time and you get the same machine back. That's infrastructure as code, applied to your own workstation. The restart also makes sure Claude loads `CLAUDE.md` (its project memory) and your new shortcuts.
- [x] **w0-8a · Python primer, part 1 (sections 3–4)** · 30 min · by hand
  - Do:
    1. Read sections 3 and 4.
    2. Type every example into the codespace. Don't copy-paste.
    3. Type the examples yourself. Don't ask Claude to write them.
  - Done when: You can write a small function with a loop and an if.
  - Why: Next week's code is functions, loops and lists. Typing builds memory; reading alone doesn't.
  - Link: [Open the Python tutorial](https://docs.python.org/3/tutorial/)
- [ ] **w0-8b · Python primer, part 2 (5, 7.2, dataclasses)** · 30 min · by hand
  - Do:
    1. Tutorial section 5 (lists, dicts) and 7.2 (files).
    2. Read the first page of **dataclasses**.
    3. Try it: save a list of dicts to a file, one JSON object per line.
  - Done when: Your file has one JSON object per line.
  - Why: That exact format (JSON Lines) is how your match events will be stored.
  - Link: [Open dataclasses](https://docs.python.org/3/library/dataclasses.html)
- [x] **w0-9 · Write your first devlog entry** · 5 min · with Claude
  - Do:
    1. After your last step today, run `/done` and choose "stop for today".
    2. Give it 3 short lines: did, learned, next.
  - Done when: `docs/devlog.md` has today's entry, pushed to GitHub.
  - Why: It becomes your blog post. The "next" line tells you where to start next session, so you don't spend energy deciding.

## Week 1 · Load a real match

Dates: Mon Oct 12 – Sun Oct 18 (2026-10-12 → 2026-10-18) · Total: ≈5 h 20 min · **Win:** Leicester 3–1 Everton as a tested event file, with all 42 shots on a pitch.

- [ ] **w1-0 · Register with StatsBomb** · 5 min · by hand
  - Do:
    1. Open the StatsBomb resource centre.
    2. Register with your **personal** email.
    3. Download the media pack (you'll need the logo for the README and app).
  - Done when: Registration confirmed and the logo is saved.
  - Why: Their licence requires the logo on anything you publish.
  - Link: [Open the StatsBomb resource centre](https://statsbomb.com/resource-centre)
- [ ] **w1-1 · Try pytest** · 30 min · with Claude
  - Do:
    1. Create `tests/test_hello.py` with the code below.
    2. Run `pytest`.
    3. Change 2 to 3, run again, read the failure.
  - Command:
    ```python
    def test_add():
        assert 1 + 1 == 2
    ```
  - Done when: You've seen "1 passed" and "1 failed".
  - Why: Every metric gets a test. A test is config validation for your own code.
- [ ] **w1-2 · Define the event schema** · 45 min · with Claude
  - Do:
    1. In `src/ingest/schema.py`, make a dataclass `Event`.
    2. Fields: id, sb_id, match_id, period, minute, second, team, player, type, x, y, end_x, end_y, outcome, play_pattern, body_part, shot_type, xg.
    3. `id` = StatsBomb `index`; `sb_id` = the StatsBomb UUID.
    4. Types: pass, corner, shot, tackle, interception, clearance, block, pressure, foul, own_goal.
    5. Pitch is 120 × 80. Coordinates are from the acting team's view, attacking toward x = 120.
  - Done when: You can create an Event and print it.
  - Why: The schema is the contract between all stages, like agreeing on a packet format before writing a protocol. With team-relative coordinates, the final third is always x > 80.
- [ ] **w1-3 · Fetch the data** · 45 min · with Claude
  - Do:
    1. In `src/ingest/fetch.py`, use statsbombpy to download the 2015/16 match list (competition 2, season 27).
    2. Download the events for match 3754033 into `data/raw/` (git-ignored).
    3. Check `docs/DATA.md` (licence summary) and run `git status`: nothing from `data/raw/` shows up.
  - Done when: `data/raw/` holds the match list and 3,269 events, and `git status` doesn't list them.
  - Why: Like pulling a config backup to a local share: you keep a copy to work on, but it never goes into the public repo.
- [ ] **w1-4a · Converter, core** · 1 h · with Claude
  - Do:
    1. Test first: pick 3 StatsBomb events by hand (a pass, a shot, a tackle) and write what each Event should look like.
    2. In `src/ingest/statsbomb_adapter.py`, turn StatsBomb events into Event objects.
    3. Write `data/match_3754033.jsonl` (git-ignored), one event per line, sorted by index.
  - Done when: The 3 tests pass and the file has 3,269 lines.
  - Why: An adapter keeps StatsBomb's format at the edge, like a NAT: everything inside speaks your own format.
- [ ] **w1-4b · Converter: clock, added time and own goals** · 45 min · with Claude
  - Do:
    1. Clock: regular time shows minute + 1 (4:58 → 5'); added time shows 45+n / 90+n (see the `football-metrics` skill).
    2. Own Goal For → `own_goal`, counted in the score.
    3. Tests: the four goals come out as 5' (index 240), 33' (1324), 65' (2418) and 88' (3115).
  - Done when: pytest is green.
  - Why: The clock is what fans see. Getting 4:58 to show as 5' is the difference between looking right and looking broken.
- [ ] **w1-5 · Sanity check** · 30 min · with Claude
  - Do:
    1. Load the file in pandas and work out the score: it must be 3–1.
    2. Plot all 42 shots on a pitch with mplsoccer (`pitch_type="statsbomb"`), sized by xG.
  - Command:
    ```python
    import pandas as pd
    df = pd.read_json("data/match_3754033.jsonl", lines=True)
    print(df["type"].value_counts())
    ```
  - Done when: Score 3–1, and the shot map shows 42 shots with Vardy's penalty as one of the biggest dots.
  - Why: A known answer checks the whole converter at once, like pinging a known host after a routing change.
- [ ] **w1-6 · Watch the livestream** · 1 h · by hand · fixed time: Wed Oct 14 · 12:00
  - Do:
    1. Watch live.
    2. Take notes in `docs/reactor-notes.md`.
    3. Ask one question, e.g. "Container Apps jobs or Event Hubs consumers for per-match processing?"
  - Done when: Notes are pushed.
  - Why: Free access to the architecture Microsoft recommends for this exact scenario.

## Week 2 · From events to football facts

Dates: Mon Oct 19 – Sun Oct 25 (2026-10-19 → 2026-10-25) · Total: ≈4 h 45 min · **Win:** Tested numbers: possession, field tilt, PPDA.

- [ ] **w2-1 · Replay events like a live feed** · 45 min · with Claude
  - Do:
    1. In `src/ingest/replay.py`, read the JSONL in order (sorted by `id`, the StatsBomb index).
    2. Yield one event at a time, with a speed factor (e.g. 20× real time).
  - Done when: Events print with short pauses between them.
  - Why: Broadcasters get events as they happen. Later you swap the file reader for Event Hubs without touching anything else.
- [ ] **w2-2 · Build the match state** · 1 h · with Claude
  - Do:
    1. In `src/interpret/state.py`, make a `MatchState` class.
    2. Update it on every event: score (goals + own goals), on-ball events per team, events in the last 5 minutes, passes per team per third.
  - Done when: After a full replay, the score matches the goals in the file.
  - Why: One source of truth that every metric reads, like a routing table.
- [ ] **w2-3a · Possession % (test first)** · 1 h · with Claude
  - Do:
    1. First write a test with 5–10 hand-made events where you know the answer.
    2. Then write the function: each team's share of on-ball events in the last 5 minutes.
  - Done when: pytest is green.
  - Why: Writing the test first proves your numbers before any AI touches them.
- [ ] **w2-3b · Field tilt (test first)** · 1 h · with Claude
  - Do:
    1. Test first, then function.
    2. Field tilt = each team's share of final-third passes.
  - Done when: pytest is green.
  - Why: It shows who controls territory, which raw possession hides.
- [ ] **w2-3c · PPDA (test first)** · 1 h · with Claude
  - Do:
    1. Test first, then function.
    2. PPDA = opponent passes in their own defensive 60% ÷ your tackles + interceptions in that same zone.
    3. Coordinates are team-relative, so the zone flips: it's x < 72 in the opponent's events and x > 48 in your events (120 − 72 = 48). Your defensive actions are tackles (Duel: Tackle) and interceptions.
    4. Add one test that checks the zone from both teams' sides.
    5. Lower PPDA = harder pressing.
  - Done when: pytest is green.
  - Why: The standard pressing stat. As a coach you already know what it means on the grass.
## Week 3 · Milestone 1: local demo (MILESTONE)

Dates: Mon Oct 26 – Sun Nov 1 (2026-10-26 → 2026-11-01) · Total: ≈6 h 45 min · **Win:** Leicester–Everton replays in your browser with live stats and template insights.

- [ ] **w3-1 · Momentum metric (with a test)** · 45 min · with Claude
  - Do:
    1. Rolling count of threatening events (final-third entries, shots) per team over 5 minutes.
    2. Write the test, then the function.
  - Done when: pytest is green.
  - Why: Momentum swings are the moments commentators talk about.
- [ ] **w3-2a · Fact cards: goals, late drama, big chances, set pieces** · 1 h · with Claude
  - Do:
    1. In `src/interpret/detectors.py`, write the detectors for Goal in context (10), Late drama (10), Big chance (9) and Set-piece threat (8). Rules are in the `football-metrics` skill.
    2. Each returns a card: card_id, type, priority, clock, team, player, facts, compare, evidence_ids.
    3. `compare` = this match so far or the opponent for now; season-to-date arrives in w4-0.
  - Done when: A replay produces cards for all four goals and the 5' and 88' big chances.
  - Why: Fact cards are the bridge from numbers to words. Every AI sentence later must trace back to evidence_ids.
- [ ] **w3-2b · Fact cards: pressing, momentum, defensive wall + sense check** · 1 h · with Claude
  - Do:
    1. Detectors for Pressing surge (6), Momentum swing (6) and Defensive wall (5).
    2. Replay the full match and read every card.
    3. Ask: "Would I tell my U11s this at half-time?"
  - Done when: One replay gives 5–20 cards that make football sense.
  - Why: Your coaching eye is the quality filter no data scientist has.
- [ ] **w3-2c · Priority queue and cooldowns** · 45 min · with Claude
  - Do:
    1. In `src/interpret/queue.py`: merge cards about the same event, show the highest priority, at most one insight per match minute, 5-minute cooldown per player and per topic.
    2. Goals skip the queue and the cooldown.
    3. One test per rule.
  - Done when: pytest is green and a replay shows no two insights in the same minute.
  - Why: Like alert dedup and suppression windows: without them, the screen floods the way a pager does.
- [ ] **w3-3a · Streamlit page: score and stats** · 1 h 15 min · with Claude
  - Do:
    1. Use pair mode for this one: ask Claude to write it, then `/explain` the key parts.
    2. In `src/app/main.py`: a replay-speed slider, live score, possession, field tilt.
    3. Run it, then click **Open in Browser** when Codespaces offers port 8501.
  - Command:
    ```bash
    PYTHONPATH=. streamlit run src/app/main.py
    ```
  - Done when: The score and stats update as the match replays.
  - Why: User experience is a fifth of the brief, and it's what a hiring manager sees first.
- [ ] **w3-3b · Streamlit page: insight feed with templates** · 1 h · with Claude
  - Do:
    1. Write one template line per card type, e.g. `GOAL 65' · Jamie Vardy (penalty) · Leicester City 3–0`.
    2. Show the queue's insights in a feed as they appear, newest on top, each as its template line.
  - Done when: Template insights pop up during the replay.
  - Why: The Nov 1 demo already shows insights without AI, and these templates become the AI's fallback in w4-7.
- [ ] **w3-3c · Streamlit page: "Why?" pitch view** · 45 min · with Claude
  - Do:
    1. Give each card a **Why?** expander.
    2. Plot the card's evidence events on a pitch with mplsoccer (`pitch_type="statsbomb"`).
  - Done when: Clicking Why? shows the events on a pitch.
  - Why: Explainability you can see: the claim and its proof side by side.
- [ ] **w3-4 · Tag Milestone 1 and show it** · 15 min · with Claude · milestone
  - Do:
    1. Write the devlog entry.
    2. Run the command below.
    3. Post a screenshot on LinkedIn, with the StatsBomb logo.
  - Command:
    ```bash
    git tag v0.1 && git push --tags
    ```
  - Done when: v0.1 shows under Tags on GitHub.
  - Why: A tag is a known-good version you can always go back to, like a config backup before a change window.
- [ ] **w3-5 · Look at official entries** · 30 min · by hand · fixed time: after Oct 27 · optional
  - Do:
    1. Search GitHub for public repos about "Inside the Game" hackathon.
    2. Note one idea per repo that beats yours.
  - Done when: Three ideas in your devlog.
  - Why: Free lessons from dozens of people solving the same brief.

## Week 4 · Azure and your first AI call

Dates: Mon Nov 2 – Sun Nov 8 (2026-11-02 → 2026-11-08) · Total: ≈7 h 15 min · **Win:** One AI-written insight with season and pre-match context. No API key. No wrong numbers.

- [ ] **w4-0 · Season baselines** · 1 h 45 min + download time · with Claude
  - Do:
    1. Download all 380 event files for 2015/16 into `data/raw/` (≈1.13 GB, git-ignored).
    2. Compute team and player totals and league ranks by date. Use only matches played BEFORE the replayed match.
    3. Save one small summary file, `data/season_baselines.json` (this one is committed).
    4. Add Elo, form (last 5), unbeaten/winless runs and head-to-head per team, per date, to the same file. Rules in the `football-metrics` skill: start 1500, home +60, K = 20, same-day matches excluded.
    5. Test first, with the known values: Elo Leicester 1624 (1st), Tottenham 1602 (2nd), Arsenal 1571 (3rd), Everton 1484 (11th); expected score 0.76; form Leicester W W D W D, Everton D D D L W; Leicester unbeaten in 10; earlier meeting 2015-12-19, Everton 2–3 Leicester City.
    6. Add season-to-date `compare` to the goal cards.
  - Done when: Vardy shows 22 league goals before 2016-05-07, the 65' penalty card says his 24th, and the Elo tests are green.
  - Why: "His 24th of the season" is what fans want. "Earlier matches only" is like a point-in-time restore: no data from the future.
- [ ] **w4-0b · Pre-match card and upset watch** · 30 min · with Claude
  - Do:
    1. Pre-match context card, shown at kickoff and half-time outside the live queue: Elo rank for both teams, the Elo favourite, form, runs of 5+, earlier meeting.
    2. Upset watch (priority 7): fires when the team that wasn't the Elo favourite takes the lead, once per lead change.
    3. Tests: hand-made events where the underdog scores first; then replay 3753976 (Arsenal 1–2 Swansea): it fires once, at 74' (index 3201).
  - Done when: pytest is green and the Leicester–Everton pre-match card reads right.
  - Why: Form guides and giant-killings are what UK fans talk about before and during a match. Elo is a cheap, tested model the AI can only quote, never invent.

- [ ] **w4-1 · Create your Azure free account** · 30 min · by hand
  - Do:
    1. Use your **personal** Microsoft account, never your work one.
    2. Have a phone and a non-prepaid card ready.
  - Done when: The portal shows your subscription.
  - Why: The USD 200 credit lasts 30 days. Starting now covers weeks 4–6 exactly.
  - Link: [Open Azure free account](https://azure.microsoft.com/free/)
- [ ] **w4-2 · Resource group + budget alert** · 15 min · by hand
  - Do:
    1. Portal → **Resource groups** → **Create**: `rg-football-insights`, region East US (check model availability first).
    2. **Cost Management** → **Budgets** → **Add**: USD 30 a month, alerts at 50%, 80%, 100%.
  - Done when: Both appear in the portal.
  - Why: One folder for everything, deletable in one click. The budget is your smoke detector.
- [ ] **w4-3 · Foundry project + one small model** · 45 min · by hand
  - Do:
    1. Sign in and create a project in `rg-football-insights`.
    2. Deploy one small, low-cost chat model.
    3. Copy the project endpoint. It looks like `https://<resource>.services.ai.azure.com/api/projects/<project>`.
  - Done when: The playground answers "hello".
  - Why: Foundry is the headline technology of the brief: models and agents in one place. Button names change often; follow Microsoft's quickstart if the screen differs.
  - Link: [Open Microsoft Foundry](https://ai.azure.com)
- [ ] **w4-4 · Call the model from Python, no API key** · 45 min · with Claude
  - Do:
    1. Portal → your Foundry resource → **Access control (IAM)** → **Add role assignment** → **Foundry User** → your own user → **Review + assign**.
    2. In the codespace, run `az version`. If it's missing, install it with the first command below.
    3. Log in, install the SDK, run Microsoft's first-call sample with your endpoint.
    4. If Microsoft's sample uses `AzureCliCredential`, switch it to `DefaultAzureCredential`. It works on your codespace now and on Azure later.
    5. Replace the sample's model name with your own deployment name.
  - Command:
    ```bash
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    az login --use-device-code
    pip install "azure-ai-projects>=2.3.0" azure-identity
    ```
  - Done when: The script prints a model answer.
  - Why: No key anywhere means nothing can leak from a public repo. It's your Entra RBAC skill applied directly. A 403 right after the role assignment usually means: wait 5–10 minutes.
- [ ] **w4-5 · Narrator: one fact card → one insight** · 1 h · with Claude
  - Do:
    1. In `src/explain/narrator.py`, send ONE fact card.
    2. Ask for JSON wording with named slots, e.g. `{player_surname}`, `{xg_pct}`, `{goals_season_ord}`. The model never types numbers.
    3. Code fills the slots from the card.
  - Done when: Valid, filled wording for five different cards.
  - Why: Get the simplest version working before adding agents. If this fails, agents won't fix it.
- [ ] **w4-6 · Verifier in plain code** · 1 h 15 min · with Claude
  - Do:
    1. In `src/explain/verify.py`, reject text with a number or number word ("fourth", "twice", "hat-trick") that didn't come from a slot.
    2. Reject names not in the card (surname alone is allowed), comparisons ("most", "season high") with no `compare` field, win probabilities drawn from Elo ("a 76% chance"), and body or intelligence words ("pacy", "clever") with no card field.
    3. Fail → ask for one rewrite. Fail again → template.
  - Done when: A planted "his fourth hat-trick" and a planted "pacy Vardy" both get rejected.
  - Why: A broadcaster can never show a wrong stat on air. Code can't hallucinate, and the bias guard is also a StatsBomb licence rule.
- [ ] **w4-7 · Template fallback** · 30 min · with Claude
  - Do:
    1. Reuse the w3-3b templates as the fallback.
    2. The AI gets 1.2 seconds of real time and one retry; then the template goes out.
    3. Log `source: ai` or `source: template` for every insight.
  - Done when: With the model switched off, the replay still shows every insight, logged as `source: template`.
  - Why: Like a failover link: the screen never goes blank because one dependency is slow.

## Week 5 · Agents and personalization

Dates: Mon Nov 9 – Sun Nov 15 (2026-11-09 → 2026-11-15) · Total: ≈6 h 30 min · **Win:** Two agents, two audiences, English and French.

- [ ] **w5-1 · Run your first agent** · 45 min · with Claude
  - Do:
    1. Install the agent framework.
    2. Run Microsoft's "your first agent" sample with your endpoint and model.
    3. If Microsoft's sample uses `AzureCliCredential`, switch it to `DefaultAzureCredential`. It works on your codespace now and on Azure later.
    4. Replace the sample's model name with your own deployment name.
  - Command:
    ```bash
    pip install agent-framework-foundry
    ```
  - Done when: The agent answers.
  - Why: Learn the framework on a toy before your own code.
- [ ] **w5-2a · Analyst agent with a tool** · 1 h 15 min · with Claude
  - Do:
    1. Write a normal Python function `get_match_state(minute)`.
    2. Give it to an Analyst agent as a tool.
    3. The Analyst picks which fact cards matter and says why.
  - Done when: The Analyst calls the tool and returns a short reasoning note.
  - Why: Agentic design is a fifth of the brief. Each agent has one job, like roles in a network.
- [ ] **w5-2b · Narrator agent + verify gate** · 1 h 15 min · with Claude
  - Do:
    1. A Narrator agent turns the Analyst's note into text for a profile.
    2. Your verifier code checks it, with one rewrite allowed.
    3. Log every hand-off.
  - Done when: One card flows Analyst → Narrator → verifier, and the log shows each step.
  - Why: The verifier stays code because a model checking a model can make the same mistake.
- [ ] **w5-3 · Overlay JSON** · 45 min · with Claude
  - Do:
    1. In `src/render/overlay.py`, turn each verified insight into: match_clock, display_seconds, headline, text, evidence_ids, language.
  - Done when: A full match produces overlays in time order.
  - Why: Timed, machine-readable output is what broadcast graphics systems consume.
- [ ] **w5-4 · Two profiles + EN/FR** · 1 h · with Claude
  - Do:
    1. **Analyst**: numbers-heavy. **Casual fan**: plain words.
    2. Each in English or French.
  - Done when: The same card reads clearly differently for each profile.
  - Why: Personalize is a required stage. Two clearly different outputs prove the point.
- [ ] **w5-5a · Streamlit: profile + language picker** · 45 min · with Claude
  - Do:
    1. Add a sidebar with profile and language.
  - Done when: Switching profile changes new insights.
  - Why: The viewer controls the story, which is the whole point of personalization.
- [ ] **w5-5b · Streamlit: verified insights in the feed** · 45 min · with Claude
  - Do:
    1. Replace fact cards in the feed with verified AI insights.
    2. Keep the **Why?** pitch view on each one.
  - Done when: The feed shows AI insights with their evidence.
  - Why: This is the demo moment for the video.

## Week 6 · Milestone 2: ship it (MILESTONE)

Dates: Mon Nov 16 – Sun Nov 22 (2026-11-16 → 2026-11-22) · Total: ≈6 h 15 min · **Win:** Live on Azure, a 2-minute video, a blog post, tag v1.0.

- [ ] **w6-1 · Write the Dockerfile** · 45 min · with Claude
  - Do:
    1. Add the Azure packages from weeks 4–5 to requirements.txt.
    2. Make the app fetch and convert match 3754033 at start-up if `data/match_3754033.jsonl` is missing.
    3. Add a `.dockerignore` with `data/raw/` and `data/*.jsonl`: no raw or converted event files in the image.
    4. Create the Dockerfile below, then build and run it.
  - Command:
    ```dockerfile
    FROM python:3.12-slim
    WORKDIR /app
    ENV PYTHONPATH=/app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY src/ src/
    COPY data/season_baselines.json data/season_baselines.json
    EXPOSE 8501
    CMD ["streamlit", "run", "src/app/main.py", "--server.address", "0.0.0.0", "--server.port", "8501"]
    ```
  - Done when: `docker build -t football-insights .` then `docker run -p 8501:8501 football-insights` opens the app.
  - Why: A container runs the same everywhere. Fetching at start-up keeps StatsBomb data out of a public image, as the licence requires.
- [ ] **w6-2 · Deploy to Azure Container Apps** · 1 h · with Claude
  - Do:
    1. Check the code uses `DefaultAzureCredential` everywhere (you switched in w4-4).
    2. Run the commands below.
    3. Portal → the container app → **Identity** → System assigned **On** → Save.
    4. Give that identity **Foundry User** on the Foundry resource.
    5. Add your project endpoint as an environment variable on the app.
  - Command:
    ```bash
    az extension add --name containerapp --upgrade
    az provider register --namespace Microsoft.App
    az provider register --namespace Microsoft.OperationalInsights
    az containerapp up --name football-insights --resource-group rg-football-insights --source .
    ```
  - Done when: The public URL works on your phone.
  - Why: The managed identity is the app's own account, like a computer account in Active Directory. No secrets exist anywhere.
- [ ] **w6-3 · Logs you can quote** · 30 min · with Claude
  - Do:
    1. Print one JSON log line per insight: time from event to screen, tokens, verifier pass or fail, `source` (ai or template).
    2. Query them in Log Analytics with KQL.
  - Done when: One KQL query returns your pass rate and fallback rate.
  - Why: "92% verified, 1.4 s per insight" makes your README and interviews concrete.
- [ ] **w6-4a · Write the video script** · 30 min · with Claude
  - Do:
    1. About 250 words.
    2. Problem 10 s · live replay with an insight 40 s · Why? evidence 20 s · profile + language switch 20 s · architecture 20 s · what's next 10 s.
  - Done when: Read aloud, it's under 2 minutes.
  - Why: The script keeps you under 2:00, the same limit as the official rules.
- [ ] **w6-4b · Record and upload the video** · 1 h · by hand
  - Do:
    1. Record your screen with the script.
    2. Upload to YouTube.
    3. No copyrighted music, no Premier League logos.
  - Done when: Under 2:00 and uploaded.
  - Why: Most people will watch the video, not run the code.
- [ ] **w6-5 · README** · 1 h · with Claude
  - Do:
    1. Pitch paragraph, architecture diagram, run steps, your numbers from the logs.
    2. Add: "Independent project inspired by the Microsoft × Premier League Inside the Game brief. Not an official entry. Data: StatsBomb open data (Premier League 2015/16), not for commercial use."
    3. Add the StatsBomb logo.
  - Done when: README is pushed.
  - Why: Honest and clear beats flashy. It shows professionalism.
- [ ] **w6-6a · Blog post** · 1 h · with Claude
  - Do:
    1. Turn your devlog into a post on mradelvand.github.io.
    2. Include the diagram and the video.
  - Done when: The post is live.
  - Why: This is the link you'll send to clubs and sports-tech companies.
- [ ] **w6-6b · LinkedIn + winners comparison + v1.0** · 30 min · by hand · milestone
  - Do:
    1. After winners are announced (by Nov 20), add one paragraph: what they did differently.
    2. Post on LinkedIn.
    3. Tag the release.
  - Command:
    ```bash
    git tag v1.0 && git push --tags
    ```
  - Done when: Post is live and v1.0 is tagged.
  - Why: Full time. You shipped a real football AI project.

## Later (from December)

Ideas live in `docs/parking-lot.md`. Top candidates: CI/CD with GitHub Actions and OIDC, Event Hubs ingestion, an MCP tool server, Fabric Real-Time Intelligence, more detectors and profiles, then the MIT Sloan Sports Analytics Conference hackathon (Feb 2027).

## Changes from the Oct 11, 2026 review (pre-match context)

- **w4-0:** adds Elo, form, unbeaten runs and head-to-head, with known values as tests, +45 min.
- **w4-0b (new):** pre-match context card and upset watch, 30 min. Second demo match: 3753976, Arsenal 1–2 Swansea.
- **w4-6:** the verifier also rejects Elo win-probability wording.
- **Totals:** 38 h 20 → 39 h 35 required.

## Changes from the Oct 9, 2026 review (StatsBomb + UK insights)

- **Data:** real StatsBomb open data (Premier League 2015/16) replaces the simulator. Demo match: Leicester 3–1 Everton, 2016-05-07. Licence rules in `docs/DATA.md`; product brief in `docs/brief.md`.
- **w1-0 (new):** register with StatsBomb, 5 min.
- **w1-2:** StatsBomb fields, 120 × 80 pitch, `src/ingest/schema.py`.
- **w1-3, w1-4a, w1-4b, w1-5:** fetch, converter, clock and own goals, sanity check replace the team file and simulator (−1 h 15 net).
- **w2-3c:** PPDA zone is now x < 72 / x > 48.
- **w2-4 removed:** no realism check needed on real data.
- **w3-2a, w3-2b:** the seven UK insight types, cards with `priority` and `compare`.
- **w3-2c (new):** priority queue and cooldowns, +45 min.
- **w3-3b:** template insights in the feed, +15 min.
- **w4-0 (new):** season baselines, earlier matches only, +1 h.
- **w4-5, w4-6:** slot wording; verifier also checks names, number words, comparisons and the bias guard (+15 min).
- **w4-7 (new):** template fallback with a 1.2 s budget, +30 min.
- **w6-1, w6-3, w6-5:** no event files in the image; event-to-screen time and fallback rate in logs; StatsBomb credit in the README.
- **Totals:** 36 h 30 → 38 h 20 required.

## Changes from the Oct 8, 2026 review

- **w0-4b (new):** install Claude Code and paste one setup prompt that builds the workspace, including a dev container that installs Python packages, Azure CLI, Docker and Claude Code on every rebuild.
- **w0-5 removed:** GitHub Copilot overlaps with Claude Code and with "typing builds memory".
- **w0-6, w0-7 changed:** the dev container replaces the venv; `.gitkeep` files keep empty folders in Git; `pyproject.toml` sets pytest's `pythonpath` so imports between src folders work.
- **w0-7b (new):** rebuild the codespace so the dev container applies, then restart Claude Code.
- **w0-9 changed:** `/done` writes the devlog.
- **w2-3c:** the PPDA zone flips between teams (opponent x < 63 = your x > 42), with a test from both sides.
- **w3-3a:** 1 h 15, pair mode, run Streamlit with `PYTHONPATH=.`.
- **w4-4, w5-1:** use `DefaultAzureCredential` from the start and your own deployment name.
- **w6-1:** Dockerfile sets `ENV PYTHONPATH=/app`.
- Checked on Oct 8, 2026: Foundry User role, `azure-ai-projects` 2.8.0 (so `>=2.3.0` is fine), `agent-framework-foundry` 1.13.1, `az containerapp up` prerequisites, Azure free account (USD 200 for 30 days).
