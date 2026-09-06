Methodology
##1. Objective

The objective is to evaluate UAV cyber-physical attack detection
while controlling for schema leakage, temporal overlap, probability
miscalibration, and uncertain predictions.

##2. Raw dataset reconstruction

The raw CSV contains:

54,774 data rows
10 embedded section headers
54,784 total physical lines

The ten reconstructed sections are shown below.

| Section            | Class     | Modality   | Schema            |   Rows |   Predictors |
|:-------------------|:----------|:-----------|:------------------|-------:|-------------:|
| cyber_benign       | Benign    | cyber      | cyber_schema_A    |   9425 |           37 |
| physical_benign    | Benign    | physical   | physical_schema_A |   4290 |           16 |
| cyber_dos          | DoS       | cyber      | cyber_schema_A    |  11671 |           37 |
| physical_dos       | DoS       | physical   | physical_schema_A |    973 |           16 |
| cyber_replay       | Replay    | cyber      | cyber_schema_A    |  12006 |           37 |
| physical_replay    | Replay    | physical   | physical_schema_A |    973 |           16 |
| cyber_evil_twin    | Evil Twin | cyber      | cyber_schema_B    |   5683 |           34 |
| physical_evil_twin | Evil Twin | physical   | physical_schema_B |   5473 |           21 |
| cyber_fdi          | FDI       | cyber      | cyber_schema_B    |   3473 |           34 |
| physical_fdi       | FDI       | physical   | physical_schema_C |    807 |           31 |

Different classes do not always use the same schema.

A single naïve five-class table was therefore rejected.

##3. Schema-aware experiments

The primary comparable Schema-A classes are:

Benign
DoS
Replay

Separate Cyber and Physical Schema-A experiments were created.

Evil Twin and FDI were retained only in secondary within-schema
analyses.

##4. Feature governance

Likely identity and collection variables were excluded before model
development.

Cyber exclusions included variables such as:

timestamps,
frame identifiers,
MAC/IP endpoint identity,
sequence identifiers,
raw payload-like fields.

Timestamps were retained only for segmentation.

##5. Temporal windowing

Cyber Schema A:

Window size = 20 observations
Gap threshold = 16.873730 seconds
Total windows = 1632
Exact duplicate windows = 0
Conflicting window groups = 0

Physical Schema A:

Window size = 10 observations
Gap threshold = 2.780835 seconds
Total windows = 605

Windows never cross timestamp resets or long temporal gaps.

##6. Frozen temporal split

Entire temporal segments were assigned to only one partition.

| Partition   |   Windows |
|:------------|----------:|
| train       |       960 |
| validation  |       317 |
| test        |       355 |

The split was chronological within each class.

The test partition remained locked during:

model selection,
feature selection,
calibration fitting,
routing threshold selection.
##7. Baseline modeling

Development baselines:

Logistic Regression
Random Forest
XGBoost

Macro F1 was the primary multiclass development metric.

##8. Physical confounding audit

Physical XGBoost initially achieved a validation Macro F1 of 1.0000.

Barometer validation ranges showed strong separation:

| Class   |   Minimum |   Maximum |   Median |
|:--------|----------:|----------:|---------:|
| Benign  |   21157   |   21337.9 |  21301.6 |
| DoS     |   14174.4 |   14394.4 |  14184.6 |
| Replay  |   19088   |   19147.1 |  19118.7 |

Sensitivity results:

| configuration                   |   features |   accuracy |   balanced_accuracy |   macro_f1 |   log_loss |
|:--------------------------------|-----------:|-----------:|--------------------:|-----------:|-----------:|
| all_features                    |         98 |   1        |            1        |   1        | 0.00864497 |
| remove_all_barometer            |         91 |   0.496183 |            0.295767 |   0.303182 | 1.50331    |
| remove_barometer_absolute_level |         94 |   0.480916 |            0.279453 |   0.280255 | 1.51415    |
| all_sources_dynamic_only        |         42 |   0.549618 |            0.36164  |   0.358044 | 1.25755    |

The perfect physical result was therefore not promoted as
general attack-detection evidence.

##9. Cyber feature reduction

Full temporal cyber representation:

161 aggregated features

Predefined selection rule:

Select the smallest tested source-family subset retaining at least
99% of full-model validation Macro F1.

Selected source families:

time_since_last_packet
wlan.fc.type

Selected aggregated features:

14

Retention:

99.40%

##10. Hierarchical formulation

Stage 1:

Benign vs Attack

Stage 2:

DoS vs Replay

Stage-1 validation Attack F1:

0.9710

Stage-2 validation Macro F1:

0.5262

The weak Stage-2 result prevented attack-specific autonomous response.

##11. Calibration

Temperature scaling was fitted using grouped OOF probabilities from
the frozen Train set.

Temperature:

1.71111364

The calibration mapping was adopted because validation log loss
improved according to the predefined rule.

##12. Risk-aware routing

Uncertainty:

1 - max(P(Benign), P(Attack))

The validation threshold was selected to capture at least 75% of
prediction errors while minimizing verification burden.

Frozen uncertainty threshold:

0.114250

##13. Locked Test policy

Before test evaluation, the model and all decision artifacts were frozen.

The test was then consumed once.

No post-test tuning is permitted.
