# Results

## Primary Stage-1 Attack Detector

The final primary task is:

**Benign vs Attack**

### Validation vs Locked Test

| Metric | Validation | Locked Test |
|---|---:|---:|
| Accuracy | 0.9590 | 0.6958 |
| Balanced Accuracy | 0.9612 | 0.7915 |
| Attack Precision | 0.9864 | 1.0000 |
| Attack Recall | 0.9561 | 0.5830 |
| Attack F1 | 0.9710 | 0.7366 |
| ROC-AUC | 0.9886 | 0.9369 |
| PR-AUC | 0.9961 | 0.9777 |

The Locked Test exposed a substantial temporal generalization gap.

Despite lower threshold-level recall, ranking ability remained stronger:

- ROC-AUC = **0.9369**
- PR-AUC = **0.9777**

![Validation vs Locked Test](results/figures/validation_vs_locked_test.png)

## Locked Test Confusion Matrix

| index       |   Pred Benign |   Pred Attack |
|:------------|--------------:|--------------:|
| True Benign |            96 |             0 |
| True Attack |           108 |           151 |

Interpretation:

- 96 benign windows correctly classified
- 0 false-positive attacks
- 151 attacks correctly detected
- 108 attacks missed

![Confusion Matrix](results/figures/locked_test_confusion_matrix.png)

The detector therefore became highly conservative at the frozen
0.5 operating threshold.

## Calibration

Temperature scaling changed zero class predictions.

| Metric | Raw | Temperature Scaled |
|---|---:|---:|
| Log Loss | 0.7889 | 0.5741 |
| Brier | 0.2334 | 0.1978 |
| Equal-width ECE | 0.2173 | 0.1628 |
| Adaptive ECE | 0.2175 | 0.1662 |

All reported calibration-quality metrics improved on the Locked Test.

![Calibration](results/figures/locked_test_calibration_generalization.png)

## Risk-Aware Routing

Frozen test routing:

| Metric | Value |
|---|---:|
| Test windows | 355 |
| Base errors | 108 |
| Verification windows | 160 |
| Verification fraction | 45.07% |
| Errors captured | 94 |
| Error capture | 87.04% |
| Automated windows | 195 |
| Automated fraction | 54.93% |
| Automated accuracy | 92.82% |
| Automated Attack precision | 1.0000 |
| Automated Attack recall | 0.8783 |
| Automated Attack F1 | 0.9352 |

![Risk-Aware Routing](results/figures/locked_test_risk_aware_routing.png)

## Attack Attribution

Stage-2 DoS-vs-Replay validation:

- Accuracy = **0.5263**
- Balanced Accuracy = **0.5369**
- Macro F1 = **0.5262**
- ROC-AUC = **0.5537**

This result is insufficient for reliable autonomous attack-specific
response.

## Physical Branch

Sensitivity analysis:

| configuration                   |   features |   accuracy |   balanced_accuracy |   macro_f1 |   log_loss |
|:--------------------------------|-----------:|-----------:|--------------------:|-----------:|-----------:|
| all_features                    |         98 |   1        |            1        |   1        | 0.00864497 |
| remove_all_barometer            |         91 |   0.496183 |            0.295767 |   0.303182 | 1.50331    |
| remove_barometer_absolute_level |         94 |   0.480916 |            0.279453 |   0.280255 | 1.51415    |
| all_sources_dynamic_only        |         42 |   0.549618 |            0.36164  |   0.358044 | 1.25755    |

The apparent perfect physical validation result was dominated by
barometer baseline information.

![Physical Confounding](results/figures/physical_barometer_confounding.png)

The physical branch is reported as a confounding case study.
