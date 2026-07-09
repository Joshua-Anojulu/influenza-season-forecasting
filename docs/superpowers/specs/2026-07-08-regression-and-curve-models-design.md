# 06 — Regression and phenomenological curve models

**Status:** approved design, not yet implemented.
**Date:** 2026-07-08.
**Author:** Joshua (with Claude as advisor).
**Origin:** Dr. Subhro Mitra's feedback at the last stopping point, evaluated and adapted below.

## 1. Context

The pipeline is complete through `05_forecasting.ipynb`. Established, honest findings:

- Peak-**timing** is near the naive noise floor at realistic decision weeks (baselines ~3.3–3.8 wk MAE, ≤37% within ±1). ARIMA and Prophet do not beat that floor; their within-season extrapolation places the peak at the horizon boundary (Prophet) or produces a cap-clipped plateau with no locatable peak week (ARIMA, now flagged `peak_ambiguous`).
- Peak-**severity** is the more tractable target (within-season running-max floor reaches MAE 0.84 by W=16), but the off-the-shelf models do not exploit it.
- Prophet's 80% intervals are pinned near the historical-max cap while real peaks fall well below, so empirical LOSO coverage is ~6–11% versus a nominal 80%: severe overconfidence.

This notebook adds two model families better matched to the problem, under the identical firewall, and keeps the project's **characterization-first** posture: the deliverable is a clear description of what this data can and cannot support, not a marginally better RMSE.

## 2. Framing (decided)

Lead with the precise, two-pronged result, not the looser "severity is harder than the literature suggests":

1. Peak timing is at or below the naive floor at realistic lead times.
2. Off-the-shelf models are severely overconfident on severity intervals (~6–11% coverage vs 80%).

Severity is described as tractable-but-unexploited by the off-the-shelf models. Notebook 06 tests whether purpose-built models change that.

## 3. Goals / non-goals

