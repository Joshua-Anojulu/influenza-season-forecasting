# 07 template features, H1, and D3

**Status:** Preliminary descriptive analysis. **Option B assumption, pending advisor decision.**

## Availability and firewall

Index firewall: PASS across 66 (season,W) pairs. Every feature index is <= W.

This does not establish reporting availability. FluSurv-NET, NREVSS, and FluVaxView are lagged or revised,
so models containing hospitalization, strain, or vaccine features are retrospective and explanatory.

## Panel performance

| panel | W | n | conceptual_features | model_columns | MAE | RMSE | baselineC_MAE |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | 8 | 19 | 7 | 9 | 1.195 | 1.442 | 2.667 |
| B | 8 | 14 | 9 | 11 | 1.268 | 1.571 | 2.628 |
| A | 12 | 19 | 7 | 9 | 1.075 | 1.332 | 1.241 |
| B | 12 | 14 | 9 | 11 | 0.906 | 1.276 | 1.176 |
| A | 16 | 19 | 7 | 9 | 1.175 | 1.593 | 0.842 |
| B | 16 | 14 | 9 | 11 | 0.577 | 0.707 | 0.711 |

Panel A has 9 columns on 19 seasons, a strained ratio. Panel B has 11 columns on 14 seasons and is
high-variance descriptive analysis, not evidence for a feature-importance ranking.

## H1 paired comparison

| W | n | lags_MAE | cum_ili_MAE | climatology_MAE | baselineC_MAE | mean_delta_lags_minus_cum | bootstrap_95_lo | bootstrap_95_hi | lag_wins | cum_ili_wins | ties | sign_test_p |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 8 | 19 | 1.125 | 1.256 | 1.341 | 2.667 | -0.131 | -0.451 | 0.145 | 10 | 9 | 0 | 1.0 |
| 12 | 19 | 1.189 | 1.124 | 1.341 | 1.241 | 0.065 | -0.342 | 0.558 | 9 | 10 | 0 | 1.0 |
| 16 | 19 | 1.067 | 0.917 | 1.341 | 0.842 | 0.15 | -0.082 | 0.421 | 9 | 10 | 0 | 1.0 |

No descriptive support for H1: the lag model does not have lower MAE at all three W values. The paired intervals and sign tests are descriptive at n=19 with three W comparisons.

Negative paired deltas favor the lags-only model. Sign tests are exact and two-sided; bootstrap
intervals cover the paired mean delta. Three W values were inspected, so no result confirms H1.

### Per-season paired errors

