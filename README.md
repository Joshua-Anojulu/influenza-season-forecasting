# Influenza Season Forecasting

Forecasting the timing and severity of US influenza season peaks from public CDC surveillance data.

> **Status:** Analysis pipeline complete (notebooks 01-05). Results below are **preliminary and
> descriptive**: the sample is small (19 modeled seasons) and the core scope decisions are still
> pending the advisor's sign-off (see *Scope decisions pending review*). Nothing here is a final
> claim.

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
scope only. Raw files live in `data/raw/` (gitignored).

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
  used for a calibration analysis. A Random Forest severity classifier is **planned but not yet
  implemented**.
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
- **Peak severity is more tractable, but these off-the-shelf models do not exploit it.** The
  climatology severity floor is MAE 1.34 (`% WEIGHTED ILI`), and the within-season running max sharpens
  to MAE 0.84 once early January is observed (W=16). ARIMA/Prophet only appear to "beat" the floor at
  W=8, where the baseline is degenerate; at the realistic W=12 and W=16 they lose to the within-season
  floor, overshooting milder seasons by extrapolating toward the historical-max ceiling.
- **Primary affirmative finding (calibration).** Prophet's 80% prediction intervals are pinned near
  the historical-max ceiling (~7-7.5 ILI) while most seasons peak 1.5-4 points below them, so empirical
  LOSO coverage is only **~6-12% (1-2 of ~17 seasons) versus the nominal 80%**, about an order of
  magnitude too low: severely overconfident, by the same upward-extrapolation pathology as the point
  forecasts. (Per-W coverage is granular at this sample size; W=12 tips between 5.9% and 11.8% under
  Stan optimizer convergence, so the fit is seeded and the result is quoted as a range.) This stands
  independently of point-forecast skill.

These are honest negative point-forecast results plus a calibration contribution, not a claim that
peak forecasting is solved or impossible. Capturing the epidemic turnover would require model
structure (a curve or mechanistic model) that ARIMA/Prophet lack; that is future work.

## Repository structure

```
.
├── CLAUDE.md             # working agreement + project memory (read first)
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── 01_data_inventory.ipynb   # load, audit, document source quirks and decisions
│   ├── 02_cleaning.ipynb         # season alignment, smoothed targets, strain stitch
│   ├── 03_eda.ipynb              # trajectories, distributions, missingness map
│   ├── 04_baselines.ipynb        # naive floors, lead-time-matched
│   └── 05_forecasting.ipynb      # ARIMA / Prophet under the leakage firewall
├── data/raw/             # gitignored; CDC source CSVs
├── figures/              # generated EDA + calibration figures
├── results/              # baseline + forecasting summaries (markdown + JSON)
└── src/                  # shared utilities (currently empty)
```

Notebooks are committed without execution outputs; each reconstructs the cleaned data from 02's
committed logic and re-runs deterministically (Prophet's interval sampling is the one stochastic
element). `AGENTS.md`, if present, is an auto-generated duplicate of `CLAUDE.md` and is gitignored.

## Scope decisions pending review

The following are working assumptions awaiting the advisor's sign-off and are marked as such wherever
they appear in code and docs:

- Feature strategy (Option B: core features 2003+, enrichment 2009+) versus an all-feature 2009+ design.
- The within-season-forecasting framing and the specific decision weeks W.
- Whether the template's target metrics (e.g. peak week within +/-1 on >=70% of seasons) are goals
  rather than deliverables. The 04/05 results suggest the +/-1-on-70% timing target is likely
  unreachable honestly at these lead times.
- National-only scope versus HHS-regional.

## Notes and limitations

The training signal is bounded by sample size: 22 complete seasons, 19 after pandemic exclusions.
Pandemic seasons introduce structural breaks and are held out as labeled special cases. These
constraints limit how strong any honest result can be, and are addressed explicitly in the analysis
rather than worked around. Honest negative results are reported as such.

## Reproducing

```
pip install -r requirements.txt
# place the CDC source CSVs in data/raw/ (see notebook 01 for filenames and loading quirks)
# run notebooks 01 -> 05 in order
```

## Background

- Reich NG et al. (2019). A collaborative multiyear, multimodel assessment of seasonal influenza forecasting in the United States. *PNAS* 116(8), 3146-3154.
- Biggerstaff M et al. (2018). Results from the second year of a collaborative effort to forecast influenza seasons in the United States. *Epidemics* 24, 26-33.
- Viboud C & Vespignani A. (2019). The future of influenza forecasts. *PNAS* 116(8), 2802-2804.

## Acknowledgment

Research project advised by Dr. Subhro Mitra.
