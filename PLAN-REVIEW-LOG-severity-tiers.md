# Plan review log: CDC-anchored severity tiers, D2, and the severity classifier

Companion to `PLAN-severity-tiers.md`. Append-only argument transcript.

Separate from `PLAN-REVIEW-LOG.md`, which records the 2026-07-21 template-adherence plan that was
approved, built by Codex, verified by Claude, and committed as `e65cc1d`. That file is closed.

## Task

Define the season severity tier variable deferred since `PLAN.md`, anchored to CDC's published
intensity thresholds per Dr. Mitra's 2026-08-02 ruling, then build its two consumers: D2, the
HHS-region severity heatmap, and the Random Forest severity classifier.

## Resolved caps

| Var | Value |
|---|---|
| `MAX_ROUNDS` | 5 |
| `MAX_ATTEMPTS` | 8 |
| `PLAN_FILE` | `PLAN-severity-tiers.md` |
| `LOG_FILE` | `PLAN-REVIEW-LOG-severity-tiers.md` |
| `reviewer` | `codex` (grounding `repo`, selection `auto`, status verified 2026-07-26) |
| observed version | `codex-cli 0.145.0` via pinned release binary |

**Version pinning note.** The `codex` on PATH is 0.146.0, which is outside the adapter's
`verified_versions` range (`>=0.130 <0.146`) and therefore fails closed to UNVERIFIED. It is
independently unusable as a reviewer: on 0.146.0 the read-only sandbox blocks the shell write but
Codex retries via `apply_patch`, which succeeds. Every round in this log runs the pinned
`0.145.0-x86_64-pc-windows-msvc` binary, which denies both write paths.

## Act 1 summary (grill, Claude and Joshua, 2026-08-02)

Joshua delegated the resolution with the instruction to align with the project documents and
Dr. Mitra's stated reasoning rather than pick from menus. The tree resolved as follows.

**Anchor: ILI-only, using CDC's published all-ages ILI intensity thresholds.** Established during the
grill that CDC's actual season classification is a 2-of-3 indicator vote over ILI, hospitalization
rate, and pneumonia-and-influenza mortality, which this project cannot reproduce: it has ILI from
2003, hospitalization only from 2009, and no mortality data. The deciding argument against pulling
mortality to reproduce the framework is Dr. Mitra's own reasoning in the same email, where he
rejected restricting the feature work to 2009+ because "you would lose too many seasons." He also
asked for what is "cleaner for a student project."

**Labels are CDC's four, verbatim: Low, Moderate, High, Very High.** The task was framed as
"Low/Moderate/High/Severe"; renaming CDC's top tier while claiming to anchor to CDC was rejected as
misrepresenting the source.

**Peak statistic is CDC's own**, the geometric mean of the season's three highest weekly raw
`% WEIGHTED ILI` values, not the project's existing 3-week smoothed `peak_ili_pct`. The thresholds
are confidence limits on that specific statistic; comparing a differently constructed peak to them
makes the anchoring decorative. Both variants are persisted as a sensitivity.

**Grounding numbers established during Act 1**, computed from the committed raw data: modeling-set
tier counts are Low 6 / Moderate 8 / High 5 / Very High 0; ILI-only tiering agrees with CDC's 12
published season classifications on 8 of 12; four seasons flip tier depending on the peak statistic;
2021-22 sits 0.02 above IT50.

**Three classes for the classifier**, since Very High is empty across all 22 seasons and a 4-class
LOSO model would carry a structurally unpredictable class.

**D2 uses national thresholds on regional data**, with the mismatch stated and quantified
diagnostically, because computing region-specific thresholds is exactly the self-defined-quantile
approach the ruling rejected.

## Threshold provenance

`IT50 = 4.4`, `IT90 = 6.6`, `IT98 = 8.6` (percent of outpatient visits for ILI, all ages).
Source: Biggerstaff M, et al. American Journal of Epidemiology. 2018;187(5):1040-1050.
doi:10.1093/aje/kwx334. Reference period 2003-04 through 2014-15, excluding the 2009 pandemic.
CDC's current operational thresholds could not be retrieved during planning; cdc.gov returned
HTTP 403 to every automated fetch. Recorded as an open risk in the plan.

---

## Round 1 — codex

| | |
|---|---|
| reviewer | `codex` |
| model | gpt-5.5, reasoning effort high |
| version | codex-cli/0.145.0 (pinned release binary) |
| session | `019fc20f-2997-7200-b2eb-c3979a205c0e` |
| grounding | `repo` (read-only sandbox; canary denied both PowerShell write and apply_patch) |
| body_sha256 | `8dc31674cb02689291443f219e562f48912b20224b69fbee43a908754afd0d53` |
| verdict | **REVISE** |

### Critique (verbatim)

1. [Certain] Conflates threshold estimation with season classification. Biggerstaff's ITs are derived
   from a geometric-mean top-values distribution, but the paper's classification table uses "Maximum %
   ILI," and CDC's current method says weekly or peak indicator values are compared to ITs. Fix: use raw
   weekly maximum for a CDC-replication tier, or rename GM3 as a project-specific GM3 tier and stop
   calling it CDC's own season peak statistic.
