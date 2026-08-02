# CLAUDE.md: Influenza Season Forecasting

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

## Project state (updated 2026-08-02)

**Status: the pipeline is complete through notebook 08. Notebooks 01-07, all 14 `results/` artifacts, all 12 `figures/`, and the findings deck are on `main` and pushed to `origin/main`; notebook 08 and its four `results/08_*` artifacts are committed locally but NOT yet pushed, so `main` is ahead of `origin/main` and `git rev-list --left-right --count origin/main...main` no longer returns `0 0`. Pushing needs Joshua's explicit go-ahead (checkpoint 4). The template-adherence pass landed as `e65cc1d` on 2026-07-21 after Claude independently re-ran the normalized protected gate (PASS 19/19) and reviewed the full diff; Act 3 of `PLAN-REVIEW-LOG.md` records the verdict. Results are preliminary and descriptive. Dr. Mitra signed off on framing, scope, and the regression/curve models on 2026-07-08, and on the D2 heatmap, severity tiers, and Option B on 2026-08-02 (see both "Confirmed by Dr. Mitra" sections); only the headline-W decision remains open below.**

**No program affiliation (confirmed by Joshua, 2026-08-01).** This project is not part of the UNT AI Summer Research Program, the NSF REU, or any other structured program. Dr. Mitra advises Joshua directly. There is therefore no inherited rubric, no poster-session date, and no externally imposed deadline or format. Do not infer deliverable requirements from any program calendar; the "template" that defines D1-D4 is the project's own, and its origin is recorded nowhere in this repo.

**Nothing is blocked on Dr. Mitra any more (as of 2026-08-02).** His reply to the 2026-07-21 follow-up cleared D2, the severity tier definition, and Option B; see "Confirmed by Dr. Mitra (2026-08-02)".

**2026-08-02 severity-tier pass (notebook 08).** Built to `PLAN-severity-tiers.md`, which was locked by grill and approved by Codex at round 3 of 5; the argument transcript is `PLAN-REVIEW-LOG-severity-tiers.md` and provenance validates as `approved-final` against the final body hash. Joshua approved the tier definition at checkpoint 2 on 2026-08-02. The notebook executes clean, outputs cleared, and all four `results/08_*` artifacts reproduce byte-identically on a fresh run.

- **The tier definition.** `IT50=4.4 / IT90=6.6 / IT98=8.6` percent of outpatient visits for ILI, all ages, from Biggerstaff M, et al. Am J Epidemiol. 2018;187(5):1040-1050, doi:10.1093/aje/kwx334. A season is tiered by `season_peak_max`, its peak weekly **raw** `% WEIGHTED ILI`. That is what CDC classifies on; the geometric mean of the three highest weeks is only how the thresholds were *estimated*, and conflating the two was caught in review. `peak_ili_pct` is untouched and remains the regression target for 05/06/07.
- **This is an ILI-only approximation, not CDC's classification**, and every artifact says so. CDC classifies by a 2-of-3 indicator vote over ILI, hospitalization rate, and P&I mortality; this project has ILI from 2003, hospitalization from 2009, and no mortality data.
- **Tier counts, 19-season modeling set: Low 4, Moderate 9, High 6, Very High 0.** No season in 22 reaches IT98, so Very High is empty by observation and the classifier is 3-class. 2020-21 is `Not assessed`, because CDC did not assess it.
- **Validation: 9/12 against CDC's published classifications overall, 9/10 among comparable seasons.** Always report both, overall first; the two excluded seasons are two of the three disagreements. 2009-10 is the pandemic season; 2008-09 is pandemic-adjacent, and our 4.886 peak against CDC's published 3.6 is a season-boundary difference (our MMWR season runs to wk39 and absorbs the spring 2009 H1N1 wave), not a framework difference. The sole comparable-season disagreement is 2014-15.
- **The classifier is a negative result.** The RF does not demonstrate an advantage over `B3`, thresholding the univariate cumulative-ILI regression. Paired bootstrap intervals for RF minus B3 include zero at W=12 and W=16; only W=8 excludes it, at a +0.003 lower bound, out of three inspected comparisons. The apparent edge is largest at the earliest decision week and decays to +0.056 by W=16, and macro-F1 is non-monotonic in W (0.634 at W=8 versus 0.456 at W=12). Both patterns are noise signatures. Do not cite the RF as beating a baseline.
- **New evidence for the holiday-artifact finding.** In all three seasons where the raw and smoothed statistics disagree on tier (2013-14, 2019-20, 2023-24), the unsmoothed CDC statistic tiers *higher*, which is the direction 02's holiday-artifact finding predicts. `tier_smoothed` is a mandatory sensitivity, not an optional one.
- **Five boundary-sensitive seasons** carry `tier_boundary_sensitive`: 2010-11, 2013-14, 2019-20, 2021-22, 2023-24. Any claim resting on one of their tiers is fragile.

