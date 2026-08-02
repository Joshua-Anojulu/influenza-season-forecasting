# Influenza Season Forecasting

Forecasting the timing and severity of US influenza season peaks from public CDC surveillance data.

> **Status:** Analysis pipeline executed through notebook 08. Results below are **preliminary and
> descriptive**: the sample is small (19 modeled seasons). Notebooks 01-07 and their artifacts are
> committed and pushed; notebook 08 and its `results/08_*` artifacts are committed locally and not
> yet pushed. The advisor confirmed the characterization-first framing, national-only scope, and
> regression/curve models on 2026-07-08, and confirmed Option B (2003+ core, 2009+ enrichment), the
> CDC-anchored severity tiers, and the regional-ILI-for-the-heatmap-only scope on 2026-08-02. The
> decision week to headline remains the one open decision. Nothing here is a final claim.

## Overview

This project investigates whether historical US influenza surveillance data can forecast two
characteristics of a flu season:

1. **Peak timing:** the MMWR week at which influenza-like illness (ILI) activity peaks.
2. **Peak severity:** the `% WEIGHTED ILI` value reached at that peak.

Both are framed as **within-season forecasting** tasks. Standing at a fixed decision week W of an
ongoing season, and using only data available through W, a model forecasts the remaining trajectory,
from which the predicted peak week and peak height are read off. Repeating this at different decision
weeks yields an accuracy-versus-lead-time analysis. Validation is leave-one-season-out (LOSO).

## Data sources

All data is publicly available from the CDC. This project is not affiliated with the CDC. National
scope for every model; regional (HHS Regions) ILI is used only as an input to the D2 figure. Raw files live in `data/raw/` (gitignored).

| Source | Contents | Role | Coverage |
|--------|----------|------|----------|
| CDC FluView / ILINet | Outpatient ILI (`% WEIGHTED ILI`) | Core target | 2003-04 to 2024-25, weekly |
| CDC NREVSS (Combined + Public Health Labs + Clinical Labs) | Virological strain composition | Core feature (`dominant_strain`) | 2003+, weekly |
| CDC FluSurv-NET | Lab-confirmed influenza hospitalization rates | Enrichment | 2009-10+, weekly |
| CDC FluVaxView | Seasonal vaccine coverage (national, all ages) | Enrichment | 2009-10+, monthly cumulative |
| WHO FluNet | Virological / strain composition | **Excluded** (2022+ only, global, redundant with NREVSS) | not used |

Strain composition comes from **NREVSS**, not WHO FluNet. NREVSS subtype reporting changed in
2015-16, so the continuous `dominant_strain` series is stitched from the Combined file (through
2014-15) and the Public Health Labs file (2015-16 on); the Clinical Labs file is positivity-only and
is not used for subtype.

## Methodology

- **Cleaning (02):** align ILINet to MMWR seasons (week 40 -> week 39); define the targets
  `peak_week` and `peak_ili_pct` on a **3-week centered smoothed** ILI series to remove a year-end
  (week 52) holiday reporting artifact; retain raw values and three audit flags (`holiday_shift`,
  `peak_week_smoothing_sensitive`, `fragile_peak_week`); stitch the NREVSS `dominant_strain` series.
- **EDA (03):** season-trajectory overlays and small multiples, peak-week / peak-ILI distributions,
  dominant-strain timeline, and a cross-source missingness map.
- **Baselines (04):** climatology (zero-information floor), persistence (prior season), within-season
  running-max at decision weeks W in {8, 12, 16}, and an exploratory strain-climatology. Every later
  model must beat the **lead-time-matched** floor (a model at W is compared to the within-season
  baseline at the same W, never to a cross-sectional floor).
- **Forecasting (05):** ARIMA and Prophet, fit within-season on data through W only (strict leakage
  firewall, audited per fit), forecasting the remaining weeks. Prophet's 80% prediction interval is
  used for a calibration analysis. Per-fit trajectories are persisted in a sidecar that does not
  alter the protected audit records. The three excluded seasons are scored separately as labeled
  structural-break stress tests, not prospective forecasts, and D1 shows six Prophet overlays at W=12.
