# Plan: Close the template-adherence gaps that do not require advisor input
_Locked via grill — by Claude + Joshua. Revised after Codex review round 1._

## Goal

An audit of this repo against `Flu_Research_Project_Template.pdf` found that zero of the four
required deliverables (template section 6) exist in finished form, even though notebooks 01-06 are
methodologically stronger than the template asks for. This plan closes every gap that does not need
Dr. Mitra's ruling: it builds D1 (forecast chart) and D3 (feature importance plot), adds the three
template features that were never implemented (`ili_pct_lag1..lag4`, `ili_pct_rolling4`,
`hosp_rate_lag1`), tests hypothesis H1 for the first time, and scores the three excluded pandemic
seasons as out-of-sample special test cases to answer secondary research question 2. D2 (severity
heatmap by HHS region) and the severity tier definition it depends on are deliberately excluded and
are the subject of a separate email to the advisor.

**Reproducibility guarantee, stated precisely.** No pre-existing value in any protected artifact may
change. Exactly one additive schema expansion is whitelisted: a new `ridge_coefficients` key in
`results/06_regression_curve_summary.json` (step 2.1). Every other protected artifact must be
bit-identical, verified by hash against `git HEAD`, not by assertion.

**Protected artifact list** (hash before and after, stop on any diff). These are the DATA artifacts
only. Notebooks 05 and 06 are intentionally edited by this plan and are NOT in the no-diff gate;
their diffs are reviewed separately as expected implementation changes.
```
results/04_baseline_summary.json      results/04_baseline_summary.md
results/05_audit.json                 results/05_forecasting_summary.json
results/05_forecasting_summary.md     results/05_survivorship.json
results/05_survivorship.md            results/06_regression_curve_summary.md
results/06_regression_curve_summary.json   (all pre-existing keys only; ridge_coefficients added)
figures/01a_trajectory_overlay.png    figures/01b_trajectory_small_multiples.png
figures/02_holiday_correction_qc.png  figures/03_peak_week_distribution.png
figures/04_peak_ili_distribution.png  figures/05_strain_timeline.png
figures/06_peakili_by_strain.png      figures/07_missingness_map.png
figures/08_prophet_calibration.png    figures/09_severity_by_W.png
```
Generate the protected set from `git ls-files results figures` at runtime rather than typing a
shorthand like `figures/01a..09` (which risks silently missing `01b`). Verify every protected file by
SHA-256 against `git HEAD`, with ONE exception: `results/06_regression_curve_summary.json` changes by
design (the `ridge_coefficients` key is added), so it is checked by key-by-key comparison of all
pre-existing keys instead of a whole-file hash. Do NOT gate on notebooks and do NOT use a whole-tree
diff: the notebooks change by design, and `PLAN.md` / `PLAN-REVIEW-LOG.md` are untracked and would
pollute a tree diff.

## Approach

### Step 1 — Notebook 05: persist trajectories and score the pandemic seasons

**1.1 Trajectories must NOT flow through `res`.** Cell 18 does `aud = res.copy()` and then
`aud.to_json(RESULTS_DIR / "05_audit.json")`, so any column added to `run_one`'s returned record
lands in the committed audit artifact and breaks the guarantee. Instead, have `run_one` append to a
module-level sidecar list `TRAJ` keyed by `(model, season, W)`, holding `h_sw`, `traj`, and the
Prophet `lo` / `hi` arrays. The record `run_one` returns is unchanged, field for field. `res` and
therefore `05_audit.json` are untouched by construction, not by a later drop step.

1.2 Write the sidecar to a NEW artifact `results/05_trajectories.json`. Rationale beyond the
guarantee: `05_audit.json` is already 49 KB of peak-level records and exists to document the
firewall; inlining ~30 weeks of trajectory per fit would obscure that purpose.

**Idempotency guard.** Initialize `TRAJ = []` in the same cell as, and immediately before, the main
forecast loop, so a cell re-run without a kernel restart cannot append duplicate records to a stale
list. Before writing the artifact, assert that `(model, season, W, split)` keys are unique and that
the record count equals the expected `2 models * 3 W * (19 + 3) seasons` minus any documented skips.

1.3 Add a SEPARATE loop scoring the three excluded seasons (`2008-09`, `2009-10`, `2020-21`) with
both ARIMA and Prophet at W in {8, 12, 16}. These seasons are never in any training set. Because the
target season is not a member of `ev`, the LOSO cap exclusion is inapplicable: use
`cap = ev["peak_ili_pct"].max()` over all 19 modeled seasons.

