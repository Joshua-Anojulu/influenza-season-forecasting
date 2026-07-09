# 06 Regression and Curve Models — Implementation Plan

> **For agentic workers:** Use superpowers:executing-plans or subagent-driven-development to implement task-by-task. Steps use checkbox (`- [ ]`) syntax.
> **Verification idiom (this repo has no pytest suite):** each task's deliverable is verified by (a) in-notebook `assert` statements, and (b) a clean end-to-end run via `nbconvert` with CWD-independent paths, and (c) an eyeball check of printed outputs against stated expectations. This mirrors how 01–05 are verified.

**Goal:** Add a leakage-safe real-time severity forecast, a retrospective explanatory ridge, and a Gaussian phenomenological curve model in `notebooks/06_regression_and_curve.ipynb`, then a slide deck and a consolidated Drive package.

**Architecture:** New notebook 06 reconstructs 02's cleaned data (same deterministic pattern as 03/04/05), builds strictly-through-W features, and evaluates three models under the identical LOSO firewall as 05, comparing to baseline C at the same W and to 05's ARIMA/Prophet.

**Tech Stack:** pandas, numpy, matplotlib, scipy (curve_fit). Ridge is closed-form NumPy — **no scikit-learn dependency added**.

## Global Constraints (verbatim from spec + CLAUDE.md)

- Python 3.14; pandas==3.0.3, numpy==2.4.6, matplotlib==3.10.9, statsmodels==0.14.6, prophet==1.3.0, scipy (already present via statsmodels). No new pinned deps unless approved.
- LOSO over the 19 non-pandemic seasons (exclude 2008-09, 2009-10, 2020-21). Decision weeks W ∈ {8,12,16} on the `sw` axis (wk40=1 … wk39=52).
- Every feature uses ONLY data with `sw ≤ W`. Predicted peak weeks read ONLY from `sw > W`; seasons peaking by W reported separately.
- No em dashes in prose/markdown output. Targets are 02's smoothed `peak_ili_pct` / `peak_week`.
- Notebook committed WITHOUT outputs. No file under `data/raw/` is ever tracked. No commit or push without Joshua's explicit go-ahead.
- Accuracy: nothing described as done that isn't; all claims descriptive; label the explanatory model as NOT a real-time forecast.

---

### Task 1: Notebook scaffold + deterministic data reconstruction

**Files:** Create `notebooks/06_regression_and_curve.ipynb`

**Interfaces produced:** `weekly` (per-week ILINet, complete seasons, with `season, ssy, sw` columns), `season_table` (per-season smoothed `peak_week, peak_ili_pct, sw_true, fragile_peak_week`), `ev` (19 eval seasons), `EVAL`, helpers `season_of/sw/sw_to_week`, `baseline04` (loaded), `bl05` (loaded from `results/05_forecasting_summary.json`).

- [ ] **Step 1:** Markdown title cell describing purpose (three models, firewall, characterization-first), stating the explanatory ridge is NOT a real-time forecast.
- [ ] **Step 2:** Setup cell copied from 05's corrected setup: CWD-independent `DATA_DIR = next((Path(p) for p in ["data/raw","../data/raw"] if Path(p).exists()), Path("data/raw"))`, `RESULTS_DIR`/`FIG_DIR` off `DATA_DIR.parent.parent`, `EXCLUDED`, `DECISION_WEEKS=[8,12,16]`, `SEED=42`, `season_of/sw/sw_to_week`.
- [ ] **Step 3:** Reconstruction cell identical in logic to 05 cell 2 building `weekly`, `season_table` (sm3/sm5 targets, fragile flag, sw_true), `EVAL`, `ev`; load `baseline04` and `bl05` from `results/`.
- [ ] **Step 4:** Assertions: `assert len(EVAL)==19 and int(ev["fragile_peak_week"].sum())==8`. Print eval season count.
- [ ] **Step 5 (verify):** Run `nbconvert` (see Verify Command below). Expect clean, assertions pass, prints "eval seasons: 19".

---

### Task 2: Leakage-safe through-W features + firewall audit

**Files:** Modify `notebooks/06_regression_and_curve.ipynb`

**Interfaces produced:** `features_at(s, W) -> dict(cum_ili_thruW, dominant_strain_thruW, vax_coverage_thruW)`; `feat_df(W)` returning a per-eval-season DataFrame of features.

- [ ] **Step 1:** Build `cum_ili_thruW`: for season `s`, `weekly[(season==s)&(sw<=W)]["% WEIGHTED ILI"].sum()`.
- [ ] **Step 2:** Build `dominant_strain_thruW`: reload NREVSS (stitched Combined≤2014 / PHL≥2015), attach `sw`, cumulate subtype buckets over `sw ≤ W`, take idxmax; if zero specimens through W, record `"none"`.
- [ ] **Step 3:** Build `vax_coverage_thruW`: FluVaxView national all-ages (both labels), monthly; map each Month to its `sw` position (Sep=wk~36→ season start; compute the season-week of the first day of that month) and take max coverage among months whose season-week ≤ W; NaN if none by W.
- [ ] **Step 4 (firewall audit, the critical assert):** For each season and each W, recompute `cum_ili_thruW` and confirm it equals the sum over exactly the weeks with `sw≤W`, and that `cum_ili_thruW(W)` is nondecreasing in W and strictly uses no `sw>W` row. Assert: `cum_ili_thruW(W) == weekly[(season==s)&(sw<=W)]["% WEIGHTED ILI"].sum()` for all (s,W). Assert vax month-mapping never pulls a month whose season-week > W.
- [ ] **Step 5:** Print `feat_df(12)` for inspection (strain distribution, vax coverage range, cum_ili range).
- [ ] **Step 6 (verify):** `nbconvert` clean; firewall asserts pass; printed strain counts plausible (H3N2 majority).

