# Plan Review Log: Close the template-adherence gaps that do not require advisor input

Act 1 (grill) complete — plan locked with the user. MAX_ROUNDS=5.

Grill resolved five decisions: severity tiers deferred (target definition with no live consumer);
two feature panels at 19 and 14 seasons rather than one merged panel; pandemic seasons scored pure
out-of-sample and reported per-season; notebook 05 extended rather than duplicated, with new feature
work in a new 07; D1 season selection rule-based and fixed before inspecting forecasts.

Three findings from codebase exploration reshaped the plan before it was written: ARIMA returns no
prediction intervals, so D1 must be a Prophet figure; D1 and the pandemic special-case scoring are
the same job, since D1's required pandemic labels only exist if those seasons are forecast; and
`season_week` from template section 3.1 is structurally inapplicable at a fixed decision week.

## Round 1 — Codex

The plan is not implementation-ready: its own reproducibility guarantees conflict with the current notebook code.

**Findings**

1. [Certain] `PLAN.md` adds `traj`, `pi_lo_traj`, `pi_hi_traj`, and `h_sw` to `run_one`, but `05_forecasting.ipynb` writes `aud = res.copy()` straight to `results/05_audit.json`, so the audit cannot stay byte-identical.
Fix: keep trajectories in a separate sidecar list, or explicitly drop those columns before `aud.to_json`.

2. [Certain] The goal says no current `results/` file may change, but Step 2 intentionally changes `results/06_regression_curve_summary.json` by adding `ridge_coefficients`.
Fix: write `results/06_ridge_coefficients.json`, or explicitly whitelist this schema expansion and compare all pre-existing keys byte-for-byte.

3. [Certain] Step 1.6 protects `05_survivorship.json` but omits `05_survivorship.md`, even though that markdown is a committed result artifact.
Fix: hash and compare every pre-existing `results/05_*` artifact, including `.md`.

4. [Likely] “Prophet is seeded, so this must hold exactly” is too strong as a guarantee. Same-machine reproducibility is plausible with pinned versions, but optimizer and serialization drift can still occur.
Fix: define the gate as pre/post hashes against `git HEAD`, and stop on any protected artifact diff rather than asserting seed alone is sufficient.

5. [Certain] Panel A’s feature count is misstated. `cum_ili`, four lags, rolling4, and three strain one-hots are 9 model columns, not “seven conceptual features”; Panel B becomes 11 columns at n=14.
Fix: correct the count and treat Panel B as high-variance descriptive analysis, not evidence of feature importance.

6. [Certain] `ili_rolling4` is exactly a linear combination of `ili_lag_1..4`, so drop-one-feature deltas are not reliable either, not just coefficients.
Fix: use grouped ablations, for example drop the entire recent-ILI lag block, rolling summary, strain block, vax, and hosp blocks.

7. [Likely] H1 is overclaimed. A head-to-head LOSO MAE comparison at n=19 across three W values cannot establish “strongest predictor” without uncertainty and multiple-comparison caution.
Fix: report paired per-season error deltas with a simple sign test or bootstrap CI, and phrase H1 as descriptive support or no support.

8. [Certain] `hosp_rate_lag1` “most recent season-week <= W” is not a real availability firewall because FluSurv-NET is reporting-lagged.
Fix: either apply an explicit reporting lag cutoff or label every hospitalization model strictly retrospective, not real-time.

9. [Likely] Scoring 2008-09, 2009-10, and 2020-21 with a cap learned from all 19 modeled seasons is excluded-from-training, but not historically prospective because it uses future seasons.
Fix: call this a structural-break stress test, or add a chronological sensitivity using only seasons available before the special case.

10. [Certain] The plan effectively affirms Option B while `CLAUDE.md` says Option B vs Option A is still pending advisor sign-off.
Fix: mark every two-panel feature result as “Option B assumption, pending advisor decision.”

