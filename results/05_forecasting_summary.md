# 05 forecasting (LOSO, within-season firewall)

## Metrics

| model | W | n_forecast | n_detection | n_pw_defined | n_pw_ambiguous | pw_MAE_fc | pw_within1_fc | pw_within1_fc_exfrag | ili_MAE_fc | ili_RMSE_fc | ili_MAE_all19 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ARIMA | 8 | 19 | 0 | 9 | 10 | 30.78 | 0.0 | 0.0 | 1.56 | 1.873 | 1.56 |
| ARIMA | 12 | 17 | 2 | 4 | 13 | 25.25 | 0.0 | 0.0 | 2.178 | 2.49 | 2.023 |
| ARIMA | 16 | 12 | 7 | 7 | 5 | 3.0 | 14.3 | 20.0 | 1.275 | 1.461 | 1.689 |
| Prophet | 8 | 19 | 0 | 19 | 0 | 34.79 | 0.0 | 0.0 | 1.837 | 2.141 | 1.837 |
| Prophet | 12 | 17 | 2 | 17 | 0 | 34.0 | 0.0 | 0.0 | 2.145 | 2.468 | 1.942 |
| Prophet | 16 | 12 | 7 | 12 | 0 | 32.08 | 0.0 | 0.0 | 1.906 | 2.226 | 1.869 |

## Skill vs baseline C (forecast subset)

| model | W | n | n_pw | pw_model | pw_baselineC | pw_skill | ili_model | ili_baselineC | ili_skill |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ARIMA | 8 | 19 | 9 | 30.78 | 10.67 | -20.11 | 1.56 | 2.667 | 1.107 |
| ARIMA | 12 | 17 | 4 | 25.25 | 9.25 | -16.0 | 2.178 | 1.371 | -0.807 |
| ARIMA | 16 | 12 | 7 | 3.0 | 6.0 | 3.0 | 1.275 | 1.046 | -0.229 |
| Prophet | 8 | 19 | 19 | 34.79 | 9.21 | -25.58 | 1.837 | 2.667 | 0.83 |
| Prophet | 12 | 17 | 17 | 34.0 | 6.0 | -28.0 | 2.145 | 1.371 | -0.774 |
| Prophet | 16 | 12 | 12 | 32.08 | 5.67 | -26.41 | 1.906 | 1.046 | -0.86 |

## Prophet 80% calibration

| W | n | nominal | empirical_coverage | verdict |
| --- | --- | --- | --- | --- |
| 8 | 19 | 80.0 | 10.5 | over-confident |
| 12 | 17 | 80.0 | 5.9 | over-confident |
| 16 | 12 | 80.0 | 8.3 | over-confident |