**This is a structural-break stress test, not a prospective forecast, and must be labeled as such
everywhere it appears.** The cap and the training set are drawn from all 19 modeled seasons,
including seasons chronologically later than 2008-09 and 2009-10. A real-time forecaster in December
2008 had none of that information. The result characterizes how badly these models break on a
structural break; it does not estimate what the model would have predicted at the time.

1.4 Report per-season error for each excluded season individually. Do NOT pool them into a mean.
Three seasons with different failure mechanisms (2008-09 spring H1N1 emergence wave, 2009-10 the
pandemic itself, 2020-21 near-total flu absence under COVID NPIs) have no meaningful average.
Artifacts: `results/05_special_cases.json` / `.md`. These records never enter `res`, the 19-season
metrics, the skill table, or the calibration table.

1.5 Build D1 as `figures/10_D1_forecast_overlay.png`. Prophet only, W=12. Six panels selected by a
stated rule fixed before looking at any forecast: the mildest, the median, and the most severe of the
19 modeled seasons by `peak_ili_pct`, plus all three excluded seasons rendered with visually distinct
panel styling and an explicit "excluded from training, structural-break stress test" label. Each
panel shows observed ILI through W=12, the actual continuation, the Prophet mean forecast, the 80%
interval as a shaded band, and the true smoothed peak marked. Caption states that ARIMA produces no
prediction intervals in this implementation, which is why D1 is a Prophet figure.

1.6 Re-run 05 end to end and gate on the protected-artifact hash comparison defined in the Goal.
Prophet is seeded (`SEED=42`) and `requirements.txt` is pinned, which makes bit-identical
reproduction *expected*, but it is not guaranteed: Stan optimizer and serialization behaviour can
drift across environments. Treat the hash gate as the authority. If any protected artifact differs,
STOP and report the diff rather than accepting the new numbers.

### Step 2 — Notebook 06: persist the ridge coefficients

2.1 The standardized ridge coefficients are computed in cell 11 and only printed. Persist them into
`results/06_regression_curve_summary.json` under a new top-level key `ridge_coefficients`, as a list
of `{W, cum_ili, "A(H1N1)", "A(H3N2)", "B"}` records. This is the one whitelisted schema expansion.

2.2 Every pre-existing key and value in that JSON must be unchanged, compared key-by-key rather than
by whole-file hash (the file hash necessarily changes). `results/06_regression_curve_summary.md` is
fully protected and must be bit-identical.

### Step 3 — New notebook 07: template features, H1, and D3

3.1 Reconstruct `weekly` / `season_table` from 02's logic exactly as 05 and 06 do (the known,
deliberately deferred duplication debt; this plan does not attempt the `src/` refactor).

3.2 Define the new features, all indexed strictly at or before W:
- `ili_lag_k` for k in 1..4: `% WEIGHTED ILI` at season-week `W - (k-1)`. So `ili_lag_1` is the most
  recent observed week, at exactly W.
- `ili_rolling4`: mean of `ili_lag_1..ili_lag_4`.
- `hosp_rate_lag1`: FluSurv-NET weekly overall rate at the most recent season-week <= W.

**Index availability and reporting availability are different things.** The `<= W` rule guarantees
no value is drawn from a week after the decision week. It does NOT guarantee the value was published
by week W. ILI is revised but broadly available in near-real-time; FluSurv-NET is reporting-lagged,
as are NREVSS strain and FluVaxView coverage. Therefore any model containing `hosp_rate_lag1`, `vax`,
or strain is retrospective/explanatory, never a real-time forecast. This is the same caveat notebook
06 already applies, extended to hospitalization.

3.3 Two panels, matching the `S2009` precedent already in notebook 06. Column counts stated honestly
because they, not the conceptual count, drive variance:
- **Panel A (19 seasons, 2003+):** `cum_ili`, `ili_lag_1..4`, `ili_rolling4`, three strain one-hots.
  Seven conceptual features, **9 model columns, n=19**.
- **Panel B (14 seasons, 2009+):** Panel A plus `vax` and `hosp_rate_lag1`. Nine conceptual features,
  **11 model columns, n=14**.

**Panel B is high-variance descriptive analysis, not evidence of feature importance.** Eleven columns
against fourteen seasons cannot support a ranking claim. It is reported to satisfy the template's
feature coverage and to show the direction of the estimates, and it must be captioned as such.
Panel A at 9 columns against 19 seasons is also a strained ratio and carries the same caveat in
weaker form.