**D2 is the one thing still not built, and it is blocked on a manual download, not on a decision.** It needs `data/raw/ILINet_regional.csv` from CDC FluView with region type "HHS Regions"; the current `data/raw/ILINet.csv` is National-only. Notebook 08 guards on the file and records the block in `results/08_severity_classifier.json` rather than skipping silently. When built, D2 is a **continuous** heatmap of regional season peak with national thresholds as colorbar reference marks only, never regional tiers: CDC publishes no regional intensity thresholds, so a regional tier would be a label the thresholds do not license.

**Sequencing note.** D4, the research paper, is the only remaining item with no upstream dependency and was never blocked on Dr. Mitra: `PLAN.md` deferred it purely on sequencing, "after these figures exist", and they exist as of `e65cc1d`.

**Google Drive is 11 days behind git and the folder names are misleading.** The folder Dr. Mitra and two collaborators actually read is `Influenza Season Forecasting` (id `1shu3UBcQdomqO5uOCaA_4q9SSQV4DZKe`), which is ALSO `anyone with link: reader`, i.e. publicly readable. The folder named `Influenza Season Forecasting — shared` (id `1YCjOr5e_zq0k6V_Kz_hfMRMexEpKuYPw`) is private to Joshua and is a superseded 2026-07-08 copy. The live folder is missing every artifact from the template-adherence pass: notebook 07, D1, D3, `05_special_cases.*`, `05_trajectories.json`, `07_features_summary.*`. Do not treat Drive as a mirror of `main`.

**2026-07-21 template-adherence pass (committed as `e65cc1d`).** Notebook 05 now persists 132 unique per-fit trajectories in a sidecar, scores the three excluded seasons separately as structural-break stress tests, and builds D1. Notebook 06 persists its already-computed standardized ridge coefficients under the sole whitelisted new JSON key. New notebook 07 adds the template lag, rolling, and hospitalization features; tests H1; builds D3 with grouped block ablations; and scores excluded seasons under Panel A. The protected gate passes 19/19 after line-ending normalization, with every pre-existing 06 JSON key unchanged. Notebook outputs are cleared before review.

**2026-07-10 accuracy pass (pre-share).** Before consolidating materials for the lab, three claims were checked against `results/` and corrected: (1) the deck asserted ARIMA's defined peaks "do not beat the floor", which contradicted `pw_skill = +3.00` at W=16 in `05_forecasting_summary.md`; the slide now reports that number with its selection effect (see below). (2) The deck's "14 of 17 seasons fall under the interval" was true but recorded nowhere; it is now persisted. (3) This file claimed the 06 work was uncommitted, which was false. New artifact: `results/05_survivorship.{md,json}`, generated by a diagnostic cell in notebook 05.

**2026-07-10 figure pass.** Notebook 03's EDA figures were built on `~is_pandemic` (20 seasons, keeping pandemic-adjacent 2008-09) while every model uses 19. 2008-09's smoothed peak is wk38, the last week of the MMWR season (2009 H1N1 spring wave), so it injected a boundary artifact into the timing distribution and inflated `A(H1N1)` to n=7. Figures 03/04/06 now use the 19-season modeling set (`is_excluded`), asserted in 03; strain n is 6/10/3. Figure 09 annotated the Gaussian's shrinking sample (n=19/18/5) and marks the W=16 point as not comparable; the deck's severity chart matches. Re-running 06 reproduced `results/06_*` byte-identically, so no result changed.

