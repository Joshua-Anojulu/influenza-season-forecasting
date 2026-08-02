# 08 regional season severity (D2)

Regional ILI is a **figure input only**. It never enters any model: Dr. Mitra's 2026-08-02 ruling was to pull regional ILI for the heatmap only and keep the models national.

`figures/12_D2_severity_heatmap.png` is a **continuous** heatmap of regional season peak
% WEIGHTED ILI. The national IT50/IT90/IT98 appear as colorbar reference ticks only.
Cells are not coloured by, or labelled with, a regional tier.

**Why not tiered.** CDC publishes no region-specific intensity thresholds. Regional ILI has region-specific baselines, so calling a region-season 'High' off a national threshold would be a label the thresholds do not license.

Seasons marked with `*` and a dotted column are excluded from modeling as structural breaks: 2008-09, 2009-10, 2020-21.

## National-threshold exceedance, diagnostic only

DIAGNOSTIC ONLY, never a definition. Counts how often each region's season peak exceeds each NATIONAL threshold, to quantify the regional-baseline mismatch that is the reason this figure is not tiered.

| region | median_peak | n_above_IT50 | n_above_IT90 | n_above_IT98 | n_seasons |
| --- | --- | --- | --- | --- | --- |
| Region 1 | 3.896 | 7 | 5 | 2 | 21 |
| Region 2 | 5.51 | 14 | 8 | 4 | 21 |
| Region 3 | 5.642 | 17 | 8 | 2 | 21 |
| Region 4 | 6.032 | 16 | 8 | 4 | 21 |
| Region 5 | 4.639 | 11 | 4 | 2 | 21 |
| Region 6 | 9.455 | 20 | 17 | 14 | 21 |
| Region 7 | 5.705 | 15 | 8 | 3 | 21 |
| Region 8 | 4.138 | 7 | 5 | 1 | 21 |
| Region 9 | 5.008 | 15 | 7 | 2 | 21 |
| Region 10 | 4.317 | 10 | 8 | 3 | 21 |