---

### Task 3: Univariate regression — real-time severity forecast

**Files:** Modify `notebooks/06_regression_and_curve.ipynb`

**Interfaces produced:** `uni_results` DataFrame (per model×W severity MAE/RMSE), reuses 05-style `ili_mae/ili_rmse`.

- [ ] **Step 1:** Metric helpers (guarded for empty): `ili_mae(pred,true)`, `ili_rmse(pred,true)`.
- [ ] **Step 2:** For each W, LOSO: for held-out `s`, fit OLS `peak_ili_pct ~ cum_ili_thruW` on the other 18 seasons (numpy `polyfit` deg 1), predict `s`. Collect predictions.
- [ ] **Step 3:** Compute severity MAE/RMSE per W; also compute baseline C severity and climatology severity on the same seasons for reference.
- [ ] **Step 4:** Print table: W, uni_MAE, uni_RMSE, baselineC_MAE, climatology_MAE.
- [ ] **Step 5 (verify):** `nbconvert` clean; univariate MAE finite and in a sane range (roughly 1–2 ILI pts); note whether it beats climatology (expected: comparable, sharpening slightly with W).

---

### Task 4: Explanatory ridge (retrospective) — NumPy closed-form

**Files:** Modify `notebooks/06_regression_and_curve.ipynb`

**Interfaces produced:** `ridge_fit(X, y, lam)`, `ridge_loso(W)`; `ridge_results` DataFrame; `ridge_coefs` (standardized coefficients per W).

- [ ] **Step 1:** `ridge_fit(Xstd, y, lam)`: closed form `w = (XᵀX + lam·I)⁻¹ Xᵀ(y-ȳ)` on standardized X with intercept = ȳ; return weights+intercept+scaler stats. Do NOT penalize the intercept.
- [ ] **Step 2:** Feature matrix at W: `cum_ili_thruW` + one-hot(`dominant_strain_thruW` over {A(H1N1),A(H3N2),B}) + `vax_coverage_thruW`. Impute missing vax with training-fold mean (fit on train only — no leakage across the LOSO fold).
- [ ] **Step 3:** Inner LOO-CV over `lam ∈ {0.01,0.1,1,10,100}` on the 18 training seasons; pick min-CV-MAE lam; refit on all 18; predict held-out. Standardize using training-fold stats only.
- [ ] **Step 4:** Report LOSO severity MAE/RMSE per W and standardized coefficients (sign/magnitude of strain and vax). Compare to Task 3 univariate (does strain+vax help?).
- [ ] **Step 5:** Markdown: explicit label that this is retrospective/explanatory (strain reporting lag; vax survey revision), not a real-time forecast.
- [ ] **Step 6 (verify):** `nbconvert` clean; ridge MAE finite; print coefficients; state plainly whether strain/vax lower the severity MAE vs univariate (honest either way).

---

### Task 5: Gaussian phenomenological fit — severity and timing

**Files:** Modify `notebooks/06_regression_and_curve.ipynb`

**Interfaces produced:** `gauss_forecast(s, W, cap) -> dict(status, peak_read_sw, pred_peak_ili, peak_ambiguous, mu, at_bound)`; feeds a `gres` DataFrame parallel to 05's `res`.