- **Regression and curve models (06):** three models under the identical firewall, all features
  computed strictly through W. (1) A univariate real-time severity forecast from cumulative-ILI-
  through-W. (2) A retrospective explanatory ridge (cumulative ILI + dominant strain + vaccine
  coverage) testing whether strain/vaccine carry severity signal; labeled explanatory because strain
  is reporting-lagged and vaccine coverage is a revised survey estimate. (3) A symmetric Gaussian
  curve fit for both peak height and peak week. The standardized ridge coefficients are persisted
  in the summary JSON. (The Random Forest severity classifier moved to notebook 08 and is now built.)
- **Template features and H1 (07):** adds `ili_lag_1..4`, `ili_rolling4`, and
  `hosp_rate_lag1`; tests H1 with paired per-season error deltas, exact sign tests, and bootstrap
  intervals; and builds D3 from grouped block ablations plus directional standardized coefficients.
  Panel A has 9 model columns on 19 seasons. Panel B has 11 columns on 14 seasons and is explicitly
  high-variance descriptive analysis. Both panels rest on **Option B, which the advisor confirmed on
  2026-08-02** (2003+ core with 2009+ enrichment where it exists), so they are no longer provisional.
  Hospitalization, strain, and vaccine models are retrospective because an index cutoff does not
  establish reporting availability.