**Both panels are marked "Option B assumption, pending advisor decision."** `CLAUDE.md` lists Option
B (core 2003+, enrichment 2009+) versus Option A (all-feature, 2009+ only) as an open decision. The
two-panel structure presupposes Option B, so every result it produces is provisional on that call.

3.4 Reuse 06's `ridge_fit` / `ridge_pred` / `ridge_loso` structure (nested lambda CV, LOSO), but with
a **corrected imputation path**, because 06's is insufficient for Panel B in two ways that would
otherwise be inherited:
- **Column-wise, not last-column-only.** 06 imputes exactly one trailing column (`Xtr[:, -1]`, the
  `vax` feature). Panel B has two enrichment columns that can be missing (`vax` and `hosp_rate_lag1`),
  so 07 must impute every NaN-containing column by its own train-fold mean, not just the last.
- **Imputed inside each inner CV split, not once at the outer fold.** 06 fills NaNs once using the
  full outer-training fold before the inner lambda-selection splits run, which leaks each inner
  validation row's fold-mates into its own imputed value and biases lambda choice. In 07, compute
  each imputation mean from the inner-training rows only and apply it to the inner-validation row.
  06 is committed and protected, so this plan does NOT change 06; 07 diverges deliberately and the
  divergence is noted in 07 and in the docs (06's simpler path affects only lambda selection on a
  single feature, a minor effect, which is why 06 is left as-is rather than reopened).

3.5 Test H1 for the first time. H1 states that ILI% from the 4 weeks prior to prediction is the
strongest predictor of peak severity. Operationalize as a head-to-head LOSO comparison at each W:
a lags-only model (`ili_lag_1..4` + `ili_rolling4`) versus the incumbent `cum_ili`-only univariate
model, against the climatology floor (1.341) and the lead-time-matched baseline C.

**Report paired per-season error deltas with a sign test and a bootstrap CI, not a bare MAE
comparison.** At n=19 across three W values, a difference in mean absolute error cannot establish
"strongest predictor," and comparing at three W values invites a multiple-comparison error. Phrase
the conclusion as descriptive support or no support for H1, never as confirmation. Note in the
writeup that `cum_ili` is NOT a template feature, so this is also a test of whether the project's
substitution was justified.

3.6 Build D3 as `figures/11_D3_feature_importance.png`, two subplots (Panel A, Panel B). Report
standardized ridge coefficients alongside **grouped block ablations**, not drop-one-feature deltas.

Rationale: `ili_rolling4` is exactly the arithmetic mean of `ili_lag_1..4`, an exact linear
dependence. Under that dependence neither individual coefficients nor single-feature drop deltas are
interpretable, because dropping one lag leaves its information recoverable from the other four
columns. Ablate whole blocks instead: the recent-ILI lag block, the rolling summary, `cum_ili`, the
strain block, and in Panel B `vax` and `hosp_rate_lag1`. That yields five ablation groups in Panel A
and seven in Panel B, satisfying the template's five-feature comparison with a measure that is
actually valid. Include the interpretive commentary the template requires, and state the collinearity
explicitly in the caption.

3.7 Score the three excluded seasons out-of-sample under the Panel A model as well, carrying the same
structural-break-stress-test labeling as 1.3, so the pandemic special-case analysis covers the
regression track and not only ARIMA/Prophet.

3.8 Firewall audit in 07 mirroring 06's assertion style: assert every feature index is <= W for every
(season, W) pair, and assert the lag features contain no value drawn from a week after W. Label this
in the notebook as an **index** firewall, distinct from the **reporting-availability** caveat in 3.2,
so the assertion is not mistaken for a real-time-availability guarantee.

3.9 Artifacts: `results/07_features_summary.json` and `.md`.

### Step 4 — Documentation

4.1 Update `README.md` and `CLAUDE.md` to reflect what now exists, in the honest register the working
agreement requires: describe what was built, not what it might show.

4.2 Record the two template deviations, in the notebook and later in the paper's Limitations section:
- `season_week` (template section 3.1) **is not usable as a season-level predictor at a fixed decision
  week**, because it equals W for every season in the slice and has zero variance. It is retained
  throughout the project as the within-season time axis (`sw`), which is how 05's trajectory models
  and every figure already use it. The deviation is narrow and specific, not a wholesale exclusion.
- ARIMA produces no prediction intervals in this implementation, so the template's interval-coverage
  metric is reported for Prophet only.

## Key decisions and tradeoffs