- [ ] **Step 1:** `from scipy.optimize import curve_fit`; define `g(t,c,A,mu,sigma)=c+A*np.exp(-(t-mu)**2/(2*sigma**2))`.
- [ ] **Step 2:** `gauss_forecast`: slice `obs = weekly[(season==s)&(sw<=W)]`; fit `g` to (obs.sw, obs ILI) with bounds `c∈[0,min_obs]`, `A∈[0,cap]`, `mu∈[1,52]`, `sigma∈[1,20]`, seed (min, max-min, sw@max, 4). On exception → status `skip:fit-fail`.
- [ ] **Step 3:** Readout: severity=`c+A`; peak `mu` → nearest sw in the forecast region. Status: `peak_already_observed_at_W` if `mu ≤ W`, else `forecast`. `peak_ambiguous` if any bound hit within tol OR fitted curve is monotone over the horizon (reuse 05's plateau idea: max attained at >1 forecast week within 1e-9). `at_bound` recorded.
- [ ] **Step 4:** Run over model="Gaussian", all W, all EVAL (LOSO `cap` = training-max peak, `season!=s`). Build `gres`; assert firewall: every fit used only `sw≤W` (by construction), and every `forecast` peak has `peak_read_sw>W`.
- [ ] **Step 5:** Metrics: severity MAE/RMSE (all forecast seasons); peak-week MAE/within±1 on the non-ambiguous forecast subset (reuse 05 `wk_mae/wk_w1`), with `n_pw_defined`/`n_pw_ambiguous`/`n_at_bound` printed. Compare peak-week to baseline C and to 05's ARIMA/Prophet at the same W.
- [ ] **Step 6:** Print the expected-instability note: report bound-hit counts by W (expect high at W=8, lower at W=12/16).
- [ ] **Step 7 (verify):** `nbconvert` clean; firewall asserts pass; W=8 shows many bound-hits/ambiguous (as predicted); W=16 yields more defined peaks; timing reported honestly.

---

### Task 6: Consolidated 06 summary, skill table, leak check, saved artifacts

**Files:** Modify `notebooks/06_regression_and_curve.ipynb`; Create `results/06_regression_curve_summary.{md,json}`; Create any `figures/09_*.png`.

- [ ] **Step 1:** Assemble a combined metrics table across the three 06 models (+ carry 05's ARIMA/Prophet and baseline C for the same W) on both targets where applicable.
- [ ] **Step 2:** Skill-vs-baseline-C table (severity for all three; timing for Gaussian only), reusing 05's convention (`skill>0` beats floor); include `n`/`n_pw`.
- [ ] **Step 3:** Leak-detection check mirroring 05: flag any 06 model/W with `pw_within1 > 60%` as suspected leak; print verdict.
- [ ] **Step 4:** One figure: severity MAE vs W for {climatology, baseline C, univariate, ridge, Gaussian} (`figures/09_severity_skill_by_W.png`); optionally a Gaussian example-fit panel.
- [ ] **Step 5:** Save `results/06_regression_curve_summary.md` (tabulate-free, blank-NaN) and `.json`.
- [ ] **Step 6 (verify):** `nbconvert` clean; results files written; figure saved; numbers self-consistent with earlier cells.
- [ ] **Step 7 (CHECKPOINT — CLAUDE.md):** This is model code. STOP and show Joshua the executed 06 outputs and results summary. Do not commit until he approves.

---

### Task 7: Docs updates (README + CLAUDE.md)

**Files:** Modify `README.md`, `CLAUDE.md`

- [ ] **Step 1:** README: status line → "pipeline complete 01–06"; add a Results paragraph for 06 (real-time univariate forecast, explanatory ridge finding, Gaussian on timing), honest tense; add 06 to repo-structure and methodology.
- [ ] **Step 2:** CLAUDE.md: move Dr. Mitra-confirmed items out of "Open decisions" into a "Confirmed by advisor (2026-07-08)" block — precise two-pronged framing; national-first (regional later as robustness); add ridge + Gaussian as approved models. Update "Project state" to note 06.
- [ ] **Step 3 (verify):** Re-read both; confirm no claim states 06 results before the notebook has produced them; no em dashes.

---

### Task 8: Slide deck (~10–12 min) to Dr. Mitra's structure

**Files:** Create deck via the `slides` skill (HTML), saved under `docs/` or a `slides/` dir.

- [ ] **Step 1:** Invoke the `slides` skill. Structure: motivation+data (2), holiday-artifact data-quality finding (1), model comparison with honest error metrics (2–3), Prophet calibration finding (1), next steps (1).
- [ ] **Step 2:** Populate with the ACTUAL numbers from 04/05/06 results files (no invented figures); characterization-first narrative; the precise two-pronged framing.
- [ ] **Step 3 (verify):** Deck opens/renders; every number traces to a results file; timing ~10–12 min of content.

---

### Task 9: Final Drive consolidation (Joshua shares manually)

**Files:** Upload to the existing Drive folder `1YCjOr5e_zq0k6V_Kz_hfMRMexEpKuYPw`.

- [ ] **Step 1:** Create `notebooks/` subfolder; upload notebooks 01–06 (textContent, `application/json`, disable conversion).
- [ ] **Step 2:** Upload updated README, requirements.txt to root; refreshed `results/04,05,06` summaries to `results/`.
- [ ] **Step 3:** Upload the slide deck (docs/ or root).
- [ ] **Step 4:** Confirm NOT sharing raw data / CLAUDE.md / AGENTS.md (unless Joshua overrides). Report the folder link for Joshua to share.

---

## Self-Review

**Spec coverage:** §5.1→Task 3, §5.2→Task 4, §5.3→Task 5, §4 harness→Tasks 1–2/6, §6 deliverables→Tasks 6–9, §2 framing→Tasks 7–8, §7 risks (Gaussian instability, real-time caveats, small-n, leakage)→Tasks 2/4/5. Covered.
**Placeholder scan:** feature/ridge/Gaussian math specified; no "handle edge cases" left abstract (skip/ambiguous statuses defined). OK.
**Type consistency:** `peak_ambiguous`, `wk_mae/wk_w1`, `sw`, `cap`, status strings reused exactly as in 05. OK.
**Checkpoints:** model-code stop at Task 6 Step 7; no auto-commit anywhere (CLAUDE.md).
