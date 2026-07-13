# IPL Ball-by-Ball SQL Analytics (2008–2024)

A SQL analytics project on 17 seasons of Indian Premier League cricket — 1,095 matches and 260,920 individual deliveries — covering team performance, player statistics, toss strategy, and in-match scoring patterns.

Built as part of my SQL portfolio for business analyst roles  

## Why this project

Most cricket-dataset portfolios stop at "who scored the most runs." This project is framed around **business-style questions** instead — the kind an analyst would actually be asked to answer, not just descriptive trivia:

- Does winning the toss and choosing to field actually carry a measurable advantage?
- At what target score does chasing stop being reliably successful?
- Is T20 scoring genuinely getting more aggressive over time, or does it just feel that way?
- Does "home advantage" show up in the data at all?

## Data source

- **Origin**: [IPL Complete Dataset (2008–2024)](https://www.kaggle.com/datasets/patrickb1912/ipl-complete-dataset-20082020) by patrickb1912 on Kaggle
- `matches.csv` — 1,095 rows, one per match
- `deliveries.csv` — 260,920 rows, one per ball bowled (not included in this repo due to size — see setup below)

## Data cleaning — real issues found and fixed

This is real, messy data, not a hand-crafted toy dataset. Issues found and how they were handled (see `scripts/01_clean_and_load.py`):

- **Whitespace in column headers** — `deliveries.csv` was exported with headers like `' inning '` and `' batting_team  '` (stray leading/trailing spaces). Stripped before loading.
- **`'NA'` string literals instead of true NULLs** — both files use the literal text `"NA"` rather than a blank value, which silently breaks aggregate functions (`COUNT`, `AVG`, etc. can't tell the difference between "no dismissal" and the string "NA"). Converted to proper SQL `NULL`.
- **A genuine data-entry typo**: `"Rising Pune Supergiant"` vs `"Rising Pune Supergiants"` — same franchise across two seasons (2016/2017), spelled inconsistently. This one **was** normalized.
- **Franchise renames were deliberately NOT merged** — e.g. Delhi Daredevils → Delhi Capitals, Kings XI Punjab → Punjab Kings, Deccan Chargers → Sunrisers Hyderabad, Royal Challengers Bangalore → Royal Challengers Bengaluru. Unlike the typo above, these are genuinely different franchise eras in IPL's own record-keeping, so collapsing them would misrepresent team history rather than clean it. This is a judgment call, documented here rather than hidden.

## Schema

Two base tables (`matches`, `deliveries`) plus four views built specifically so a BI tool can connect to clean, pre-aggregated data instead of raw ball-by-ball rows:

| View | Purpose |
|---|---|
| `vw_match_summary` | One row per match with derived fields (batting-first team, toss-winner-won flag) |
| `vw_team_season_summary` | Team performance aggregated by season (matches, wins, win %) |
| `vw_batter_season_stats` | Batter stats per season (runs, strike rate, fours/sixes, dismissals) |
| `vw_phase_wise_scoring` | Powerplay / Middle / Death overs scoring breakdown per innings |

6 indexes were added on columns used repeatedly in `GROUP BY`/`JOIN`/`WHERE` across the 25 queries — `deliveries` is 260K+ rows, so this is a real performance consideration, not decoration.

## The 25 queries

Grouped into 4 sections — see `queries.sql` for full SQL:

1. **League overview** (Q1–4): match counts, most successful teams, venue distribution
2. **Batting analysis** (Q5–12): boundary dependency %, favorite opposition per batter, strike rate/average leaders, batters never dismissed for a duck
3. **Bowling & match dynamics** (Q13–17): venue scoring tendencies, bowler-vs-batter dismissal patterns, best totals-defenders, 200+ score upset losses
4. **Toss/phase/chase/home analysis** (Q18–25): toss decision impact, phase-wise scoring trends, chase success rate by target range, home advantage

### A few real findings

- **Chase success rate drops sharply with target size**: 82.3% success chasing under 140, falling to just 19.2% for 200+ targets — a clean, monotonic relationship (Q22)
- **Fielding first after winning the toss wins more often**: 53.9% win rate vs. 45.4% for batting first (Q18) — quantifies a widely-believed but rarely measured IPL strategy
- **Home advantage is measurable but not huge** — see Q23 for team-by-team home vs. away win rates (methodology note: "home city" is inferred as the city each team has played in most often, since IPL doesn't have a single fixed home-venue column — not ground truth)



## Planned next step: Power BI

The four views are intentionally structured to be BI-tool-ready — a Power BI report connecting directly to `vw_match_summary`, `vw_team_season_summary`, `vw_batter_season_stats`, and `vw_phase_wise_scoring` (rather than the raw 260K-row `deliveries` table) is the next planned addition to this project.

## Tools

PostgreSQL 16 · Python (pandas, SQLAlchemy) for data loading