**Goals**
- A leakage-safe real-time severity forecast that is honest about what is available at decision week W.
- A retrospective explanatory model quantifying how dominant strain and vaccine coverage relate to peak severity (addresses Dr. Mitra's request without pretending it is a real-time forecast).
- A phenomenological curve model that respects the rise-and-fall shape and yields a genuine (non-artifact) peak week, our honest attempt at the timing target.

**Non-goals**
- No compartmental / mechanistic epidemic model.
- No HHS-regional analysis yet (national-first; regional is a later robustness check, per Dr. Mitra).
- No tuning for a better headline number at the expense of honesty.

## 4. Evaluation harness (shared with 04/05)

- Reconstruct 02's cleaned data from the raw files (same deterministic pattern as 03/04/05).
- LOSO over the **19 non-pandemic seasons** (exclude 2008-09, 2009-10, 2020-21).
- Decision weeks **W ∈ {8, 12, 16}** on the `sw` axis (wk40=1 … wk39=52).
- **Firewall:** every feature uses only data with `sw ≤ W`. For any model that predicts a peak week, the peak is read only from the forecast region (`sw > W`); seasons whose true peak already occurred by W are reported separately, as in 05.
- Compare each model to **baseline C at the same W** and to 05's ARIMA/Prophet on the matching target(s).
- Reuse 05's metric helpers: peak-week MAE and % within ±1 on the forecast subset (and excluding `fragile_peak_week`), severity MAE/RMSE.
- Results to `results/06_*.{md,json}`; figures to `figures/`. Notebook committed without outputs.

## 5. Models

### 5.1 Univariate regression — real-time severity forecast

- **Predictor:** `cum_ili_thruW` = sum of `% WEIGHTED ILI` over `sw ≤ W` (wk40 through W).
- **Target:** `peak_ili_pct` (02's smoothed severity target).
- **Fit:** cross-sectional LOSO. For each held-out season, fit an ordinary least-squares line on the other 18 seasons' `(cum_ili_thruW, peak_ili_pct)` pairs; predict the held-out season.
- **Honesty:** `cum_ili_thruW` is derived from ILINet, which is the one signal genuinely in hand at W. Labeled the real-time severity forecast.
- **Report:** MAE/RMSE at each W vs baseline C severity and vs climatology.

### 5.2 Explanatory ridge — retrospective, NOT a forecast

- **Predictors (all through-W):** `cum_ili_thruW`, `dominant_strain_thruW` (one-hot of the leading NREVSS subtype from counts cumulated over `sw ≤ W`), `vax_coverage_thruW` (max FluVaxView cumulative coverage among monthly snapshots whose month has completed by W).
- **Target:** `peak_ili_pct`.
- **Fit:** standardize features within each fold; ridge with L2 penalty chosen by inner leave-one-out CV on the training seasons; outer LOSO for evaluation.
- **Label:** explicitly explanatory, not a real-time forecast, because NREVSS strain is reporting-lagged and FluVaxView coverage is a survey estimate revised after the season. The writeup states this plainly.
- **Report:** LOSO MAE/RMSE, standardized coefficients (direction and magnitude of the strain/vax associations), and whether adding strain+vax improves on the univariate model.

### 5.3 Gaussian phenomenological fit — severity and timing

- **Model:** `y(t) = c + A · exp(-(t - μ)² / (2σ²))` fit to raw `% WEIGHTED ILI` against season-week `t`, using only `sw ≤ W`.
- **Fit:** `scipy.optimize.curve_fit` with bounds — `c ∈ [0, min(observed ILI through W)]`, `A ∈ [0, cap]` (cap = LOSO training-max peak, as in 05), `μ ∈ [1, 52]` (season-week), `σ ∈ [1, 20]`. Seed from heuristics: `c`=min observed, `A`=max observed − min, `μ`=season-week of the observed max, `σ`=4.
- **Readout:** severity = `c + A`; peak week = `μ` mapped to the nearest MMWR week and `sw`.
- **Firewall:** if `μ ≤ W` the fit says the peak already passed → treated as "peak already observed at W" (reported separately), mirroring 05. Degenerate fits (parameters at bounds, or a monotone segment with no interior max) are flagged `peak_ambiguous` and excluded from the timing metric, reusing 05's guard.
- **Expectation (stated up front):** at W=8 the observed segment is usually the rising limb only, so μ/A/σ are weakly constrained and the fit is expected to be unstable or bound-pinned; the model should be informative mainly at W=12/16. Report the count of bound-hits and ambiguous fits.
- **Report:** severity MAE/RMSE and peak-week MAE/within±1 vs baseline C and vs 05's models, on both targets.

## 6. Deliverables

1. `notebooks/06_regression_and_curve.ipynb` (committed without outputs).
2. `results/06_regression_curve_summary.{md,json}` and any 06 figures in `figures/`.
3. A ~10–12 minute slide deck to Dr. Mitra's structure: motivation + data (2), holiday-artifact data-quality finding (1), model comparison with honest error metrics (2–3), Prophet calibration finding (1), next steps (1).
4. CLAUDE.md "Open decisions" updated to record what Dr. Mitra confirmed (framing, national-first, the added models).
5. A consolidated Google Drive folder of shareable materials for Dr. Mitra's students to follow and replicate (Joshua shares it manually).

## 7. Risks and mitigations

- **Gaussian instability at short W** — bounded fit + explicit reporting of bound-hits/ambiguous fits; do not hide degenerate fits behind an argmax.
- **Real-time availability of strain/vax** — quarantined into the explanatory model, never the real-time forecast; caveats stated.
- **Small n (19 seasons)** — regularization for the ridge; all claims stay descriptive; report subset sizes (e.g. `n_pw`) wherever a metric is computed on a reduced set.
- **Leakage** — every feature defined strictly on `sw ≤ W`; the firewall audit from 05 is extended to 06's features.

## 8. Checkpoints (per CLAUDE.md)

- Target/feature definitions here (Section 5) are reviewed before modeling code is written.
- Nothing is committed or pushed without Joshua's explicit go-ahead; no data under `data/raw/` is ever tracked.
