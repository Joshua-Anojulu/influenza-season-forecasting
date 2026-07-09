# Start here — influenza season forecasting pipeline

A short guide for running and replicating this pipeline. The goal is to follow the workflow
notebook by notebook and reproduce the results. Everything is national-scope US influenza
surveillance; the target is the timing and severity of each season's ILI peak.

## 1. Set up the environment

- Python 3.14.
- From the project root: `pip install -r requirements.txt`.

## 2. Get the data (public CDC files)

The raw files are not included. Download these six from the CDC and place them in `data/raw/`
with exactly these names (notebook 01 documents every file and its loading quirks):

| File | Source |
|------|--------|
| `ILINet.csv` | CDC FluView Interactive (ILINet, national) |
| `ICL_NREVSS_Combined_prior_to_2015_16.csv` | CDC FluView Interactive (NREVSS) |
| `ICL_NREVSS_Public_Health_Labs.csv` | CDC FluView Interactive (NREVSS) |
| `ICL_NREVSS_Clinical_Labs.csv` | CDC FluView Interactive (NREVSS) |
| `FluSurveillance_Custom_Download_Data.csv` | CDC FluSurv-NET |
| `FluVaxView.csv` | CDC FluVaxView |

CDC FluView Interactive: https://gis.cdc.gov/grasp/fluview/fluportaldashboard.html

If your download has a different filename (the CDC exports use long default names), rename it to
match the table. Notebook 01 checks that all six files are present and prints what is missing.

## 3. Run the notebooks in order

Run **01 through 06 in order** from the project root, either way:

- **Jupyter:** launch `jupyter notebook` (or JupyterLab) from the project root and run each notebook top to bottom.
- **Headless:** `jupyter nbconvert --to notebook --execute notebooks/01_data_inventory.ipynb` (repeat for 02...06).

Run from the project root so the `data/raw/` paths resolve. The notebooks are committed without
outputs and each one rebuilds the cleaned data from notebook 02's logic, so they are deterministic
and can be run independently once the data is in place.

## 4. What each notebook does

1. `01_data_inventory` — load and audit every source; document the loading quirks.
2. `02_cleaning` — align to MMWR seasons; build the `peak_week` / `peak_ili_pct` targets on a
   3-week centered smoother (removes a year-end reporting artifact); stitch the NREVSS strain series.
3. `03_eda` — season trajectories, peak distributions, strain timeline, missingness map.
4. `04_baselines` — naive floors (climatology, persistence, within-season running max) that every model must beat.
5. `05_forecasting` — ARIMA and Prophet under a strict leakage firewall; interval calibration.
6. `06_regression_and_curve` — a univariate severity regression, an explanatory ridge, and a Gaussian curve fit.

Outputs land in `results/` (metric tables, markdown + JSON) and `figures/` (PNG). If those folders
are included here, use them as an answer key to check your run against.

## 5. Ground rules to keep in mind (this is the point of the exercise)

- **Leakage firewall.** Every model feature uses only data through the decision week W. Nothing after
  W may touch a season's forecast. Cumulative-season features are the classic trap; check each one.
- **Validation is leave-one-season-out** over 19 non-pandemic seasons (2008-09, 2009-10, 2020-21 are held out).
- **Lead-time matching.** A model at decision week W is compared only to the baseline at the same W.
- **Characterization first.** Honest negative results (a model failing to beat the floor) are valid
  and reportable, not failures to hide. The aim is a clear description of what the data can and cannot support.
