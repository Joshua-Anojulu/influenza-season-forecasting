---
review_provenance:
  schema_version: 2
  status: approved-final
  rounds:
    - round: 1
      reviewer: codex
      model: gpt-5.5
      version: codex-cli/0.145.0
      session: 019fc20f-2997-7200-b2eb-c3979a205c0e
      grounding: repo
      qualifying: true
      verdict: REVISE
      body_sha256: 8dc31674cb02689291443f219e562f48912b20224b69fbee43a908754afd0d53
    - round: 2
      reviewer: codex
      model: gpt-5.5
      version: codex-cli/0.145.0
      session: 019fc20f-2997-7200-b2eb-c3979a205c0e
      grounding: repo
      qualifying: true
      verdict: APPROVED
      body_sha256: not-recorded-body-superseded-before-hashing
      note: >-
        Approved with six tightening fixes attached. Those fixes were applied, so this
        approval covers a body that no longer exists. The hash was not captured before the
        edits; it is recorded as absent rather than reconstructed. Round 3 re-approves the
        final body and is the round provenance rests on.
    - round: 3
      reviewer: codex
      model: gpt-5.5
      version: codex-cli/0.145.0
      session: 019fc20f-2997-7200-b2eb-c3979a205c0e
      grounding: repo
      qualifying: true
      verdict: APPROVED
      body_sha256: 968a5fafb91b95c0bef4a0c008648738b1c0c32751cce9521129fa91189bdccc
  historical_cross_model_review: true
  final_body_cross_model_approved: true
  final_body_sha256: 968a5fafb91b95c0bef4a0c008648738b1c0c32751cce9521129fa91189bdccc
  degraded_rounds: []
---

# Plan: CDC-anchored season severity tiers, D2 regional heatmap, and a 3-class severity classifier
_Locked via grill, by Claude + Joshua, 2026-08-02. Revised after Codex round 1._

## Goal

Define the season severity tier variable this project has deferred since `PLAN.md`, anchored to CDC's
published intensity thresholds per Dr. Mitra's 2026-08-02 ruling ("use CDC's published intensity
thresholds. Anchoring to the literature is cleaner for a student project and more defensible than
self-defined quantiles"). Then build its two consumers: D2, the HHS-region season severity figure,
which his same email unblocked as "option 2, pull regional ILI for the heatmap only, keep the models
national"; and the Random Forest severity classifier, an optional stretch goal since `PLAN.md` and
the last unbuilt item in the template.

The tier definition is a target definition, so working-agreement checkpoint 2 applies: Joshua reviews
the concrete threshold mapping and the labeled season table before any of it is committed.

## Approach

### Step 0. Acquire regional ILINet (prerequisite, blocks Step 4 only)

`data/raw/ILINet.csv` is National-only: `REGION TYPE` has exactly one distinct value, `National`.
D2 needs a second file, downloaded by Joshua from CDC FluView with region type **HHS Regions**, saved
as `data/raw/ILINet_regional.csv`. It is gitignored like every other raw file.

On arrival, assert before use: 10 distinct regions; `% WEIGHTED ILI` present and non-null across the
2003 wk40 to 2025 wk39 span; the same `skiprows=1` and `X` sentinel quirks as the national file. If
`% WEIGHTED ILI` is unpopulated at region level, STOP and escalate rather than silently substituting
`%UNWEIGHTED ILI`, which is a different measure and not what the thresholds were built on.

### Step 1. Pin the thresholds as cited constants

One module-level constants block, the only place these numbers appear:

```
CDC_ILI_IT = {"IT50": 4.4, "IT90": 6.6, "IT98": 8.6}   # percent of outpatient visits, all ages
CDC_IT_SOURCE = "Biggerstaff M, et al. Am J Epidemiol. 2018;187(5):1040-1050. doi:10.1093/aje/kwx334"
CDC_IT_REFERENCE_SEASONS = "2003-04 through 2014-15, excluding the 2009 pandemic"
CDC_IT_VINTAGE_NOTE = "2018 published values. CDC's current operational values are not pinned here \
and are not claimed to differ; using the published historical values is a deliberate methodological \
choice, made for citability."
```

