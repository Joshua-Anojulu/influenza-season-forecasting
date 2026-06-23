# Influenza Season Forecasting

Forecasting the timing and severity of US influenza season peaks from public CDC and WHO surveillance data.

> **Status:** Early development — data acquisition and pipeline setup. No results yet.

## Overview

This project investigates whether historical US influenza surveillance data can forecast two characteristics of a flu season:

1. **Peak timing:** the MMWR week at which influenza-like illness (ILI) activity peaks.
2. **Peak severity:** the ILI% value reached at that peak.

Both are framed as **within-season forecasting** tasks. Standing at a fixed week of an ongoing season, and using only data available up to that week, the model forecasts the remaining trajectory, from which the predicted peak week and peak height are read off. Repeating this at different decision weeks yields an accuracy-versus-lead-time analysis.

## Data Sources

All data is publicly available from the CDC and WHO. This project is not affiliated with either organization.

| Source | Contents | Resolution |
|--------|----------|------------|
| CDC FluView / ILINet | Outpatient influenza-like illness (ILI%) | Weekly |
| CDC FluSurv-NET | Laboratory-confirmed influenza hospitalization rates | Weekly |
| WHO FluNet | Virological data / circulating strain composition | Weekly |
| CDC FluVaxView | Seasonal vaccine coverage | Annual |

## Methodology (planned)

- **Baselines:** historical-median peak week; prior-season / historical-mean peak ILI%. All models are benchmarked against these.
- **Models:** ARIMA and Prophet within-season forecasts; optional Random Forest classifier for Low / Moderate / High severity tiers.
- **Validation:** leave-one-season-out (walk-forward) evaluation.
- **Metrics:** peak-week error (weeks); peak-ILI MAE and RMSE; 80% prediction-interval coverage (calibration).
- **Outlier handling:** the 2009 H1N1 and 2020–21 COVID-disrupted seasons are held out as labeled special cases rather than included in training.

## Repository Structure

```
.
├── notebooks/
│   ├── 01_data_inventory.ipynb   # data acquisition + structure/missingness audit
│   ├── 02_cleaning.ipynb         # season alignment, interpolation, normalization
│   ├── 03_eda.ipynb              # season trajectories, peak distributions
│   ├── 04_baselines.ipynb        # naive benchmarks
│   └── 05_forecasting.ipynb      # ARIMA / Prophet / RF + validation harness
├── data/                         # raw + processed data (gitignored)
├── src/                          # shared utilities
└── README.md
```

## Notes and Limitations

The training signal is fundamentally limited by sample size: roughly 20 complete flu seasons. Pandemic seasons introduce structural breaks. These constraints bound how strong any honest result can be, and are addressed explicitly in the analysis rather than worked around.

## Background

- Reich NG et al. (2019). A collaborative multiyear, multimodel assessment of seasonal influenza forecasting in the United States. *PNAS* 116(8), 3146–3154.
- Biggerstaff M et al. (2018). Results from the second year of a collaborative effort to forecast influenza seasons in the United States. *Epidemics* 24, 26–33.
- Viboud C & Vespignani A. (2019). The future of influenza forecasts. *PNAS* 116(8), 2802–2804.

## Acknowledgment

Research project advised by Dr. Subhro Mitra.