| season | W | true_ili | lags_pred | cum_ili_pred | lags_abs_error | cum_ili_abs_error | delta_lags_minus_cum |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2003-04 | 8 | 7.499 | 7.5846 | 5.1122 | 0.0856 | 2.3868 | -2.3012 |
| 2004-05 | 8 | 5.029 | 4.3441 | 4.5723 | 0.6849 | 0.4567 | 0.2282 |
| 2005-06 | 8 | 3.128 | 4.5225 | 4.8852 | 1.3945 | 1.7572 | -0.3627 |
| 2006-07 | 8 | 3.456 | 4.8383 | 4.7947 | 1.3823 | 1.3387 | 0.0436 |
| 2007-08 | 8 | 5.793 | 4.4166 | 4.6639 | 1.3764 | 1.1291 | 0.2473 |
| 2010-11 | 8 | 4.523 | 4.6122 | 4.7503 | 0.0892 | 0.2273 | -0.1381 |
| 2011-12 | 8 | 2.258 | 4.2987 | 4.8636 | 2.0407 | 2.6056 | -0.5649 |
| 2012-13 | 8 | 5.012 | 4.9951 | 4.8843 | 0.0169 | 0.1277 | -0.1108 |
| 2013-14 | 8 | 4.145 | 4.5032 | 4.8071 | 0.3582 | 0.6621 | -0.3039 |
| 2014-15 | 8 | 5.455 | 4.7835 | 4.7897 | 0.6715 | 0.6653 | 0.0062 |
| 2015-16 | 8 | 3.348 | 4.6566 | 4.8938 | 1.3086 | 1.5458 | -0.2372 |
| 2016-17 | 8 | 4.834 | 4.5511 | 4.7812 | 0.2829 | 0.0528 | 0.2302 |
| 2017-18 | 8 | 7.364 | 4.9646 | 4.8706 | 2.3994 | 2.4934 | -0.094 |
| 2018-19 | 8 | 4.937 | 4.761 | 5.0699 | 0.176 | 0.1329 | 0.0432 |
| 2019-20 | 8 | 6.553 | 6.1451 | 5.3554 | 0.4079 | 1.1976 | -0.7897 |
| 2021-22 | 8 | 4.4 | 5.232 | 5.5723 | 0.832 | 1.1723 | -0.3403 |
| 2022-23 | 8 | 7.171 | 11.3374 | 10.9441 | 4.1664 | 3.7731 | 0.3934 |
| 2023-24 | 8 | 6.248 | 7.6524 | 6.3696 | 1.4044 | 0.1216 | 1.2828 |
| 2024-25 | 8 | 7.543 | 5.2526 | 5.5331 | 2.2904 | 2.0099 | 0.2805 |
| 2003-04 | 12 | 7.499 | 2.5257 | 6.1647 | 4.9733 | 1.3343 | 3.6391 |
| 2004-05 | 12 | 5.029 | 4.1646 | 4.2645 | 0.8644 | 0.7645 | 0.0999 |
| 2005-06 | 12 | 3.128 | 4.7159 | 4.7133 | 1.5879 | 1.5853 | 0.0026 |
| 2006-07 | 12 | 3.456 | 4.6439 | 4.6306 | 1.1879 | 1.1746 | 0.0132 |
| 2007-08 | 12 | 5.793 | 3.4998 | 4.2416 | 2.2932 | 1.5514 | 0.7419 |
| 2010-11 | 12 | 4.523 | 4.5698 | 4.563 | 0.0468 | 0.04 | 0.0069 |
| 2011-12 | 12 | 2.258 | 4.2426 | 4.4897 | 1.9846 | 2.2317 | -0.2471 |
| 2012-13 | 12 | 5.012 | 5.2999 | 5.004 | 0.2879 | 0.008 | 0.2799 |
| 2013-14 | 12 | 4.145 | 4.7386 | 4.6907 | 0.5936 | 0.5457 | 0.0479 |
| 2014-15 | 12 | 5.455 | 5.4936 | 5.0154 | 0.0386 | 0.4396 | -0.401 |
| 2015-16 | 12 | 3.348 | 4.3671 | 4.5793 | 1.0191 | 1.2313 | -0.2121 |
| 2016-17 | 12 | 4.834 | 4.4517 | 4.5472 | 0.3823 | 0.2868 | 0.0955 |
| 2017-18 | 12 | 7.364 | 5.2024 | 4.9782 | 2.1616 | 2.3858 | -0.2242 |
| 2018-19 | 12 | 4.937 | 4.7443 | 4.8931 | 0.1927 | 0.0439 | 0.1488 |
| 2019-20 | 12 | 6.553 | 5.7471 | 5.5739 | 0.8059 | 0.9791 | -0.1731 |
| 2021-22 | 12 | 4.4 | 5.2442 | 5.4612 | 0.8442 | 1.0612 | -0.2169 |
| 2022-23 | 12 | 7.171 | 8.3329 | 10.487 | 1.1619 | 3.316 | -2.1541 |
| 2023-24 | 12 | 6.248 | 6.64 | 6.7242 | 0.392 | 0.4762 | -0.0842 |
| 2024-25 | 12 | 7.543 | 5.7654 | 5.6373 | 1.7776 | 1.9057 | -0.1281 |
| 2003-04 | 16 | 7.499 | 4.7048 | 6.4393 | 2.7942 | 1.0597 | 1.7346 |
| 2004-05 | 16 | 5.029 | 4.1619 | 3.8584 | 0.8671 | 1.1706 | -0.3034 |
| 2005-06 | 16 | 3.128 | 4.4556 | 4.3272 | 1.3276 | 1.1992 | 0.1284 |
| 2006-07 | 16 | 3.456 | 4.2742 | 4.1784 | 0.8182 | 0.7224 | 0.0958 |
| 2007-08 | 16 | 5.793 | 3.893 | 3.744 | 1.9 | 2.049 | -0.1491 |
| 2010-11 | 16 | 4.523 | 4.5162 | 4.2914 | 0.0068 | 0.2316 | -0.2248 |
| 2011-12 | 16 | 2.258 | 3.9051 | 3.8567 | 1.6471 | 1.5987 | 0.0484 |
| 2012-13 | 16 | 5.012 | 6.4856 | 5.3587 | 1.4736 | 0.3467 | 1.1269 |
| 2013-14 | 16 | 4.145 | 5.3656 | 4.744 | 1.2206 | 0.599 | 0.6215 |
| 2014-15 | 16 | 5.455 | 5.9704 | 5.7707 | 0.5154 | 0.3157 | 0.1997 |
| 2015-16 | 16 | 3.348 | 3.9717 | 4.0175 | 0.6237 | 0.6695 | -0.0458 |
| 2016-17 | 16 | 4.834 | 4.6901 | 4.3672 | 0.1439 | 0.4668 | -0.3229 |
| 2017-18 | 16 | 7.364 | 5.6665 | 5.6814 | 1.6975 | 1.6826 | 0.0149 |
| 2018-19 | 16 | 4.937 | 4.9143 | 4.7587 | 0.0227 | 0.1783 | -0.1556 |
| 2019-20 | 16 | 6.553 | 7.5754 | 6.1961 | 1.0224 | 0.3569 | 0.6654 |
| 2021-22 | 16 | 4.4 | 5.3728 | 5.4275 | 0.9728 | 1.0275 | -0.0547 |
| 2022-23 | 16 | 7.171 | 5.1044 | 8.7343 | 2.0666 | 1.5633 | 0.5033 |
| 2023-24 | 16 | 6.248 | 6.5039 | 7.1777 | 0.2559 | 0.9297 | -0.6737 |
| 2024-25 | 16 | 7.543 | 6.6416 | 6.2853 | 0.9014 | 1.2577 | -0.3562 |