**Citation split, and it is not cosmetic.** The framework, the tier names, and the rule that a season
is classified by where its indicators peak relative to the thresholds come from CDC's current
severity-assessment documentation. The specific numbers 4.4 / 6.6 / 8.6 come from Biggerstaff 2018
and are that paper's historical values, not CDC's present operational ones. Artifacts cite each to
the right source and never say "CDC's current thresholds."

Tier assignment uses CDC's four labels, verbatim:

| Tier | Rule |
|---|---|
| Low | peak < IT50 |
| Moderate | IT50 <= peak < IT90 |
| High | IT90 <= peak < IT98 |
| Very High | peak >= IT98 |

The label set is `Low / Moderate / High / Very High`, not `Low / Moderate / High / Severe`.

### Step 2. Define the classification statistic and assign national tiers

**The statistic compared to the thresholds is the season's peak weekly `% WEIGHTED ILI`, the raw
maximum.** This is a correction from the first draft of this plan, which used the geometric mean of
the season's three highest weeks. That conflated two different things: the geometric mean of the
highest values is how the Moving Epidemic Method *estimates the thresholds* from a pool of reference
seasons; the quantity a season is then *classified* on is where its indicator peaked. Biggerstaff
classifies on the season maximum, and CDC's in-season method compares weekly and peak values to the
thresholds.

The correction is empirically supported, not just definitional: peak-weekly-max reproduces CDC's
published season classifications on **9 of 12** seasons, against 8 of 12 for the geometric mean.

New column: `season_peak_max`. The project's existing `peak_ili_pct`, the maximum of a 3-week centered
smoother, is untouched and remains the regression target for 05/06/07. This plan adds a parallel
variable; it does not redefine the committed one.

**A tension this project is uniquely placed to state.** Notebook 02's holiday-artifact finding is
that the raw weekly maximum is inflated by the week-52 reporting artifact, which is why this project
smooths. CDC's classification statistic is the unsmoothed maximum. So the CDC-anchored tier is, by
this project's own published finding, holiday-inflated for the 5 seasons flagged `holiday_shift`.
That is a reportable observation about the CDC statistic, not a reason to deviate from it. The
smoothed-peak tier is therefore computed as a **mandatory** sensitivity, not an optional one.

Emit per season: `season_peak_max`, `tier_max` (definition of record), `tier_smoothed`, `tier_gm3`
(retained only to document the discarded alternative), `tier_margin` (distance from `season_peak_max`
to the nearest threshold), and `tier_boundary_sensitive` (true when `tier_margin < 0.30` **or** the
three variants disagree). The flag mirrors the existing `peak_week_smoothing_sensitive` and
`fragile_peak_week` idiom in 02.

Values recomputed from the committed raw data after the round-1 corrections:

- Tier counts, 19-season modeling set: **Low 4, Moderate 9, High 6, Very High 0.**
- Tier counts, 21 assessed seasons (2020-21 excluded, see below): **Low 4, Moderate 10, High 7.**
- `tier_boundary_sensitive`, modeling set: **2010-11** (margin 0.15), **2013-14** (0.19),
  **2023-24** (0.20), **2019-20** (variants disagree), **2021-22** (variants disagree).

Two numbers from the first draft were wrong and are corrected here. 2021-22 does not sit "0.02 above
IT50"; on the geometric mean it sat 0.0152 **below** it, and on the statistic now used its raw max is
4.89, a clear Moderate. And the earlier sensitivity list contradicted its own rule by omitting
2023-24. Both were caught in review, and no tier claim in this plan is now asserted without having
been recomputed.

### Step 3. Validate against CDC's published classifications

CDC published per-season overall severity for 2003-04 through 2014-15. Reproduce the comparison as a
first-class result in `results/08_tier_validation.md`:

**The two numbers are always reported together, in this order, and never separately:** agreement is
**9 of 12 overall**, and **9 of 10 after excluding the two pre-declared pandemic and pandemic-adjacent
seasons as non-comparable**. Quoting only the 9/10 would overstate agreement, because the two excluded
seasons are exactly two of the three disagreements. Disagreements across all 12: 2008-09, 2009-10,
2014-15. Sole disagreement among the 10 comparable seasons: 2014-15.

