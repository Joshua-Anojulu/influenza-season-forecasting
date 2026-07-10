# 05 supplementary diagnostics (derived from the LOSO fits)

Both tables back claims made in `slides/influenza_findings_deck.html`, recorded here so every
slide number is checkable against a committed artifact.

## 1. Prophet 80% interval split

`below` means the true peak fell under the interval's lower bound (the cap-pinning pathology);
`below_gap_*` is `pi_lo - true_ili` over those seasons, in `% WEIGHTED ILI` points.

| W | n | inside | below | above | coverage_pct | pi_lo_median | pi_hi_median | below_gap_median | below_gap_min | below_gap_max |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 8 | 19 | 2 | 16 | 1 | 10.5 | 7.02 | 7.22 | 2.16 | 0.02 | 3.53 |
| 12 | 17 | 1 | 14 | 2 | 5.9 | 7.09 | 7.54 | 2.2 | 0.64 | 3.96 |
| 16 | 12 | 1 | 10 | 1 | 8.3 | 6.68 | 7.44 | 1.97 | 0.17 | 3.37 |

Undercoverage is dominated by `below`, not by narrow intervals: at W=12 the median interval is
[7.09, 7.54], 14 of 17 seasons fall under it,
2 above, and only 1 is covered.

## 2. ARIMA defined-peak survivorship (peak-week timing)

Peak-week metrics use non-plateau forecasts only. An ARIMA log-space blow-up clipped to the
historical-max cap has no locatable peak, so it is dropped. **The dropped seasons are the severe
ones**, and the retained subset is selected by the model's own success.
`climatology_pw_MAE_matched` is LOSO climatology on that same subset: the honest timing floor.

| W | n_forecast | n_defined | n_ambiguous | arima_pw_MAE | arima_pw_within1 | baselineC_pw_MAE | climatology_pw_MAE_matched | kept_mean_true_ili | dropped_mean_true_ili | edge_vs_climatology |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 8 | 19 | 9 | 10 | 30.78 | 0.0 | 10.67 | 3.44 | 4.6 | 5.73 | -27.34 |
| 12 | 17 | 4 | 13 | 25.25 | 0.0 | 9.25 | 4.25 | 4.08 | 5.21 | -21.0 |
| 16 | 12 | 7 | 5 | 3.0 | 14.3 | 6.0 | 3.29 | 4.29 | 5.75 | 0.29 |

**Reading of W=16.** ARIMA's `pw_skill` of +3.00 against baseline C is real but misleading.
Baseline C (running max through W) is a weak *timing* rule: at W=16 many seasons have not yet
peaked, so its predicted peak week is merely the last observed maximum. Against LOSO climatology
on the same 7 seasons (3.29 wk), ARIMA's edge is
0.29 weeks at n=7, and only 14.3% land
within +/-1 week, below the 36.8% floor reported in 04. The 5 seasons ARIMA drops
have mean peak ILI 5.75 versus 4.29 for the
7 it keeps: it fails on the severe seasons and is scored on the mild ones.

Conclusion: the timing result stands as reported. No model beats the timing floor in a way that
survives the selection effect.
