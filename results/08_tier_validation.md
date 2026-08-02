# 08 severity tiers: CDC-anchored definition and validation

**ILI-only approximation of CDC's severity framework**, using CDC's published ILI intensity
thresholds. This is NOT CDC's official season severity classification, which is a 2-of-3
indicator vote over ILI, hospitalization rate, and pneumonia-and-influenza mortality.

Thresholds: IT50=4.4, IT90=6.6, IT98=8.6.
Source: Biggerstaff M, et al. Am J Epidemiol. 2018;187(5):1040-1050. doi:10.1093/aje/kwx334
Reference seasons: 2003-04 through 2014-15, excluding the 2009 pandemic

Vintage: 2018 published values. CDC's current operational values are not pinned here and are not claimed to differ; using the published historical values is a deliberate methodological choice, made for citability.

Classification statistic: season peak weekly raw % WEIGHTED ILI. The geometric mean of the
three highest weeks is how the thresholds were ESTIMATED, not how a season is classified.

## Tier counts

- Modeling set (19): {'Moderate': 9, 'High': 6, 'Low': 4}
- Assessed (21, excludes 2020-21): {'Moderate': 10, 'High': 7, 'Low': 4}
- Very High: 0 seasons in 22. No season reaches IT98=8.6.

2020-21 carries no CDC-anchored tier. CDC did not assess it.

## Validation against CDC's published classifications

**9/12 overall**, and **9/10 after excluding the two
pre-declared pandemic and pandemic-adjacent seasons as non-comparable.** Both are reported
together: quoting only the second would overstate agreement, because the two excluded
seasons are two of the three disagreements.

| season | cdc_published | ili_only_tier | season_peak_max | agrees | comparable |
| --- | --- | --- | --- | --- | --- |
| 2003-04 | High | High | 7.628 | True | True |
| 2004-05 | Moderate | Moderate | 5.442 | True | True |
| 2005-06 | Low | Low | 3.282 | True | True |
| 2006-07 | Low | Low | 3.579 | True | True |
| 2007-08 | Moderate | Moderate | 5.983 | True | True |
| 2008-09 | Low | Moderate | 4.886 | False | False |
| 2009-10 | Moderate | High | 7.715 | False | False |
| 2010-11 | Moderate | Moderate | 4.552 | True | True |
| 2011-12 | Low | Low | 2.389 | True | True |
| 2012-13 | Moderate | Moderate | 6.061 | True | True |
| 2013-14 | Moderate | Moderate | 4.591 | True | True |
| 2014-15 | High | Moderate | 5.982 | False | True |

## Season table

| season | season_peak_max | peak_ili_pct | tier_assigned | tier_smoothed | tier_gm3 | tier_margin | tier_boundary_sensitive | excluded_from_modeling |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2003-04 | 7.628 | 7.499 | High | High | High | 0.972 | False | False |
| 2004-05 | 5.442 | 5.029 | Moderate | Moderate | Moderate | 1.042 | False | False |
| 2005-06 | 3.282 | 3.128 | Low | Low | Low | 1.118 | False | False |
| 2006-07 | 3.579 | 3.456 | Low | Low | Low | 0.821 | False | False |
| 2007-08 | 5.983 | 5.793 | Moderate | Moderate | Moderate | 0.617 | False | False |
| 2008-09 | 4.886 | 4.456 | Moderate | Moderate | Moderate | 0.486 | False | True |
| 2009-10 | 7.715 | 7.454 | High | High | High | 0.885 | False | True |
| 2010-11 | 4.552 | 4.523 | Moderate | Moderate | Moderate | 0.152 | True | False |
| 2011-12 | 2.389 | 2.258 | Low | Low | Low | 2.011 | False | False |
| 2012-13 | 6.061 | 5.012 | Moderate | Moderate | Moderate | 0.539 | False | False |
| 2013-14 | 4.591 | 4.145 | Moderate | Low | Low | 0.191 | True | False |
| 2014-15 | 5.982 | 5.455 | Moderate | Moderate | Moderate | 0.618 | False | False |
| 2015-16 | 3.56 | 3.348 | Low | Low | Low | 0.84 | False | False |
| 2016-17 | 5.063 | 4.834 | Moderate | Moderate | Moderate | 0.663 | False | False |
| 2017-18 | 7.529 | 7.364 | High | High | High | 0.929 | False | False |
| 2018-19 | 5.042 | 4.937 | Moderate | Moderate | Moderate | 0.642 | False | False |
| 2019-20 | 7.062 | 6.553 | High | Moderate | High | 0.462 | True | False |
| 2020-21 | 2.266 | 2.239 | Not assessed | Low | Low | 2.134 | False | True |
| 2021-22 | 4.894 | 4.4 | Moderate | Moderate | Low | 0.494 | True | False |
| 2022-23 | 7.425 | 7.171 | High | High | High | 0.825 | False | False |
| 2023-24 | 6.796 | 6.248 | High | Moderate | Moderate | 0.196 | True | False |
| 2024-25 | 7.844 | 7.543 | High | High | High | 0.756 | False | False |

## Raw versus smoothed disagreements

In every season where the two statistics disagree, the unsmoothed CDC statistic tiers HIGHER than the project's smoothed statistic. That is the direction the notebook 02 holiday-artifact finding predicts, and it is why tier_smoothed is a mandatory sensitivity rather than an optional one.
Seasons: 2013-14, 2019-20, 2023-24.