**2008-09 and 2009-10 are excluded from the comparable-season figure because they are not comparable,
and the repo can prove it.** 2009-10 is the pandemic season itself; 2008-09 is pandemic-adjacent.
This project's MMWR season runs to week 39, so the 2008-09 season absorbs the spring 2009 H1N1 wave:
notebook 02 already records that the 2008-09 peak falls in wk39, the final week.
Our raw max for 2008-09 is 4.89; the published CDC figure for that season is 3.6, because CDC did not
fold the pandemic wave into it. Attributing that gap to CDC's 2-of-3 vote, as the first draft did, was
a misdiagnosis. It is a season-boundary difference, and both numbers are reported side by side.

The remaining genuine disagreement, 2014-15, is attributable to the framework difference: CDC
classifies by a **2-of-3 indicator vote** across ILI, hospitalization rate, and pneumonia-and-influenza
mortality, and 2014-15 was a severe season for hospitalization among older adults while its ILI peak
was mid-range. Every artifact carrying a tier states, in the artifact itself, that it is an **ILI-only
approximation of CDC's framework using CDC's published ILI intensity thresholds**, not CDC's official
season severity classification.

**2020-21 is labeled `Not assessed`, not `Low`.** CDC did not assign a severity classification to
2020-21 because influenza activity was minimal under COVID-19 mitigations, so calling it Low would be
this project's judgement wearing CDC's label. It is excluded from all tier counts and from the
classifier, and shown as a visually distinct "not assessed" cell on D2. This also matches its existing
treatment as an excluded structural break.

### Step 4. D2, the HHS-region season severity figure

**D2 is a continuous heatmap of the regional season peak, not a regional tier map.** Cell value is
each region's `season_peak_max`, on a continuous color scale, with IT50 / IT90 / IT98 marked on the
colorbar as **national reference thresholds**. Cells are not colored by, or labeled with, a regional
tier.

**The reference marks must stay visually subordinate, and that is a hard requirement, not styling.**
Thin labeled ticks on the colorbar only. No banded or discretized color scale, no contour bands across
the grid, no per-cell tier text. Any of those would re-encode the invalid regional tier comparison
through the visual channel after the plan removed it from the labels. The colorbar legend reads
"national reference thresholds" and never "regional thresholds."

This is a change from the first draft, which colored regional cells by tier using national thresholds
while simultaneously admitting regional baselines differ. That was labeling data with a category the
thresholds do not license. A continuous scale with national reference lines satisfies the template's
heatmap requirement, keeps every number on the figure true, and still shows the season-by-region
severity structure Dr. Mitra asked for.

The figure and its results file state plainly that the reference lines are national all-ages ILI
thresholds and that regional ILI has region-specific baselines, so a region crossing a national line
is not the same claim as CDC calling that region-season High.

**Optional exploratory sensitivity, not part of the definition:** region-specific thresholds derived
by the Moving Epidemic Method, reported only to quantify how far regional baselines sit from the
national reference. The first draft mischaracterized this as "self-defined quantiles"; that was wrong.
MEM is a published algorithm (Vega et al.), and the same one CDC uses. The reason it stays exploratory
is not that it is ad hoc but that CDC publishes no regional ITs, so region-specific values computed
here would be this project's, not the literature's, and Dr. Mitra's ruling was to anchor to the
literature. It never colors the figure and never labels a season.