2. [Certain] The plan's established number for 2021-22 is wrong: recomputing GM3 from local
   `data/raw/ILINet.csv` gives about 4.385, which is 0.015 below IT50, not "0.02 above IT50". Fix: remove
   all precomputed tier claims until the notebook persists an auditable table.
3. [Certain] The `tier_boundary_sensitive` list contradicts its own rule: variant disagreement triggers
   sensitivity, but 2023-24 disagrees by raw max versus GM3/smoothed and is omitted. Fix: include
   2023-24 or narrow the sensitivity definition.
4. [Certain] Treating 2020-21 as a Low tier is not CDC-anchored. CDC says 2020-21 was not assessed
   because flu activity was minimal. Fix: label 2020-21 "not assessed / structural break," exclude it
   from tier counts and classifier labels, show it visually separate.
5. [Likely] The 2008-09 validation disagreement is misdiagnosed. The plan blames CDC's 2-of-3 vote, but
   the repo's MMWR season includes the spring 2009 H1N1 boundary artifact, while Biggerstaff's 2008-09
   row is Low with max ILI 3.6. Fix: handle 2008-09 as non-comparable before using it in validation.
6. [Certain] The regional heatmap is over-labeled: it colors HHS regions as "tiers" using national
   thresholds while admitting regional baselines differ. Fix: make D2 a continuous regional peak heatmap,
   or label colors as "exceeds national all-ages ILI IT reference," not regional severity tiers.
7. [Certain] Misstates MEM by calling region-specific MEM thresholds "precisely the self-defined-quantile
   approach" the advisor rejected. MEM is a published algorithm, not ad hoc quantiles. Fix: drop the
   diagnostic or describe it as an exploratory MEM sensitivity with no definitional role.
8. [Certain] B2 and B3 are target-inconsistent. The classifier target is `tier_gm3`, but B2/B3 threshold
   predictions from models trained on `peak_ili_pct`, the smoothed peak. Fix: refit climatology and
   cumulative-ILI regression against the tier's own peak variable.
9. [Certain] Reusing notebook 07 Panel A "unchanged" makes the RF retrospective, not real-time, because
   Panel A includes reporting-lagged strain. Fix: make an ILI-only real-time classifier primary, report
   Panel A only as retrospective/explanatory.
10. [Likely] Macro-F1 pooled is defined, but n=19 with class counts 6/8/5 makes it high variance and easy
    to overinterpret. Fix: pre-register pooled macro-F1 as descriptive only, confusion matrix first, add
    bootstrap or exact uncertainty intervals.
11. [Likely] B1 "stratified random" is underspecified and can leak if it uses full-sample label
    proportions. Fix: draw from each LOSO training-fold distribution with fixed seeds over many repeats.
12. [Likely] The RF has no pre-registered hyperparameters, seed, or tuning rule. Fix: lock
    `random_state`, `n_estimators`, `max_depth` or `min_samples_leaf`, `class_weight`, and prohibit
    post-hoc tuning unless nested inside LOSO.
13. [Certain] The "current CDC values could not be retrieved" risk is stale as written. Fix: cite current
    CDC pages for framework, cite Biggerstaff only for the 4.4/6.6/8.6 historical values, and explicitly
    call the threshold vintage a methodological choice.

### Claude's response

**All 13 accepted.** Three were verified numerically against the committed raw data before acceptance
rather than taken on assertion; all three checked out.

- **#2 confirmed and it was my error.** GM3 for 2021-22 is 4.3848, which is 0.0152 *below* IT50, not
  above. I had stated the direction wrong in the plan and to Joshua. Corrected, and the correction is
  called out in the plan body so the error is visible rather than quietly patched.
- **#1 accepted, and the data supports it independently.** Switching the classification statistic from
  the geometric mean to the season's peak weekly value raises agreement with CDC's published labels from
  8/12 to **9/12**, and to **9/10** excluding pandemic-adjacent seasons. That is corroboration, not just
  a definitional preference. Went further than the fix asked: because notebook 02's own holiday-artifact
  finding says the raw maximum is inflated in 5 seasons, the smoothed-peak tier is now a **mandatory**
  sensitivity and the tension between this project's finding and CDC's statistic is written up as a
  reportable observation.
- **#5 confirmed.** Our 2008-09 raw max is 4.89 against CDC's published 3.6, and 02 already records that
  the 2008-09 peak falls in wk39, the spring 2009 H1N1 wave. It is a season-boundary difference, not the
  2-of-3 vote. Headline validation now excludes it and reports both numbers side by side.
- **#3 confirmed.** The listed set did contradict the stated rule. Corrected to the union: 2010-11,
  2013-14, 2019-20, 2021-22, 2023-24.
- **#6 accepted in its stronger form.** D2 becomes a continuous heatmap of regional season peak with
  national thresholds marked on the colorbar as reference lines only. Cells are no longer colored or
  labeled by tier.
- **#7 accepted as a correctness fix to my own wording.** Calling region-specific MEM "self-defined
  quantiles" was wrong; MEM is published and is what CDC uses. The diagnostic stays exploratory for a
  different and accurate reason: CDC publishes no regional ITs, so values computed here would be this
  project's rather than the literature's.