## Standardized ridge coefficients

Coefficients are directional context only. `ili_rolling4` is exactly the mean of the four lag columns.

| panel | W | feature | coefficient |
| --- | --- | --- | --- |
| A | 8 | cum_ili | -0.133804 |
| A | 8 | ili_lag_1 | 0.613561 |
| A | 8 | ili_lag_2 | 0.894733 |
| A | 8 | ili_lag_3 | 0.282414 |
| A | 8 | ili_lag_4 | -1.066109 |
| A | 8 | ili_rolling4 | 0.245565 |
| A | 8 | A(H1N1) | 0.057037 |
| A | 8 | A(H3N2) | 0.005932 |
| A | 8 | B | -0.071617 |
| A | 12 | cum_ili | 0.034639 |
| A | 12 | ili_lag_1 | 0.916447 |
| A | 12 | ili_lag_2 | -0.117034 |
| A | 12 | ili_lag_3 | -0.13317 |
| A | 12 | ili_lag_4 | 0.234514 |
| A | 12 | ili_rolling4 | 0.243563 |
| A | 12 | A(H1N1) | 0.100319 |
| A | 12 | A(H3N2) | -0.037439 |
| A | 12 | B | -0.062633 |
| A | 16 | cum_ili | 0.906467 |
| A | 16 | ili_lag_1 | 0.830916 |
| A | 16 | ili_lag_2 | -0.243212 |
| A | 16 | ili_lag_3 | -0.505984 |
| A | 16 | ili_lag_4 | 0.383203 |
| A | 16 | ili_rolling4 | 0.144361 |
| A | 16 | A(H1N1) | 0.033587 |
| A | 16 | A(H3N2) | -0.007879 |
| A | 16 | B | -0.052895 |
| B | 8 | cum_ili | 0.200728 |
| B | 8 | ili_lag_1 | 0.363634 |
| B | 8 | ili_lag_2 | 0.627592 |
| B | 8 | ili_lag_3 | 0.256692 |
| B | 8 | ili_lag_4 | -0.303721 |
| B | 8 | ili_rolling4 | 0.258413 |
| B | 8 | A(H1N1) | -0.090996 |
| B | 8 | A(H3N2) | -0.059322 |
| B | 8 | B | 0.187933 |
| B | 8 | vax | -0.312506 |
| B | 8 | hosp_rate_lag1 | -0.460154 |
| B | 12 | cum_ili | -0.013054 |
| B | 12 | ili_lag_1 | 0.847414 |
| B | 12 | ili_lag_2 | -0.003276 |
| B | 12 | ili_lag_3 | -0.123068 |
| B | 12 | ili_lag_4 | 0.168067 |
| B | 12 | ili_rolling4 | 0.231246 |
| B | 12 | A(H1N1) | -0.116053 |
| B | 12 | A(H3N2) | -0.036425 |
| B | 12 | B | 0.185961 |
| B | 12 | vax | -0.276913 |
| B | 12 | hosp_rate_lag1 | 0.248153 |
| B | 16 | cum_ili | 0.980843 |
| B | 16 | ili_lag_1 | 0.498364 |
| B | 16 | ili_lag_2 | 0.075657 |
| B | 16 | ili_lag_3 | -0.085006 |
| B | 16 | ili_lag_4 | -0.272831 |
| B | 16 | ili_rolling4 | 0.037972 |
| B | 16 | A(H1N1) | -0.036672 |
| B | 16 | A(H3N2) | 0.042188 |
| B | 16 | B | -0.014164 |
| B | 16 | vax | 0.168112 |
| B | 16 | hosp_rate_lag1 | 0.449143 |