Pandemic seasons 2009-10 and 2020-21 appear, visually marked as excluded structural breaks, consistent
with the treatment Dr. Mitra endorsed ("the pandemic season out-of-sample documentation is a good
instinct, keep it"). 2008-09 is marked too, for the wk39 boundary artifact recorded in 02.

Output: `figures/12_D2_severity_heatmap.png`, `results/08_regional_severity.{json,md}`.

### Step 5. The severity classifier

**Three classes, not four.** Very High is empty across every assessed season, so a 4-class model
carries a class it can never predict and whose per-class F1 is 0 by construction. The empty class is
documented, not silently dropped.

Target: `tier_max` on the 19-season modeling set. Class counts are **Low 4, Moderate 9, High 6**.
Low at n=4 means some LOSO folds train on 3 examples of it; that is thin, is stated up front, and is
part of why the baselines below exist.

**Two feature sets, and the primary one is real-time.** The first draft said to reuse notebook 07's
Panel A unchanged and called the result a real-time classifier. That was wrong: Panel A includes
reporting-lagged strain, and CLAUDE.md already records that panels containing it are retrospective and
explanatory. Splitting it:

- **Primary, real-time:** ILI-derived through-W features only (`cum_ili`, `ili_lag_1..4`,
  `ili_rolling4`), near-real-time under the project's index-cutoff convention. ILINet is itself
  revised after initial publication, so "near-real-time" is the accurate claim and "without reporting
  lag" would not be; the substantive contrast is with strain, hospitalization and vaccine coverage,
  which are materially lag-reported or survey-revised. Audited under the identical leakage firewall as
  05/06/07.
- **Secondary, retrospective:** the full Panel A including strain, reported as explanatory only and
  never described as a forecast.

The label derives from the full-season peak, which is by design: the firewall governs features, not
the target, exactly as in 05 and 06 where the target is also the realized season peak. The audit must
confirm no *feature* window reaches past W, and must state that the label does, so the distinction is
explicit rather than assumed.

**Baselines before models (checkpoint 3).** The classifier ships only after being measured against:

- `B0` majority class (predicts Moderate always).
- `B1` stratified random, drawing from **each LOSO training fold's** class distribution, seeded, over
  many repeats, with the mean and spread reported. Drawing from full-sample proportions would leak the
  held-out season's label distribution.
- `B2` climatology threshold: apply the tiers to the LOSO climatology mean peak. Expected to be
  degenerate; a degenerate baseline that scores well is itself the finding.
- `B3` **regression-then-threshold**, the baseline that matters: predict the season peak with the
  cumulative-ILI univariate regression, then apply the thresholds to the prediction.

**B2 and B3 are refit against `season_peak_max`, not reused from 06.** Notebook 06's regression targets
`peak_ili_pct`, the smoothed peak; thresholding its predictions against thresholds defined on the raw
max would compare two different scales. The refit is a like-for-like rerun of 06's model specification
on the new target, and its own MAE is reported so the comparison to 06 stays visible.

Stated once, explicitly, so the notebook cannot drift from it: **B3 is `y = season_peak_max`,
`x = cum_ili_through_W`, refit under LOSO inside each W in {8, 12, 16}, with no reuse of any stored
notebook 06 prediction.** B2 likewise recomputes LOSO climatology on `season_peak_max`.

**Pre-registered model configuration**, locked before any result is seen, to stop hyperparameters from
becoming a post-hoc search at n=19: `random_state=42`, `n_estimators=500`, `min_samples_leaf=2`,
`max_depth=None`, `class_weight="balanced"`, `max_features="sqrt"`. Any tuning must be nested inside
the LOSO loop or it does not happen at all.

**Scoring.** Report, in this order: the 3x3 confusion matrix first, then per-class F1, accuracy, and
macro-F1. Each fold predicts one held-out season, so predictions are pooled across all 19 folds and
macro-F1 computed once over the pool; a per-fold average is undefined on a single-sample fold. Macro-F1
at n=19 with these class counts is high-variance, so it is pre-registered as **descriptive only**, and
carries a bootstrap interval over seasons rather than being reported as a bare point estimate.

Pre-commit, before seeing results: if the RF does not beat B3, that is the finding, reported in the
same voice as 05's negative result and 07's unsupported H1. No post-hoc feature search to rescue it.

Output: `results/08_severity_classifier.{json,md}`.

### Step 6. Artifacts, documentation, and the review gate

One new notebook, `08_severity_tiers.ipynb`, covering Steps 1 through 5, executed clean with outputs
cleared, consistent with every other notebook in the repo.

Before anything is committed, present to Joshua: the constants block, the full labeled season table
with margins and sensitivity flags, and the validation table. That is checkpoint 2. Update `CLAUDE.md`,
`AGENTS.md` and `README.md` only after he approves the definition. No commit and no push without his
explicit instruction (checkpoint 4), and confirm `git ls-files data/` is empty before any push.

## Key decisions and tradeoffs

**ILI-only, rather than reproducing CDC's real 2-of-3 framework.** Reproducing it needs pneumonia and
influenza mortality, which this project does not have, plus hospitalization rates that only begin in
2009-10. Dr. Mitra's own reasoning in the same email settles it: on the feature question he wrote "do
not restrict to 2009+, you would lose too many seasons." A 3-indicator framework restricts full
fidelity to exactly the range he told us not to restrict to, and he asked for what is "cleaner for a
student project." Rejected alternatives: pull NCHS mortality and implement the full vote (scope creep);
adopt CDC's 12 published labels verbatim (leaves 10 of 22 seasons unlabeled, including every season
since 2015-16, gutting both consumers).