11. [Likely] The `season_week` deviation is overstated. It has zero variance in a fixed-W season-level table, but it is still a valid time index in trajectory models and plots.
Fix: document the narrower deviation: not usable as a season-level predictor at fixed W, retained as the within-season time axis.

12. [Certain] The current worktree has untracked `PLAN.md` and `PLAN-REVIEW-LOG.md`, so “diff to confirm” must be precise about comparing protected tracked artifacts, not the whole tree.
Fix: use explicit protected file hashes or `git diff -- results figures notebooks/05_forecasting.ipynb notebooks/06_regression_and_curve.ipynb`.

VERDICT: REVISE
### Claude's response

Accepted all twelve findings. Three were load-bearing enough to change the shape of the plan.

**#1 (accepted, verified independently).** Confirmed against the notebook: cell 18 does
`aud = res.copy()` then `aud.to_json(RESULTS_DIR / "05_audit.json")`. Adding trajectory fields to
`run_one`'s record would have flowed into `res` and silently mutated a committed artifact while the
plan claimed byte-identical reproduction. Fixed structurally rather than with a drop step:
trajectories now accumulate in a sidecar `TRAJ` list and `run_one`'s return value is unchanged field
for field, so `res` is incapable of changing.

**#6 (accepted, sharpest catch).** `ili_rolling4` is the exact arithmetic mean of `ili_lag_1..4`.
Under that exact linear dependence, drop-one-feature deltas are as invalid as raw coefficients,
because dropping one lag leaves its information recoverable from the remaining columns. My original
mitigation was insufficient. Replaced with grouped block ablations (lag block, rolling summary,
cum_ili, strain block, plus vax and hosp in Panel B), which still yields 5 and 7 comparison groups
and so satisfies the template's five-feature requirement with a valid measure.

**#5 (accepted).** Column counts corrected and made the primary framing, since columns not concepts
drive variance: Panel A is 9 columns at n=19, Panel B is 11 at n=14. Panel B is now explicitly
captioned high-variance descriptive analysis that cannot support an importance ranking.

Accepted with no argument: #2 (the goal contradicted itself; the guarantee is now stated precisely
with one whitelisted additive key and a key-by-key comparison for that file), #3 and #12 (protected
artifact list now enumerated explicitly and includes every `.md`, compared against `git HEAD` rather
than a whole-tree diff that untracked `PLAN.md` would pollute), #4 (seeding makes reproduction
expected, not guaranteed; the hash gate is now the authority), #7 (H1 now reported with paired
per-season deltas, a sign test and a bootstrap CI, phrased as support or no support), #8 (index
availability and reporting availability separated explicitly; any model containing hosp, vax or
strain is retrospective), #9 (the pandemic scoring is now labeled a structural-break stress test, not
a prospective forecast, since its cap and training set draw on chronologically later seasons),
#10 (all two-panel results marked "Option B assumption, pending advisor decision"), and #11 (the
`season_week` deviation narrowed to "not usable as a season-level predictor at fixed W", retained as
the within-season time axis, which is how the project already uses it).

Declined, with reason: Codex's optional alternative under #9, a chronologically-restricted refit per
special case. It would make those forecasts genuinely prospective, but it requires rebuilding the
training set per season for a stress test that is explanatory by design. Recorded as an out-of-scope
item with the limitation stated instead.

## Round 2 — Codex

_(Note: round 2 was re-issued after a harness flag error. `codex exec resume` rejects `-C`; the session already carries its cwd. First attempt exited 2 with a stale verdict file; corrected invocation below is the genuine re-review.)_

The revision fixed most prior methodological problems, but it still has two implementation traps that can make the plan fail or leak during model selection.

**Findings**

1. [Certain] The reproducibility gate still suggests `git diff --exit-code -- results figures notebooks/05_forecasting.ipynb notebooks/06_regression_and_curve.ipynb`, but Steps 1 and 2 intentionally modify notebooks 05 and 06. That command will fail even when the protected artifacts are unchanged.
Fix: remove notebooks from the no-diff gate, hash only protected artifacts, and review notebook diffs separately as expected implementation changes.