## Grouped block ablations

Positive delta_MAE means removing the block worsened LOSO error. Grouped ablations are the primary
importance measure because individual lag coefficients and drop-one-lag deltas are not interpretable.

| panel | W | group | full_MAE | ablated_MAE | delta_MAE |
| --- | --- | --- | --- | --- | --- |
| A | 8 | cum_ili | 1.195 | 1.211 | 0.016 |
| A | 8 | recent_ILI_lag_block | 1.195 | 1.204 | 0.009 |
| A | 8 | rolling_summary | 1.195 | 1.172 | -0.023 |
| A | 8 | recent_ILI_all | 1.195 | 1.279 | 0.084 |
| A | 8 | strain_block | 1.195 | 1.32 | 0.125 |
| A | 12 | cum_ili | 1.075 | 1.008 | -0.067 |
| A | 12 | recent_ILI_lag_block | 1.075 | 1.066 | -0.009 |
| A | 12 | rolling_summary | 1.075 | 1.045 | -0.03 |
| A | 12 | recent_ILI_all | 1.075 | 1.19 | 0.116 |
| A | 12 | strain_block | 1.075 | 1.101 | 0.027 |
| A | 16 | cum_ili | 1.175 | 1.092 | -0.082 |
| A | 16 | recent_ILI_lag_block | 1.175 | 0.859 | -0.316 |
| A | 16 | rolling_summary | 1.175 | 1.171 | -0.004 |
| A | 16 | recent_ILI_all | 1.175 | 0.981 | -0.194 |
| A | 16 | strain_block | 1.175 | 0.966 | -0.209 |
| B | 8 | cum_ili | 1.268 | 1.27 | 0.002 |
| B | 8 | recent_ILI_lag_block | 1.268 | 1.244 | -0.024 |
| B | 8 | rolling_summary | 1.268 | 1.263 | -0.005 |
| B | 8 | recent_ILI_all | 1.268 | 1.234 | -0.034 |
| B | 8 | strain_block | 1.268 | 1.238 | -0.03 |
| B | 8 | vax | 1.268 | 1.252 | -0.016 |
| B | 8 | hosp_rate_lag1 | 1.268 | 1.174 | -0.094 |
| B | 12 | cum_ili | 0.906 | 1.258 | 0.353 |
| B | 12 | recent_ILI_lag_block | 0.906 | 1.178 | 0.272 |
| B | 12 | rolling_summary | 0.906 | 1.091 | 0.185 |
| B | 12 | recent_ILI_all | 0.906 | 1.272 | 0.366 |
| B | 12 | strain_block | 0.906 | 1.124 | 0.218 |
| B | 12 | vax | 0.906 | 0.996 | 0.09 |
| B | 12 | hosp_rate_lag1 | 0.906 | 1.212 | 0.306 |
| B | 16 | cum_ili | 0.577 | 0.779 | 0.202 |
| B | 16 | recent_ILI_lag_block | 0.577 | 0.684 | 0.107 |
| B | 16 | rolling_summary | 0.577 | 0.574 | -0.003 |
| B | 16 | recent_ILI_all | 0.577 | 0.532 | -0.045 |
| B | 16 | strain_block | 0.577 | 0.5 | -0.078 |
| B | 16 | vax | 0.577 | 0.577 | -0.0 |
| B | 16 | hosp_rate_lag1 | 0.577 | 0.362 | -0.215 |

