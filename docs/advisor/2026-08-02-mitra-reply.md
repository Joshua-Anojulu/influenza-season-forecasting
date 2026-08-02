# Dr. Mitra's reply, received 2026-08-02

Response to Joshua's 2026-07-21 follow-up email, which asked three decision questions
(D2 heatmap scope, severity classification method, feature-panel season range).
He was traveling internationally, which is why the reply took twelve days.

Transcribed verbatim below. Interpretation and the resulting project decisions live in
`CLAUDE.md` under "Confirmed by Dr. Mitra (2026-08-02)"; this file is the source text only.

---

> Joshua,
>
> Sorry for the late response I am traveling internationally.
>
> Quick answers on all three decision points.
>
> Heatmap: go with option 2. Pull regional ILI for the heatmap only - it satisfies the template
> requirement and costs you little, since ILINet already provides it. Keep the models national.
>
> Severity classification: use CDC's published intensity thresholds. Anchoring to the literature
> is cleaner for a student project and more defensible than self-defined quantiles.
>
> Feature work: keep 2003 onward as your core and use the 2009+ enrichment where it exists, as you
> have planned. Do not restrict to 2009+ - you would lose too many seasons.
>
> The pandemic season out-of-sample documentation is a good instinct. Keep it.
>
> Best regards,

---

## What this does and does not settle

Settled: D2 scope (regional ILI as figure input only), the severity tier method (CDC published
intensity thresholds, not self-defined quantiles), Option B (2003+ core, 2009+ enrichment), and
the pandemic out-of-sample framing.

Not settled, because it was not asked: which decision week W to headline. That stays open.

Not granted, and not to be inferred: he did not approve any specific numeric tier boundaries, and
he did not approve regional modeling. "Keep the models national" is an explicit restatement of the
2026-07-08 scope ruling, not a relaxation of it.