**Severity tiers deferred.** Defining Low/Moderate/High/Severe is a target definition, gated on
Joshua's review by working-agreement checkpoint 2. Its only committed consumer, D2, is blocked on
Dr. Mitra's national-versus-regional ruling, and the other consumer (the RF classifier) is an
explicitly optional stretch goal. Hardening a target definition with no live consumer is premature.

**Two feature panels rather than one.** `hosp_rate_lag1` and `vax` both start in 2009-10, capping at
14 seasons, while lags, rolling4, strain and `cum_ili` run on all 19. A single 14-season model would
discard 5 seasons of the only signal that has ever beaten a floor, at n=19. Rejected alternatives:
single 14-season model (sample loss); 19-season only (leaves FluSurv-NET unused a second time, and
the template names `hosp_rate_lag1` explicitly).

**Pandemic seasons scored pure out-of-sample, never folded into training,** and labeled a
structural-break stress test rather than a prospective forecast. Rejected alternative: a 22-season
leave-them-in sensitivity, which would answer "how much do the headline metrics degrade" directly but
costs an extra run of every model and risks the 22-season figures being misread as the headline.
Also rejected as scope creep: a chronologically-restricted refit for each special case, which would
make them genuinely prospective but requires rebuilding the training set per season.

**05 extended rather than duplicated,** with trajectories carried in a sidecar so `res` and the audit
artifact are structurally incapable of changing. Rejected alternative: the `src/` refactor, correct
long-term but touching four committed verified notebooks. That debt stays deferred and recorded.

**D1 season selection is rule-based and stated in advance.** Given this project's own survivorship
analysis of ARIMA's W=16 result, a hand-picked set of display seasons would be an obvious and fair
criticism. The rule is fixed before any forecast is inspected.

**D3 uses grouped block ablations rather than per-feature importance.** The exact linear dependence
between `ili_rolling4` and the four lags makes any per-feature ranking invalid, whether derived from
coefficients or from drop-one deltas. Reporting one anyway would be the kind of inflated claim the
working agreement forbids.

## Risks and open questions

**Feature count versus sample size is the dominant risk.** 9 columns against n=19 in Panel A, 11
against n=14 in Panel B. Ridge with nested lambda CV under LOSO is the mitigation, but the ratio is
itself a limitation and is stated as one. There is a real chance both panels underperform the
univariate `cum_ili` model, which would be a legitimate reportable negative consistent with 06's
existing finding that strain and vaccine coverage add nothing.

**Re-run reproducibility is gated, not assumed.** Steps 1 and 2 stop on any protected-artifact hash
diff. Seeding makes success expected but does not guarantee it.

**2020-21 will produce a large error, and that is the point.** Flu was near-absent under COVID NPIs.
Models extrapolating toward the historical-max ceiling will overshoot enormously. Present this as a
characterization of the structural break, not as either a model failure or a validation of the
exclusion decision.

**FluSurv-NET is entirely missing for 2020-21** (all 31 missing weekly rates fall in that season, per
notebook 03). If 2020-21 is scored out-of-sample under Panel B, `hosp_rate_lag1` is undefined and the
train-mean imputation path fires. Flag this explicitly wherever that number appears rather than
letting an imputed feature pass silently.

**Still open, and NOT resolved by this plan** (both pre-existing open decisions in the README):
which decision week W to headline (this plan uses 8/12/16 throughout, W=12 for D1), and Option A
versus Option B (this plan's two-panel structure presupposes Option B and every result it produces is
marked provisional on that call).

## Out of scope

- **D2, the season severity heatmap by HHS region.** Blocked on Dr. Mitra's ruling. No region-resolved
  ILI exists anywhere in the pipeline.
- **The severity tier definition.** Deferred with D2.
- **The Random Forest severity classifier** and its macro-F1 metric. Optional stretch goal, dependent
  on the deferred tier definition.
- **D4, the research paper.** Sequenced after these figures exist, since `results/` and the new
  figures are its raw material.
- **The `src/` refactor** of 02's duplicated reconstruction logic. Known debt, deliberately deferred.
- **Adding prediction intervals to ARIMA.** Scope creep into reviewed model code that would change
  05's committed outputs.
- **Chronologically-restricted refits** for the pandemic special cases. Would make them prospective;
  rejected as scope creep, with the limitation stated instead.
- **Any git commit or push.** Working-agreement checkpoints 2 and 4: model code and target definitions
  are reviewed by Joshua before they land, and nothing is pushed without his explicit instruction.