Committed and verified baseline state (each notebook executed clean; committed without outputs; reconstructs 02's logic from raw):
- `01_data_inventory.ipynb`: load/audit all sources; documents quirks, Option B, FluNet exclusion.
- `02_cleaning.ipynb`: MMWR season alignment; targets `peak_week` / `peak_ili_pct` on a 3-week centered smoother (removes the wk52 holiday artifact); flags `holiday_shift` (5 seasons), `peak_week_smoothing_sensitive`, `fragile_peak_week` (9 seasons); NREVSS `dominant_strain` stitch (validated at the 2015-16 seam). 22 complete seasons.
- `03_eda.ipynb`: trajectories, distributions, strain timeline, missingness map (surfaced that FluSurv's 31 missing weekly rates are all 2020-21, correcting an earlier note). Figures in `figures/`.
- `04_baselines.ipynb`: LOSO floors on 19 seasons (exclude 2008-09, 2009-10, 2020-21). Finding: peak-WEEK timing has no strong naive floor (~3.3-3.8 wk MAE, <=37% within +/-1); severity is more tractable (climatology MAE 1.34, within-season running max 0.84 by W=16). Lead-time-matched bars for 05. Results in `results/`.
- `05_forecasting.ipynb`: ARIMA + Prophet within-season under a strict leakage firewall (audited clean across 114 fits: last_obs <= W, peak read only from the forecast region, no leak). Honest negative point-forecast result: models do not beat the floor at realistic W. **The one apparent exception is ARIMA timing at W=16** (`pw_skill = +3.00` vs baseline C): it survives only because peak-week metrics drop `peak_ambiguous` (plateau) forecasts, and the 5 dropped seasons are the severe ones (mean peak ILI 5.75 vs 4.29 for the 7 kept). Against LOSO climatology on the same 7 seasons the edge is 0.29 wk at n=7, with 1 of 7 within +/-1. Quantified in `results/05_survivorship.md`; do not cite the +3.00 without it. Primary affirmative finding: Prophet's 80% intervals are cap-pinned near the historical-max ceiling (median interval at W=12 is [7.09, 7.54]) while most peaks fall well below, so empirical LOSO coverage is 5.9-10.5% across W (1-2 of 12-19 seasons) vs nominal 80% (Prophet fit seeded, seed=42, so these are reproducible; an earlier unseeded run read 11.8% at W=12, two seasons sitting within 0.04 ILI of an interval edge) - severely overconfident. At W=12, 14 of 17 seasons fall below the interval, 2 above, 1 inside. RF severity classifier: planned, not implemented.
- `06_regression_and_curve.ipynb`: three models under the identical firewall, all features strictly through-W (firewall audited). (1) Univariate regression (cumulative-ILI-through-W to peak severity): the first model to honestly beat a floor, MAE 1.26/1.12/0.92 at W=8/12/16, beats climatology (1.34) at every W and beats/ties baseline C at W=8,12. (2) Explanatory ridge (retrospective, NOT a forecast; ili+strain on 19, +vaccine on the 2009+ 14-season subset): strain and vaccine coverage add no severity signal beyond cumulative ILI (standardized ILI coef +0.77..+1.13, all strain coefs <0.09). (3) Gaussian curve fit: bound-pinned on most seasons, worse than climatology on severity, rarely a defined peak week, a reported negative. Results in `results/06_*`, figure `figures/09_severity_by_W.png`.

Template-adherence extension (executed clean, outputs cleared, reviewed, committed as `e65cc1d`, pushed):
- `05_forecasting.ipynb` now writes `results/05_trajectories.json` through a module-level sidecar, leaving `res` and the protected audit schema unchanged. It scores 2008-09, 2009-10, and 2020-21 separately in `results/05_special_cases.{json,md}` as excluded-from-training structural-break stress tests, not prospective forecasts. D1 is `figures/10_D1_forecast_overlay.png`, Prophet-only at W=12 because ARIMA has no intervals in this implementation.
- `06_regression_and_curve.ipynb` now persists the existing all-19, lambda=1 standardized ridge coefficients under `ridge_coefficients` in `results/06_regression_curve_summary.json`. No pre-existing key or value changed.
- `07_features_and_hypothesis.ipynb` adds `ili_lag_1..4`, `ili_rolling4`, and `hosp_rate_lag1`. Its index firewall passes across all 66 complete-season/W pairs, but hospitalization, strain, and vaccine remain reporting-lagged, so affected panels are retrospective and explanatory. Panel A is 9 columns on n=19; Panel B is 11 columns on n=14 and high-variance descriptive analysis. Both are marked "Option B assumption, pending advisor decision." H1 receives no descriptive support: lags-only MAE is 1.125/1.189/1.067 versus cumulative-ILI MAE 1.256/1.124/0.917 at W=8/12/16, all paired bootstrap intervals include zero, and sign tests are 10-9 or 9-10. Results are in `results/07_features_summary.{json,md}`; D3 is `figures/11_D3_feature_importance.png` and uses grouped block ablations because rolling4 is exactly determined by the four lags.

Recorded template deviations: `season_week` has zero variance as a season-level predictor at fixed W and remains only the within-season time axis `sw`; ARIMA has no prediction intervals in this implementation, so D1 and interval coverage are Prophet-only.

Open follow-ups (not yet done): empty `src/` with 02's logic duplicated across 03/04/05/06/07/08 (deferred refactor); D2 blocked on the manual regional ILINet download; D4, the research paper, unstarted.

## The task

Two season-target questions, framed as within-season forecasting:
1. Peak timing: the MMWR week of the seasonal ILI peak.
2. Peak severity: the `% WEIGHTED ILI` value at that peak.

Approach: stand at a fixed decision week W, use only data through W, forecast the remaining trajectory, read off predicted peak week and height. Vary W to produce an accuracy-vs-lead-time curve. Validation: leave-one-season-out.

The binding constraint is sample size: roughly 22 complete seasons (2003-04 through 2024-25). Pandemic seasons (2009 H1N1, 2020-21) are held out as labeled special cases. Honest negative results (a model failing to beat the baseline, especially at long lead times) are valid and reportable, not failures to hide.

## Data sources and their quirks (hard-won; do not rediscover)

All national scope. Raw files live in `data/raw/` (gitignored).

- **ILINet.csv**: core ILI% target. `skiprows=1` (line 1 is a title sentence). Missing sentinel `X`. Target column `% WEIGHTED ILI` (not unweighted), clean across 1,148 weeks, range 0.35 to 7.84. Covers 2003 wk40 to 2025 wk39, about 22 complete seasons.
- **NREVSS strain** (core feature, 2003+): three files, `skiprows=1`. The 2015-16 reporting break: the Combined file carries subtype pre-2015-16; the Public Health Labs file carries subtype 2015-16 onward; the Clinical Labs file is positivity only (NOT subtype). To build a continuous `dominant_strain` series, stitch Combined + Public Health Labs across the break.
- **FluSurv-NET hospitalizations** (enrichment, 2009+ under Option B): `skiprows=2`, missing sentinel `null`, drop the disclaimer footer by filtering `CATCHMENT == "Entire Network"`, then all four category columns to "Overall". 510 weekly rows, 2009-10 to 2024-25. 31 weekly rates missing, and all 31 fall in the 2020-21 season (the entire season is blank; flu was near-absent under COVID NPIs); no other season has a missing weekly rate.
- **FluVaxView vaccine coverage** (enrichment, 2009+ under Option B): national all-ages is `Geography == "United States"` and `Dimension Type == "Age"` and Dimension in {`>=6 Months`, `Greater than 6 Months flu`}. The label changed in 2023-24, so accept both or that season silently drops. Coverage is monotonic cumulative within a season, so the season-end value is the max over months. Do NOT take the highest month number (Jan-May sort below December). 16 seasons, 41.7% to 52.1%.
- **WHO FluNet: EXCLUDED.** Only spans 2022-2026, is global, and is redundant with NREVSS. Not used. This is a recorded decision, not an omission.

## Methodology plan (pipeline)

1. `01_data_inventory.ipynb`: DONE. Load, audit, document quirks and decisions.
2. `02_cleaning.ipynb`: DONE. MMWR season alignment (40 to 39); `peak_ili_pct` / `peak_week` on the 3-week centered smoother; boundary + holiday + fragile flags; NREVSS strain stitch.
3. `03_eda.ipynb`: DONE. Season trajectories, peak distributions, strain timeline, missingness map; pandemic / pandemic-adjacent and fragile seasons marked.
4. `04_baselines.ipynb`: DONE. Climatology, persistence, within-season running-max (W in {8,12,16}), exploratory strain-climatology. Lead-time-matched floors; everything later must beat the floor at its own W.
5. `05_forecasting.ipynb`: DONE for ARIMA + Prophet under the LOSO firewall (metrics: peak-week error, peak-ILI MAE/RMSE, 80% interval coverage). The RF severity classifier moved to notebook 08 and is now built. Calibration is reported as a primary result.
6. `06_regression_and_curve.ipynb`: DONE. Univariate severity regression (real-time), explanatory ridge (retrospective; +dominant strain +vaccine coverage), and a Gaussian curve fit, all through-W under the same LOSO firewall. The univariate regression is the first model to beat a floor on severity; strain and vaccine coverage add nothing; the Gaussian does not beat the floor on either target.
7. `07_features_and_hypothesis.ipynb`: DONE, committed as `e65cc1d`. Template lag/rolling/hospitalization features, H1 paired comparison, two Option B ridge panels (Option B confirmed by Dr. Mitra 2026-08-02, so the "provisional" labels are now settled), grouped block ablations, excluded-season Panel A stress tests, and D3. H1 is not descriptively supported.
8. `08_severity_tiers.ipynb`: DONE. CDC-anchored severity tier definition, validation against CDC's published season classifications, and the 3-class RF severity classifier with four baselines. The classifier is a negative result. D2 is guarded and not built, pending the regional ILINet download.

## Repo structure

```
influenza-season-forecasting/
├── CLAUDE.md
├── README.md
├── START_HERE.md       # student-facing getting-started guide
├── requirements.txt
├── .gitignore          # data/, *.csv, .ipynb_checkpoints/, __pycache__/, .env, .DS_Store, AGENTS.md
├── notebooks/
│   ├── 01_data_inventory.ipynb   # done, verified
│   ├── 02_cleaning.ipynb         # done
│   ├── 03_eda.ipynb              # done
│   ├── 04_baselines.ipynb        # done
│   ├── 05_forecasting.ipynb      # done (ARIMA + Prophet; RF not implemented)
│   ├── 06_regression_and_curve.ipynb  # done (univariate regression + ridge + Gaussian)
│   ├── 07_features_and_hypothesis.ipynb  # done (template features + H1 + D3)
│   └── 08_severity_tiers.ipynb       # done (CDC tiers + classifier; D2 blocked on regional pull)
├── data/raw/           # gitignored; place the CDC source files here
├── figures/            # generated EDA + calibration figures (tracked)
├── results/            # baseline + forecasting summaries, md + json (tracked)
├── slides/             # findings deck (PowerPoint / Google Slides)
├── docs/superpowers/   # the 2026-07-08 regression/curve design spec + implementation plan
│                       # (specs/ and plans/); the CURRENT plan and its adversarial review
│                       # are PLAN.md + PLAN-REVIEW-LOG.md at the repo root, not here
└── src/                # shared utilities (currently empty)
```

## Confirmed by Dr. Mitra (2026-07-08)

- **Framing: characterization study first.** Lead with the precise two-pronged result (peak timing near the naive floor; off-the-shelf models severely overconfident, 5.9-10.5% interval coverage vs 80%). Do not bury the honest finding chasing a better RMSE. This implies the template's target metrics are goals, not deliverables.
- **Scope: national now.** Get the national methodology clean; one or two HHS regions as a later robustness check, not now.
- **Models: add a regression approach and a phenomenological curve** (done in 06). Ridge with strain + vaccine coverage was requested; built leakage-safe and split into a real-time univariate forecast plus a retrospective explanatory ridge (strain/vax are lag-reported/survey-revised, not real-time).
- **Presentation:** ~10-12 min deck (motivation+data 2 slides, holiday-artifact 1, model comparison 2-3, Prophet calibration 1, next steps 1).
- **Sharing:** consolidate shareable materials in a Google Drive folder; Joshua shares manually.

## Confirmed by Dr. Mitra (2026-08-02)

Reply to the 2026-07-21 follow-up, received 2026-08-02. He answered three of the three questions asked, plus volunteered a fourth confirmation. Verbatim source is `docs/advisor/2026-08-02-mitra-reply.md`.

- **D2 heatmap: option 2. Pull regional ILI for the heatmap only; the models stay national.** His words: it "satisfies the template requirement and costs you little, since ILINet already provides it." This does NOT reopen the national-versus-regional modeling scope confirmed on 2026-07-08; regional data enters as a descriptive figure input and nothing else. D2 is unblocked.
- **Severity tiers: use CDC's published intensity thresholds, not self-defined quantiles.** His reasoning: "anchoring to the literature is cleaner for a student project and more defensible." The tier definition is unblocked as a decision but is still a target definition, so working-agreement checkpoint 2 applies: Joshua reviews the concrete threshold mapping before it lands. The exact CDC framework and its published values must be cited from source, not reconstructed from memory.
- **Option B is CONFIRMED: core 2003+, enrichment where it exists 2009+. Do NOT restrict to 2009+.** His words: "you would lose too many seasons." This resolves the longest-standing open decision in this repo. Every "Option B assumption, pending advisor decision" label in notebook 07, `results/07_features_summary.*`, D3, and the two-panel structure is now a confirmed choice rather than a provisional one, and those labels should be updated to say so.
- **Pandemic seasons scored pure out-of-sample stay as built.** He called the documentation "a good instinct. Keep it." Confirms `results/05_special_cases.*` and the structural-break framing.

**Unblocked by the above, not yet built:** D2 (regional severity heatmap, needs a regional ILINet pull into `data/raw/`), the CDC-anchored severity tier definition, and the RF severity classifier that depends on those tiers. None of this is started. Do not describe any of it as done.

## Open decisions (still not resolved)

- The specific decision week(s) W to headline (currently 8/12/16 throughout). Not raised in the 2026-07-21 follow-up, so his 2026-08-02 reply does not touch it. Still Joshua's call or a question for the next advisor update.
