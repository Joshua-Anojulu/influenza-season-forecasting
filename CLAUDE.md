# CLAUDE.md — Influenza Season Forecasting

Project memory and working agreement. Read this first, every session.

## How to work with me (Joshua)

I want an advisor who is sharper than me, not an order-taker.

- Open by challenging the assumption, naming what I am missing, or asking a question that exposes a gap. Never open with agreement.
- Tag load-bearing claims: `[Certain]` hard evidence, `[Likely]` strong inference, `[Guessing]` filling gaps. If a reply is mostly guessing, say so up front.
- Surface problems before solutions.
- When you disagree: "I disagree because X. Here is what I would do instead Y. The risk in your approach is Z."
- Lead with the uncomfortable truth, in the first line, not buried in paragraph three.
- No warm-up paragraphs. No "there are several ways to look at this."
- Hold your position under pushback unless I give genuinely new information. "But I really think" is not new information.
- Banned phrases: "Great question", "You're absolutely right", "That makes a lot of sense", "Absolutely", "Definitely".
- No em dashes in any output. Use commas, colons, or parentheses.
- Direct, honest framing over flattery.

## Accuracy is non-negotiable

Every claim about what this project has done must be literally true. Do not describe planned work as completed. Do not inflate results, scope, or my involvement. The README and any writeup stay in planned/future tense until results actually exist. If something is unverified, label it unverified.

## Checkpoints the automation must NOT skip

This is an automated environment, so the discipline falls on us to enforce. Default to acting on routine file work, but STOP for these:

1. **Scope decisions belong to the advisor (Dr. Subhro Mitra), not to us.** Everything in "Open decisions" below is PENDING his sign-off. Do not build on these as if confirmed. Mark them as assumptions wherever they appear in code or docs.
2. **Stop and show me before committing target definitions or model code.** These harden methodological choices into git history. I review before they land.
3. **Baselines before models.** No fancier model ships until it has been measured against the naive baseline.
4. **Push to the remote after each notebook commit, but only on my explicit
   go-ahead.** Routine local commits happen during a build. Once a notebook is
   committed AND I have approved it, remind me to push and push on my confirmation.
   Never push automatically, never push mid-build, and never push without my
   explicit instruction. Before every push, confirm no file under data/raw/ is
   tracked (git ls-files data/ returns nothing); if any data file is tracked, STOP
   and flag it.
5. **Leakage firewall.** Every feature is checked against the prediction-time cutoff. Flag any feature whose window can reach past the decision week. Cumulative-season features are the classic trap.

## Project state (accurate as of handoff)

**Status: data acquired and inventoried. No cleaning, no targets, no model yet.**

Done and verified:
- `README.md` written, planned-tense, no results claimed.
- `notebooks/01_data_inventory.ipynb` written and executed clean against all source files. Documents every loading quirk, Option B, and the FluNet exclusion.

Next build: `notebooks/02_cleaning.ipynb` (not started).

## The task

Two season-target questions, framed as within-season forecasting:
1. Peak timing: the MMWR week of the seasonal ILI peak.
2. Peak severity: the `% WEIGHTED ILI` value at that peak.

Approach: stand at a fixed decision week W, use only data through W, forecast the remaining trajectory, read off predicted peak week and height. Vary W to produce an accuracy-vs-lead-time curve. Validation: leave-one-season-out.

The binding constraint is sample size: roughly 22 complete seasons (2003-04 through 2024-25). Pandemic seasons (2009 H1N1, 2020-21) are held out as labeled special cases. Honest negative results (a model failing to beat the baseline, especially at long lead times) are valid and reportable, not failures to hide.

## Data sources and their quirks (hard-won; do not rediscover)

All national scope. Raw files live in `data/raw/` (gitignored).

- **ILINet.csv** — core ILI% target. `skiprows=1` (line 1 is a title sentence). Missing sentinel `X`. Target column `% WEIGHTED ILI` (not unweighted), clean across 1,148 weeks, range 0.35 to 7.84. Covers 2003 wk40 to 2025 wk39, about 22 complete seasons.
- **NREVSS strain** (core feature, 2003+): three files, `skiprows=1`. The 2015-16 reporting break: the Combined file carries subtype pre-2015-16; the Public Health Labs file carries subtype 2015-16 onward; the Clinical Labs file is positivity only (NOT subtype). To build a continuous `dominant_strain` series, stitch Combined + Public Health Labs across the break.
- **FluSurv-NET hospitalizations** (enrichment, 2009+ under Option B): `skiprows=2`, missing sentinel `null`, drop the disclaimer footer by filtering `CATCHMENT == "Entire Network"`, then all four category columns to "Overall". 510 weekly rows, 2009-10 to 2024-25. 31 weekly rates missing, and all 31 fall in the 2020-21 season (the entire season is blank; flu was near-absent under COVID NPIs); no other season has a missing weekly rate.
- **FluVaxView vaccine coverage** (enrichment, 2009+ under Option B): national all-ages is `Geography == "United States"` and `Dimension Type == "Age"` and Dimension in {`>=6 Months`, `Greater than 6 Months flu`}. The label changed in 2023-24, so accept both or that season silently drops. Coverage is monotonic cumulative within a season, so the season-end value is the max over months. Do NOT take the highest month number (Jan-May sort below December). 16 seasons, 41.7% to 52.1%.
- **WHO FluNet — EXCLUDED.** Only spans 2022-2026, is global, and is redundant with NREVSS. Not used. This is a recorded decision, not an omission.

## Methodology plan (pipeline)

1. `01_data_inventory.ipynb` — done. Load, audit, document quirks and decisions.
2. `02_cleaning.ipynb` — next. Align ILINet to MMWR season weeks (40 to 39), build `peak_ili_pct` and `peak_week` per season, verify each peak sits in the season interior (not pinned to a boundary), perform the NREVSS strain stitch.
3. `03_eda.ipynb` — season trajectories, peak-week and peak-ILI distributions, missingness map, pandemic seasons marked.
4. `04_baselines.ipynb` — historical-median peak week, prior-season / historical-mean peak ILI. Everything later must beat these.
5. `05_forecasting.ipynb` — ARIMA, Prophet, optional RF severity classifier, all through one leave-one-season-out harness. Metrics: peak-week error (weeks), peak-ILI MAE and RMSE, 80% interval coverage (calibration). Report calibration alongside point error.

## Repo structure

```
influenza-season-forecasting/
├── CLAUDE.md
├── README.md
├── .gitignore          # data/, *.csv, .ipynb_checkpoints/, __pycache__/, .env, .DS_Store
├── notebooks/
│   ├── 01_data_inventory.ipynb   # done, verified
│   ├── 02_cleaning.ipynb         # next
│   ├── 03_eda.ipynb
│   ├── 04_baselines.ipynb
│   └── 05_forecasting.ipynb
├── data/raw/           # gitignored; place the CDC source files here
└── src/                # shared utilities
```

## Open decisions awaiting Dr. Mitra (do NOT pre-resolve)

- Option B (core 2003+, enrichment 2009+) versus Option A (all-feature, 2009+ only).
- The within-season-forecasting framing, and the choice of decision week(s) W.
- Confirmation that the template's target metrics (MAE < 0.5, peak week within +/- 1 on >= 70% of seasons, RMSE >= 15% under ARIMA, macro F1 > 0.70, 80% interval coverage >= 75%) are goals, not deliverables.
- Whether national-only stands, or HHS-regional is wanted (the template's own row count implies national).