At W=12 in Panel A, the largest positive block-ablation delta is recent_ILI_all at +0.116 MAE. This is descriptive, not a feature ranking.
At W=12 in Panel B, the largest positive block-ablation delta is recent_ILI_all at +0.366 MAE. This is descriptive, not a feature ranking.

## Excluded seasons under Panel A

Excluded-from-training structural-break stress test, not a prospective forecast. Results are per season, never pooled.

| season | mechanism | W | model | selected_lambda | true_ili | pred_ili | error | abs_error | label |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2008-09 | pandemic-adjacent: spring 2009 H1N1 emergence wave | 8 | Panel A ridge | 0.1 | 4.456 | 4.033 | -0.423 | 0.423 | excluded-from-training structural-break stress test, not prospective |
| 2008-09 | pandemic-adjacent: spring 2009 H1N1 emergence wave | 12 | Panel A ridge | 100.0 | 4.456 | 4.4933 | 0.0373 | 0.0373 | excluded-from-training structural-break stress test, not prospective |
| 2008-09 | pandemic-adjacent: spring 2009 H1N1 emergence wave | 16 | Panel A ridge | 1.0 | 4.456 | 3.4113 | -1.0447 | 1.0447 | excluded-from-training structural-break stress test, not prospective |
| 2009-10 | 2009 H1N1 pandemic season | 8 | Panel A ridge | 0.1 | 7.454 | -3.8046 | -11.2586 | 11.2586 | excluded-from-training structural-break stress test, not prospective |
| 2009-10 | 2009 H1N1 pandemic season | 12 | Panel A ridge | 100.0 | 7.454 | 5.2557 | -2.1983 | 2.1983 | excluded-from-training structural-break stress test, not prospective |
| 2009-10 | 2009 H1N1 pandemic season | 16 | Panel A ridge | 1.0 | 7.454 | 6.0277 | -1.4263 | 1.4263 | excluded-from-training structural-break stress test, not prospective |
| 2020-21 | near-total flu absence under COVID NPIs | 8 | Panel A ridge | 0.1 | 2.239 | 4.4175 | 2.1785 | 2.1785 | excluded-from-training structural-break stress test, not prospective |
| 2020-21 | near-total flu absence under COVID NPIs | 12 | Panel A ridge | 100.0 | 2.239 | 4.5084 | 2.2694 | 2.2694 | excluded-from-training structural-break stress test, not prospective |
| 2020-21 | near-total flu absence under COVID NPIs | 16 | Panel A ridge | 1.0 | 2.239 | 2.6721 | 0.4331 | 0.4331 | excluded-from-training structural-break stress test, not prospective |

## Deliberate imputation divergence from notebook 06

Notebook 07 imputes every missing column using its own training-fold mean and recomputes those means inside
each inner-CV split. Notebook 06's simpler last-column, outer-fold path remains protected and unchanged.

## Template deviations

- `season_week` has zero variance as a season-level predictor at fixed W. It remains the within-season time axis `sw`.
- ARIMA has no prediction intervals in this implementation. D1 and interval coverage are Prophet-only.