- **Severity tiers and classification (08):** defines season severity tiers anchored to CDC's
  published ILI intensity thresholds (IT50 4.4, IT90 6.6, IT98 8.6; Biggerstaff et al., Am J
  Epidemiol 2018, doi:10.1093/aje/kwx334), applied to the season's peak weekly raw `% WEIGHTED ILI`.
  This is an **ILI-only approximation** of CDC's framework, not CDC's official season classification,
  which is a 2-of-3 indicator vote requiring hospitalization and mortality data this project does not
  have. It reproduces CDC's published classifications on **9 of 12** seasons overall and 9 of 10
  among comparable seasons. No season in 22 reaches IT98, so the classifier is 3-class. The Random
  Forest classifier is a **negative result**: it shows no demonstrated advantage over thresholding
  the univariate regression, with paired bootstrap intervals including zero at W=12 and W=16. D2, the
  regional severity heatmap, is a **continuous** map of regional season peak ILI with the national
  thresholds shown only as colorbar reference ticks; regions are never assigned tiers, because CDC
  publishes no regional thresholds and regional baselines differ sharply (Region 6's median season
  peak of 9.455 sits above the national IT98, Region 1's 3.896 below the national IT50).
- **Validation:** LOSO over the same season set throughout. Of 22 complete seasons (2003-04 to
  2024-25), **19 are modeled** after holding out 2009-10 and 2020-21 (pandemic) and 2008-09
  (pandemic-adjacent, the 2009 H1N1 emergence) as labeled special cases.
- **Metrics:** peak-week MAE (weeks) and % within +/-1; peak-ILI MAE and RMSE; 80% interval coverage
  (calibration).

## Results (preliminary)

Naive floors (04, LOSO over 19 seasons) and within-season models (05, leakage firewall audited clean
across all 114 fits: no post-W data entered any fit, the predicted peak is read only from the
forecast region, and no suspected leak). Full tables: `results/`.

- **Peak-week timing has no strong naive floor and the models do not beat it.** Every baseline lands
  at roughly 3.3-3.8 weeks MAE with at most ~37% of seasons within +/-1 week (climatology: 3.68 weeks
  MAE, 26% within +/-1). Under the firewall, ARIMA and Prophet do not improve on this at realistic
  lead times; Prophet's trend extrapolates monotonically to the forecast horizon, so its predicted
  peak lands at the season end (0% within +/-1). Peak timing appears close to the noise floor at these
  lead times on this sample.
  - **One apparent exception, and why it does not survive scrutiny.** ARIMA at W=16 posts
    `pw_skill = +3.00` against baseline C (MAE 3.00 vs 6.00 weeks) in `results/05_forecasting_summary.md`.
    Peak-week metrics are computed only on non-plateau forecasts, and that exclusion is not neutral:
    ARIMA locates a peak in just 7 of 12 forecast seasons, and the 5 it drops are the severe ones
    (mean peak ILI 5.75 versus 4.29 for the 7 it keeps). Baseline C is also a weak *timing* rule,
    since at W=16 many seasons have not yet peaked. Measured against LOSO climatology on the same 7
    seasons (3.29 weeks), ARIMA's edge is **0.29 weeks at n=7**, with 1 of 7 within +/-1, below the
    36.8% floor. Full accounting in `results/05_survivorship.md`.
- **Peak severity is more tractable, but these off-the-shelf models do not exploit it.** The
  climatology severity floor is MAE 1.34 (`% WEIGHTED ILI`), and the within-season running max sharpens
  to MAE 0.84 once early January is observed (W=16). ARIMA/Prophet only appear to "beat" the floor at
  W=8, where the baseline is degenerate; at the realistic W=12 and W=16 they lose to the within-season
  floor, overshooting milder seasons by extrapolating toward the historical-max ceiling.
- **Primary affirmative finding (calibration).** Prophet's 80% prediction intervals are pinned near
  the historical-max ceiling (the median interval at W=12 is [7.09, 7.54] ILI) while most seasons peak
  well below them: at W=12, **14 of 17 forecast seasons fall under the interval** (2 above, 1 inside),
  a median 2.20 ILI points beneath its lower bound. Empirical LOSO coverage is only **5.9-10.5% across
  W (1-2 of 12-19 seasons) versus the nominal 80%**, about an order of magnitude too low: severely
  overconfident, by the same upward-extrapolation pathology as the point forecasts. (Coverage is
  granular at this sample size, where one season is worth ~6 percentage points; an earlier *unseeded* run
  read 11.8% at W=12 because two seasons sit within 0.04 ILI of an interval edge. The fit is now seeded,
  so 5.9% is reproducible, and the order-of-magnitude undercoverage holds either way.) Per-season
  split in `results/05_survivorship.md`. This stands independently of point-forecast skill.
- **A simple regression is the first model to honestly beat a floor (06).** Predicting peak severity
  from cumulative-ILI-through-W gives MAE 1.26 / 1.12 / 0.92 at W=8 / 12 / 16, beating climatology
  (1.34) at every lead time and beating or tying the within-season running-max floor at W=8 and W=12.
  Adding dominant strain and vaccine coverage (a retrospective explanatory ridge) does **not** lower
  the error below this ILI-only model, so those covariates carry no extra severity signal at this
  sample size. A symmetric Gaussian curve fit does not beat the floor on either target: fit to a
  usually rising-limb segment its amplitude pins to the historical-max ceiling, the same overshoot
  pathology as ARIMA/Prophet.
- **H1 is not descriptively supported (07).** The lags-only model has MAE 1.125 / 1.189 / 1.067 at
  W=8 / 12 / 16 versus 1.256 / 1.124 / 0.917 for cumulative ILI. It improves at W=8 but loses at
  W=12 and W=16. All three paired bootstrap intervals include zero, and the exact sign tests split
  10-9 or 9-10 by season. This does not establish the template's claim that recent four-week ILI is
  the strongest predictor.
- **The two feature panels are descriptive, not rankings (07).** Panel A MAE is 1.195 / 1.075 / 1.175
  at W=8 / 12 / 16. Panel B MAE is 1.268 / 0.906 / 0.577 on only 14 seasons and 11 columns, so its
  lower W=12 and W=16 errors cannot support a feature-importance claim. D3 uses grouped block
  ablations because `ili_rolling4` is exactly determined by the four lag columns. The excluded
  seasons are reported separately under Panel A as structural-break stress tests, never pooled.

These are honest negative point-forecast results plus two affirmative contributions (the calibration
finding and the univariate severity forecast), not a claim that peak forecasting is solved or
impossible. A phenomenological curve model (06) was tried and did not rescue the peak, because at
realistic lead times the peak has not yet happened; capturing the turnover would require a mechanistic
model, which is future work.

## Repository structure

```
.
├── CLAUDE.md             # working agreement + project memory (read first)
├── README.md
├── START_HERE.md         # student-facing getting-started guide
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── 01_data_inventory.ipynb   # load, audit, document source quirks and decisions
│   ├── 02_cleaning.ipynb         # season alignment, smoothed targets, strain stitch
│   ├── 03_eda.ipynb              # trajectories, distributions, missingness map
│   ├── 04_baselines.ipynb        # naive floors, lead-time-matched
│   ├── 05_forecasting.ipynb      # ARIMA / Prophet under the leakage firewall
│   ├── 06_regression_and_curve.ipynb  # univariate + explanatory ridge + Gaussian curve
│   ├── 07_features_and_hypothesis.ipynb  # template features + H1 + grouped ablations
│   └── 08_severity_tiers.ipynb           # CDC-anchored severity tiers + 3-class classifier
├── data/raw/             # gitignored; CDC source CSVs
├── figures/              # generated EDA, calibration, D1, and D3 figures
├── results/              # baseline + forecasting summaries (markdown + JSON)
├── slides/               # findings deck (PowerPoint / Google Slides)
├── docs/                 # design spec + implementation plan
└── src/                  # shared utilities (currently empty)
```

Notebooks are committed without execution outputs; each reconstructs the cleaned data from 02's
committed logic and re-runs deterministically (Prophet's interval sampling is the one stochastic
element). `AGENTS.md`, if present, is an auto-generated duplicate of `CLAUDE.md` and is gitignored.

## Scope decisions

Confirmed by the advisor (2026-07-08): the characterization-first framing and the within-season
formulation; national-only scope for now (one or two HHS regions later as a robustness check); and the
template's target metrics (e.g. peak week within +/-1 on >=70% of seasons) as goals, not deliverables
(the 04/05 results indicate the +/-1-on-70% timing target is likely unreachable honestly at these lead
times).

Still open:

- Feature strategy (Option B: core features 2003+, enrichment 2009+) versus an all-feature 2009+ design.
- Which decision week(s) W to headline (currently 8, 12, 16 throughout).

## Notes and limitations

The training signal is bounded by sample size: 22 complete seasons, 19 after pandemic exclusions.
Pandemic seasons introduce structural breaks and are held out as labeled special cases. These
constraints limit how strong any honest result can be, and are addressed explicitly in the analysis
rather than worked around. Honest negative results are reported as such.

Two template deviations are explicit. `season_week` is not usable as a season-level predictor at a
fixed W because it equals W for every season and has zero variance; it remains the within-season time
axis `sw`. ARIMA produces no prediction intervals in this implementation, so D1 and interval coverage
are Prophet-only.

## Reproducing

```
pip install -r requirements.txt
# place the CDC source CSVs in data/raw/ (see notebook 01 for filenames and loading quirks)
# run notebooks 01 -> 07 in order
```

## Background

- Reich NG et al. (2019). A collaborative multiyear, multimodel assessment of seasonal influenza forecasting in the United States. *PNAS* 116(8), 3146-3154.
- Biggerstaff M et al. (2018). Results from the second year of a collaborative effort to forecast influenza seasons in the United States. *Epidemics* 24, 26-33.
- Viboud C & Vespignani A. (2019). The future of influenza forecasts. *PNAS* 116(8), 2802-2804.

## Acknowledgment

Research project advised by Dr. Subhro Mitra.
