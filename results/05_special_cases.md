# 05 excluded-season structural-break stress test

These seasons were excluded from all training and headline metrics. This is not a prospective forecast:
the cap and training reference use all 19 modeled seasons, including chronologically later seasons.
Errors are reported by season and model, never pooled across the three different failure mechanisms.

## 2008-09

Failure mechanism: pandemic-adjacent: spring 2009 H1N1 emergence wave.

| model | W | status | peak_ambiguous | true_peak_week | pred_peak_week | peak_week_abs_error | true_ili | pred_peak_ili | peak_ili_abs_error | pi_lo | pi_hi |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ARIMA | 8 | forecast | True | 38 | 7 |  | 4.456 | 1.218 | 3.238 |  |  |
| ARIMA | 12 | forecast | False | 38 | 1 | 37.0 | 4.456 | 1.449 | 3.007 |  |  |
| ARIMA | 16 | forecast | False | 38 | 4 | 34.0 | 4.456 | 1.876 | 2.58 |  |  |
| Prophet | 8 | forecast | False | 38 | 39 | 1.0 | 4.456 | 3.342 | 1.114 | 3.29 | 3.39 |
| Prophet | 12 | forecast | False | 38 | 39 | 1.0 | 4.456 | 3.973 | 0.483 | 3.908 | 4.042 |
| Prophet | 16 | forecast | False | 38 | 39 | 1.0 | 4.456 | 5.479 | 1.023 | 5.289 | 5.662 |

## 2009-10

Failure mechanism: 2009 H1N1 pandemic season.

| model | W | status | peak_ambiguous | true_peak_week | pred_peak_week | peak_week_abs_error | true_ili | pred_peak_ili | peak_ili_abs_error | pi_lo | pi_hi |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ARIMA | 8 | peak_already_observed_at_W | False | 42 | 48 | 6.0 | 7.454 | 3.211 | 4.243 |  |  |
| ARIMA | 12 | peak_already_observed_at_W | False | 42 | 39 | 49.0 | 7.454 | 3.341 | 4.113 |  |  |
| ARIMA | 16 | peak_already_observed_at_W | True | 42 | 39 |  | 7.454 | 1.893 | 5.561 |  |  |
| Prophet | 8 | peak_already_observed_at_W | False | 42 | 48 | 6.0 | 7.454 | 2.097 | 5.357 | 1.276 | 2.869 |
| Prophet | 12 | peak_already_observed_at_W | False | 42 | 52 | 10.0 | 7.454 | 1.127 | 6.327 | 0.143 | 2.103 |
| Prophet | 16 | peak_already_observed_at_W | False | 42 | 4 | 14.0 | 7.454 | 0.765 | 6.689 | 0.0 | 1.839 |

## 2020-21

Failure mechanism: near-total flu absence under COVID NPIs.

| model | W | status | peak_ambiguous | true_peak_week | pred_peak_week | peak_week_abs_error | true_ili | pred_peak_ili | peak_ili_abs_error | pi_lo | pi_hi |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ARIMA | 8 | forecast | False | 35 | 39 | 4.0 | 2.239 | 5.51 | 3.271 |  |  |
| ARIMA | 12 | forecast | False | 35 | 52 | 35.0 | 2.239 | 1.502 | 0.737 |  |  |
| ARIMA | 16 | forecast | False | 35 | 4 | 31.0 | 2.239 | 1.226 | 1.013 |  |  |
| Prophet | 8 | forecast | False | 35 | 39 | 4.0 | 2.239 | 6.196 | 3.957 | 6.126 | 6.273 |
| Prophet | 12 | forecast | False | 35 | 39 | 4.0 | 2.239 | 4.523 | 2.284 | 4.407 | 4.638 |
| Prophet | 16 | forecast | False | 35 | 39 | 4.0 | 2.239 | 2.616 | 0.377 | 2.439 | 2.805 |
