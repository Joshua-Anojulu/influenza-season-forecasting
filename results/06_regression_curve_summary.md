# 06 regression and curve models (LOSO, through-W features)

## Univariate real-time severity forecast

| W | uni_MAE | uni_RMSE | baselineC_MAE | climatology_MAE |
| --- | --- | --- | --- | --- |
| 8 | 1.256 | 1.614 | 2.667 | 1.341 |
| 12 | 1.124 | 1.421 | 1.241 | 1.341 |
| 16 | 0.917 | 1.062 | 0.842 | 1.341 |

## Explanatory ridge (retrospective; ili+strain on 19, +vax on 2009+ 14)

| W | n19 | ridge_ili_strain_MAE | n14 | ridge_ili_strain_vax_MAE | univariate_MAE |
| --- | --- | --- | --- | --- | --- |
| 8 | 19 | 1.279 | 14 | 1.459 | 1.256 |
| 12 | 19 | 1.19 | 14 | 1.258 | 1.124 |
| 16 | 19 | 0.981 | 14 | 0.969 | 0.917 |

## Gaussian curve fit

| W | n_forecast | n_pw_defined | n_at_bound | sev_MAE | pw_MAE | pw_within1 |
| --- | --- | --- | --- | --- | --- | --- |
| 8 | 19 | 5 | 14 | 2.324 | 4.8 | 20.0 |
| 12 | 18 | 1 | 17 | 2.586 | 7.0 | 0.0 |
| 16 | 5 | 0 | 18 | 1.47 |  |  |

## Severity MAE by model and W

| W | climatology | baselineC | univariate | ridge_ili_strain | ridge_plus_vax_n14 | gaussian_fcsubset |
| --- | --- | --- | --- | --- | --- | --- |
| 8 | 1.341 | 2.667 | 1.256 | 1.279 | 1.459 | 2.324 |
| 12 | 1.341 | 1.241 | 1.124 | 1.19 | 1.258 | 2.586 |
| 16 | 1.341 | 0.842 | 0.917 | 0.981 | 0.969 | 1.47 |

## Severity skill vs baseline C

| W | baselineC | univariate_skill | ridge_ili_strain_skill |
| --- | --- | --- | --- |
| 8 | 2.667 | 1.411 | 1.388 |
| 12 | 1.241 | 0.117 | 0.051 |
| 16 | 0.842 | -0.075 | -0.139 |
