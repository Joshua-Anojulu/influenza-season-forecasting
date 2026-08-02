# 08 severity classifier (3-class, LOSO, n=19)

**No demonstrated advantage over B3, the regression-then-threshold baseline.**

- Paired bootstrap intervals for RF minus B3 include zero at W=12 ([-0.1341, 0.3843]) and W=16 ([-0.2063, 0.336]). Only W=8 excludes zero, and only marginally (+0.003 lower bound).
- The pattern runs backwards. The apparent advantage is largest at W=8, the earliest and least informative decision week, and shrinks to +0.06 by W=16. A real skill advantage should not decay as more of the season becomes observable.
- Macro-F1 is itself non-monotonic in W (RF 0.6342 at W=8 versus 0.4555 at W=12). More information making the model worse is a noise signature, not a skill signature.
- Three W values were inspected. One marginal exclusion out of three comparisons is what chance produces; it is descriptive, never confirmation.

Class counts: {'Low': 4, 'Moderate': 9, 'High': 6}. Very High dropped: empty by observation.

n=19 with 3 classes and 6 feature columns. The Low class has 4 members, so some LOSO folds train on 3 examples of it. Overfitting is expected.

## Primary, real-time (ILI-only) macro-F1, pooled over 19 LOSO folds

| W | RF | B0 | B1 | B2 | B3 | RF_minus_B3 | CI95 | includes_zero |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 8 | 0.6342 | 0.2143 | 0.2907 | 0.2143 | 0.326 | 0.3082 | [0.0032, 0.577] | False |
| 12 | 0.4555 | 0.2143 | 0.2907 | 0.2143 | 0.326 | 0.1295 | [-0.1341, 0.3843] | True |
| 16 | 0.6777 | 0.2143 | 0.2907 | 0.2143 | 0.6222 | 0.0555 | [-0.2063, 0.336] | True |

Macro-F1 is pre-registered as descriptive only. Read the confusion matrix first.

### W=8 confusion matrix (rows true, cols predicted: ['Low', 'Moderate', 'High'])

| true | Low | Moderate | High |
| --- | --- | --- | --- |
| Low | 3 | 1 | 0 |
| Moderate | 3 | 4 | 2 |
| High | 0 | 1 | 5 |

accuracy 0.6316, per-class F1 {'Low': 0.6, 'Moderate': 0.5333, 'High': 0.7692}

### W=12 confusion matrix (rows true, cols predicted: ['Low', 'Moderate', 'High'])

| true | Low | Moderate | High |
| --- | --- | --- | --- |
| Low | 1 | 3 | 0 |
| Moderate | 4 | 3 | 2 |
| High | 0 | 1 | 5 |

accuracy 0.4737, per-class F1 {'Low': 0.2222, 'Moderate': 0.375, 'High': 0.7692}

### W=16 confusion matrix (rows true, cols predicted: ['Low', 'Moderate', 'High'])

| true | Low | Moderate | High |
| --- | --- | --- | --- |
| Low | 3 | 1 | 0 |
| Moderate | 2 | 6 | 1 |
| High | 1 | 1 | 4 |

accuracy 0.6842, per-class F1 {'Low': 0.6, 'Moderate': 0.7059, 'High': 0.7273}

## Secondary, retrospective (+dominant strain)

Strain is reporting-lagged, so this panel is explanatory and is never a forecast.

- W=8: RF 0.6342, paired vs B3 +0.3082 [0.0032, 0.577], includes zero False
- W=12: RF 0.5231, paired vs B3 +0.1971 [-0.093, 0.4673], includes zero True
- W=16: RF 0.6777, paired vs B3 +0.0555 [-0.2063, 0.336], includes zero True

## Firewall

PASS across 66 (season,W) pairs; every feature index <= W

the LABEL derives from the full-season peak by design, as in 05 and 06 where the target is also the realized season peak; the firewall governs features, not the target

## D2

NOT BUILT. ILINet_regional.csv not present in data/raw/; Step 0 of the plan is a manual FluView download with region type 'HHS Regions'. D2 is blocked, not skipped, and nothing about regional severity is asserted here.