- **#4, #8, #9, #11, #12 accepted as written.** #9 is the most consequential of these: Panel A contains
  reporting-lagged strain, so the primary classifier is now ILI-only real-time features, with Panel A
  demoted to a retrospective secondary.
- **#10 accepted with a scope note.** Macro-F1 stays, because it is the template's named metric, but is
  pre-registered as descriptive only, reported after the confusion matrix, and carries a bootstrap
  interval.
- **#13 accepted.** Citations now split: CDC documentation for the framework and tier names, Biggerstaff
  2018 for the numeric values. The retrieval-failure note is narrowed to what actually happened, which is
  that programmatic fetches from this session got HTTP 403 while the pages are reachable in a browser.

**Nothing rejected this round.** Recomputed tier counts under the corrected statistic, modeling set:
Low 4, Moderate 9, High 6, Very High 0.

---

## Round 2 — codex

| | |
|---|---|
| session | `019fc20f-2997-7200-b2eb-c3979a205c0e` (resumed, id echo verified) |
| grounding | `repo` |
| body_sha256 | not recorded (see below) |
| verdict | **APPROVED**, with six tightening fixes attached |

### Independent verification of the round-1 corrections

Codex recomputed the revised numbers from `data/raw/ILINet.csv` rather than accepting them:

- Modeling-set tier counts Low 4 / Moderate 9 / High 6 / Very High 0: **confirmed.**
- 21-assessed-season counts Low 4 / Moderate 10 / High 7: **confirmed.**
- Boundary-sensitive list 2010-11, 2013-14, 2019-20, 2021-22, 2023-24: **confirmed correct under the
  stated rule**, with margins 0.1516, 0.1905 and 0.1962 for the three margin-triggered seasons, and
  2019-20 and 2021-22 flagged by variant disagreement rather than by margin.

### Critique (verbatim)

1. [Likely] The D2 continuous heatmap is defensible, but only if the national lines are visually
   subordinate. Heavy IT bands or cell-level labels would quietly reintroduce the invalid regional tier
   comparison. Fix: continuous color scale, thin labeled ticks on the colorbar only, titled "national
   reference thresholds," never "regional thresholds."
2. [Certain] The wording "available at decision time without reporting lag" overstates ILINet. Fix: say
   "ILI-only, near-real-time under the project's index-cutoff convention."
3. [Certain] B3 should name the exact response variable once more to prevent notebook drift. Fix: state
   `y = season_peak_max`, `x = cum_ili_through_W`, LOSO refit inside each W, no reuse of notebook 06
   predictions.
4. [Likely] The 9/10 validation framing would overstate agreement if used as the headline without 9/12
   beside it, because the two excluded seasons are exactly two of the three disagreements. Fix: always
   lead with "9/12 overall, 9/10 after excluding the two pre-declared non-comparable seasons."
5. [Certain] The plan calls 2008-09 and 2009-10 "the two pandemic-adjacent seasons," but 2009-10 is the
   pandemic season, not adjacent. Fix: write "pandemic and pandemic-adjacent."
6. [Likely] The claim "CDC has since updated its operational thresholds" remains uncited. Fix: change to
   "current operational values are not pinned here; Joshua should confirm them before external writeup."

### Claude's response

All six applied. Nothing rejected. Two were strengthened beyond the stated fix: #1 became a hard
requirement banning banded scales, contour bands and per-cell tier text, not only a styling preference,
because the objection is that the visual channel can re-encode a claim the labels no longer make; and #2
gained an explicit note that ILINet is itself revised, so the contrast being drawn is with strain,
hospitalization and vaccine rather than with lag in general.

**Provenance note, recorded rather than smoothed over.** The round-2 body hash was not captured before
the six fixes were applied, so this approval covers a body that no longer exists. It is recorded as
absent instead of reconstructed. Round 3 exists specifically to re-approve the final body.

---

## Round 3 — codex

| | |
|---|---|
| session | `019fc20f-2997-7200-b2eb-c3979a205c0e` (resumed, id echo verified) |
| grounding | `repo` |
| body_sha256 | `968a5fafb91b95c0bef4a0c008648738b1c0c32751cce9521129fa91189bdccc` |
| verdict | **APPROVED** |

Scope was deliberately narrow: confirm each of the six round-2 fixes was genuinely addressed rather than
reworded around, and confirm the edits introduced no new error or contradiction.

Codex confirmed all six as genuinely addressed and found no new material error, contradiction, or
unsupported established number. Repo unchanged; canary and both resumes ran under the read-only sandbox.

---

## Resolution

**Converged: APPROVED at round 3 of 5, 3 attempts of 8.**

Provenance validated by `scripts/validate-provenance.sh`, which returns **`approved-final`** (exit 0):
a repo-grounded, qualifying APPROVED round covers the current body hash
`968a5fafb91b95c0bef4a0c008648738b1c0c32751cce9521129fa91189bdccc`.

Awaiting Joshua's sign-off before any code is written. No notebook, figure or results artifact from this
plan exists yet.