2. [Likely] The sidecar `TRAJ` can silently duplicate trajectories if the notebook cell is rerun without kernel restart.
Fix: initialize `TRAJ = []` immediately before the main forecast loop and assert unique `(model, season, W, split)` keys plus expected counts before writing `05_trajectories.json`.

3. [Certain] Step 3.4 says to reuse 06’s imputation path for “any feature with missing values,” but the existing 06 code only imputes one last-column `vax` feature. It is not general enough for a multi-feature Panel B.
Fix: implement column-wise train-mean imputation for all NaN-containing columns.

4. [Likely] The nested lambda CV can leak through imputation if missing values are filled once using the full outer-training fold before inner validation splits.
Fix: in each inner CV split, compute imputation means from the inner-training rows only, then apply them to the inner-validation row.

5. [Likely] The protected artifact list says `figures/01a..09`, which is human-readable but not an executable file list and risks missing `01b`.
Fix: enumerate the exact ten protected figure paths or generate them from `git ls-files figures`.

Everything else I flagged in round 1 is substantially addressed: trajectory pollution of `05_audit`, the 06 JSON schema exception, H1 uncertainty, collinearity, reporting-availability caveats, Option B provisional labeling, structural-break labeling, and the narrower `season_week` deviation.

VERDICT: REVISE
### Claude's response

Round 1's twelve are confirmed by Codex as substantially addressed. Five new implementation-level
findings this round, all correct, all accepted.

**#1 (accepted).** The no-diff gate wrongly included notebooks 05 and 06, which steps 1 and 2 edit
by design; `git diff --exit-code` on them would fail on a correct run. Gate now covers DATA artifacts
only (results + figures), generated from `git ls-files` and hashed against HEAD; notebook diffs are
reviewed separately as expected changes.

**#3 and #4 (accepted, verified against the notebook).** Confirmed 06 cell 11 imputes only the last
column (`Xtr[:, -1]`) and does so once at the outer fold before the inner CV splits. Panel B has two
maybe-missing enrichment columns (vax and hosp), so single-column imputation is insufficient, and the
once-at-outer-fold fill leaks fold-mates into inner lambda selection. 07 now imputes column-wise and
inside each inner split. 06 is protected and unchanged; the divergence is deliberate and documented,
since 06's simpler path only nudges lambda selection on one feature.

**#2 (accepted).** `TRAJ` now initializes immediately before the loop with a uniqueness and
expected-count assertion before write, so a cell re-run without kernel restart cannot silently
duplicate trajectory records.

**#5 (accepted).** The `figures/01a..09` shorthand is replaced by the ten exact figure paths, and the
protected set is generated from `git ls-files results figures` at runtime so nothing (e.g. `01b`) is
silently missed.

No findings declined this round.

## Round 3 — Codex

The five round-2 blockers are addressed. The plan is now implementable without the obvious artifact-drift and imputation failures.

Remaining issue, not material enough to block:

1. [Likely] The hash-gate wording still has a small ambiguity: it says to generate the protected set from `git ls-files results figures` and verify with per-file SHA-256, but `results/06_regression_curve_summary.json` is intentionally changed by adding `ridge_coefficients`.
Fix: state explicitly that every protected file uses SHA-256 except `results/06_regression_curve_summary.json`, which uses key-by-key comparison for pre-existing keys.

No new statistical, leakage, schema, or observability blocker surfaced. The prior issues around trajectory side effects, notebook diffs, exact figure enumeration, nested-CV imputation, H1 uncertainty, collinearity, Option B caveating, structural-break labeling, and `season_week` scope are all handled.

VERDICT: APPROVED
### Claude's response

APPROVED. Converged in 3 rounds. The single residual (a wording ambiguity: the SHA-256 gate versus
the one intentionally-changed JSON) was folded in: the gate now states every protected file is
hashed except `results/06_regression_curve_summary.json`, which is checked key-by-key for its
pre-existing keys. No open disagreements between the two models.