**Peak weekly maximum as the classification statistic**, accepting that it diverges from the project's
own smoothed target and is holiday-inflated in 5 seasons by this project's own finding. The alternative
of tiering the smoothed peak keeps the project internally tidy but compares a statistic to thresholds
never built for it. Since the entire point of the ruling is defensibility against the literature,
internal tidiness loses, and the divergence is reported as a sensitivity rather than hidden.

**A continuous regional figure instead of a regional tier map.** Costs the visual simplicity of four
discrete colors. Buys a figure where every element is a claim the data supports. Inventing regional
thresholds was rejected as departing from the literature anchor; asserting national tiers over regional
data was rejected as a false label.

**Three classes.** Collapsing an empty class is a defensible response to the data; padding macro-F1
with a structurally impossible class is not. The deliverable no longer matches the template's four-tier
wording, recorded as a deviation alongside the existing `season_week` and ARIMA-interval deviations.

**Regression-then-threshold as the baseline of record.** This makes it materially harder for the RF to
look good, which is the point. The project's one honest positive result so far is a one-variable
regression, and a classifier that cannot beat discretizing it is not a contribution.

## Risks and open questions

**The regional figure still carries national reference lines**, which is a weaker claim than tiering
but not a free one. If the exploratory MEM diagnostic shows regional baselines diverge enough that the
national lines are uninformative for most regions, the honest response is to say so on the figure and
present D2 as regional peak structure with no reference lines at all.

**Threshold vintage.** 4.4 / 6.6 / 8.6 are Biggerstaff 2018's historical values. CDC's current
operational values are **not pinned here and are not asserted to differ**; no source for them is
recorded in this repo, and automated retrieval failed during planning (cdc.gov returned HTTP 403 to
every programmatic fetch attempted from this session, though the pages are reachable in a browser).
The plan uses the citable, DOI-backed 2018 values and names that vintage as a deliberate
methodological choice. Joshua should confirm the current published values before any external writeup;
if they differ materially, the tier table shifts and this decision is revisited.

**Circularity between the thresholds and the labels.** The 2018 ITs were derived from 2003-04 through
2014-15, overlapping 12 of the 19 modeling seasons, so for those seasons the label definition is
in-sample to the threshold derivation. Harmless for a descriptive figure. For the LOSO classifier it
means the label definition has seen the held-out season. Not fixable while keeping the CDC anchor;
stated as a limitation.

**n=19 with 3 classes is underpowered**, and the Low class has only 4 members. Overfitting is the
expected outcome; the baselines and the bootstrap interval exist to make that visible. Stated up front,
not discovered.

**Five seasons are boundary-sensitive** (2010-11, 2013-14, 2019-20, 2021-22, 2023-24), so any claim
resting on one of their tiers is fragile. The flag travels with the data so downstream artifacts cannot
quietly rely on a coin flip.

**Regional `% WEIGHTED ILI` availability is unverified** until the Step 0 file exists. If unpopulated
at region level, Step 4 stops for a decision rather than substituting a different measure.

## Out of scope

- **Pneumonia and influenza mortality**, and therefore CDC's actual 2-of-3 season classification.
- **Regional modeling of any kind.** Dr. Mitra: "keep the models national," on 2026-07-08 and again on
  2026-08-02. Regional data enters as a figure input and nothing else.
- **Age-stratified tiers.** CDC publishes ITs by age group; no age-resolved modeling is in this project.
- **Adopting region-specific MEM thresholds as a definition.** Permitted only as the labeled
  exploratory diagnostic in Step 4.
- **Changing `peak_ili_pct`, or any committed output of notebooks 02 through 07.**
- **D4, the research paper.** Sequenced after this, unblocked, not part of this plan.
- **The `src/` refactor.** Known debt, still deferred.
- **Any git commit or push.** Checkpoints 2 and 4.
