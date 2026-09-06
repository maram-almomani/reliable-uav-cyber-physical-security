# Reliable UAV Cyber-Physical Security

**Leakage-aware UAV attack detection, probability calibration, and risk-aware routing under temporal distribution shift.**

## Overview

This project investigates machine-learning-based cybersecurity for UAV cyber-physical systems, with emphasis on **reliability, leakage control, temporal generalization, and uncertainty-aware decision making** rather than headline classification accuracy alone.

The workflow includes:

- multi-schema dataset auditing;
- schema-aware parsing;
- leakage-oriented feature governance;
- temporal segmentation and window construction;
- chronological group-held-out evaluation;
- baseline model comparison;
- source-family feature reduction;
- probability calibration;
- uncertainty-aware routing;
- attack-attribution analysis;
- and explicit confounding diagnostics.

A central objective of the study is to determine whether strong development performance remains reliable when the model is evaluated on temporally separated observations that were not used during model, feature, calibration, or policy selection.

---

## Threat Model

The study considers a UAV cyber-physical environment in which an adversary may manipulate, disrupt, or replay communication-related activity represented in the available dataset.

### Attacker capabilities

For the primary detection task, the adversary is assumed to be capable of conducting attacks represented by the common Cyber Schema-A data:

- **Denial-of-Service (DoS):** disruption of normal communication behavior.
- **Replay:** retransmission of previously observed communication or control-related traffic.

The source dataset also contains **Evil Twin** and **False Data Injection (FDI)** scenarios. However, these observations use different feature schemas and are therefore not merged into the primary Benign-vs-Attack detector.

### Attack surface

The experimental attack surface includes:

- UAV wireless and network communication behavior;
- packet timing and protocol characteristics;
- communication-session observations available to a monitoring component;
- and physical telemetry examined separately for diagnostic and confounding analysis.

### System assumptions

The experimental framework assumes that:

- the monitoring system can observe the required cyber measurements;
- packet ordering and timing information are available for temporal segmentation;
- observations can be grouped into short temporal windows;
- the detector operates on attack categories represented in the available dataset;
- and uncertain predictions may be routed for additional verification instead of forcing an autonomous decision.

### Out of scope

The study does not claim to evaluate:

- arbitrary zero-day attacks;
- physical destruction or direct hardware tampering;
- malware execution within UAV firmware;
- cryptographic key compromise;
- GPS spoofing unless explicitly represented in the source data;
- autonomous recovery or flight-control intervention on a physical UAV;
- or guaranteed transfer to unseen UAV platforms, radios, and communication stacks.

The threat model is therefore **dataset-bounded** and should not be interpreted as a complete UAV security model.

---

## Dataset Structure

The original CSV is **not one homogeneous machine-learning table**.

Structural auditing showed that it contains ten cyber and physical sections with different schemas.

| Section | Class | Modality | Schema | Rows | Predictors |
|:---|:---|:---|:---|---:|---:|
| cyber_benign | Benign | cyber | cyber_schema_A | 9,425 | 37 |
| physical_benign | Benign | physical | physical_schema_A | 4,290 | 16 |
| cyber_dos | DoS | cyber | cyber_schema_A | 11,671 | 37 |
| physical_dos | DoS | physical | physical_schema_A | 973 | 16 |
| cyber_replay | Replay | cyber | cyber_schema_A | 12,006 | 37 |
| physical_replay | Replay | physical | physical_schema_A | 973 | 16 |
| cyber_evil_twin | Evil Twin | cyber | cyber_schema_B | 5,683 | 34 |
| physical_evil_twin | Evil Twin | physical | physical_schema_B | 5,473 | 21 |
| cyber_fdi | FDI | cyber | cyber_schema_B | 3,473 | 34 |
| physical_fdi | FDI | physical | physical_schema_C | 807 | 31 |

A naïve global classifier could therefore learn **schema identity** instead of attack behavior.

To reduce this risk, the primary comparable experiment is restricted to classes sharing **Cyber Schema A**:

- Benign
- DoS
- Replay

Evil Twin and FDI are retained only in secondary within-schema analyses where appropriate.

---

## Primary Cyber Task

The final Stage-1 security task is:

**Benign vs Attack**

where:

```text
Attack = DoS OR Replay
```

The detector operates on fixed windows of **20 cyber observations**.

Temporal segments are separated when:

- timestamps reset; or
- a sufficiently large temporal gap is observed.

Windows are never allowed to cross those temporal boundaries.

This preserves short-term communication context while reducing the risk of creating artificial sequences across unrelated recording periods.

---

## Frozen Temporal Split

The primary experiment uses a **chronological segment-grouped split**.

| Partition | Windows |
|:---|---:|
| Train | 960 |
| Validation | 317 |
| Locked Test | 355 |

Entire temporal segments are assigned to only one partition.

No temporal segment appears in more than one of:

```text
Train
Validation
Locked Test
```

The Locked Test remained inaccessible during:

- baseline model comparison;
- feature selection;
- calibration fitting;
- uncertainty-threshold selection;
- and routing-policy design.

It was evaluated only after all development decisions had been frozen.

---

## Final Feature Set

The full temporal cyber representation contained:

**161 aggregated features**

A source-family reduction procedure was used to identify the smallest predefined feature-family subset retaining at least **99% of the full-model validation Macro F1**.

The final lightweight detector uses only two source feature families:

- `time_since_last_packet`
- `wlan.fc.type`

Final aggregated feature count:

**14**

Validation Macro-F1 retention relative to the full model:

**99.40%**

![Feature reduction](results/figures/cyber_feature_reduction.png)

This reduction provides a substantially smaller input representation while preserving nearly all development-set predictive performance.

---

## Stage-1 Attack Detection Results

| Metric | Validation | Locked Test |
|---|---:|---:|
| Accuracy | 0.9590 | 0.6958 |
| Balanced Accuracy | 0.9612 | 0.7915 |
| Attack Precision | 0.9864 | 1.0000 |
| Attack Recall | 0.9561 | 0.5830 |
| Attack F1 | 0.9710 | 0.7366 |
| ROC-AUC | 0.9886 | 0.9369 |
| PR-AUC | 0.9961 | 0.9777 |

![Validation vs Locked Test](results/figures/validation_vs_locked_test.png)

The locked temporal test exposed a substantial **operating-point generalization gap**.

At the frozen decision threshold of `0.5`:

- all **96 benign windows** were classified correctly;
- **151 attack windows** were detected;
- **108 attack windows** were missed;
- and no benign window was incorrectly classified as an attack.

![Locked Test Confusion Matrix](results/figures/locked_test_confusion_matrix.png)

Although threshold-level Attack Recall decreased substantially, ranking performance remained considerably stronger:

- ROC-AUC = **0.9369**
- PR-AUC = **0.9777**

This distinction is important. The Locked Test does not support describing the frozen detector as a high-recall general UAV attack detector. Instead, it shows that **ranking ability remained strong while the fixed operating threshold generalized poorly across later temporal segments**.

---

## Attack Attribution

A secondary model was evaluated for:

**DoS vs Replay**

Validation results:

- Accuracy = **0.5263**
- Balanced Accuracy = **0.5369**
- Macro F1 = **0.5262**
- ROC-AUC = **0.5537**

This level of discrimination is not sufficiently reliable to support autonomous attack-specific responses.

The final operational formulation therefore separates:

```text
Reliable-enough primary detection:
Benign vs Attack

Weak exploratory attribution:
DoS vs Replay
```

Attack attribution is not used to trigger attack-specific autonomous actions.

---

## Probability Calibration

Temperature scaling was fitted using **grouped out-of-fold predictions from the frozen training partition only**.

Frozen temperature:

**T = 1.71111364**

Locked-Test probability-quality results:

| Metric | Raw | Temperature Scaled |
|---|---:|---:|
| Log Loss | 0.7889 | 0.5741 |
| Brier Score | 0.2334 | 0.1978 |
| Equal-width ECE | 0.2173 | 0.1628 |
| Adaptive ECE | 0.2175 | 0.1662 |

Temperature scaling changed **zero class predictions**, as expected from positive-temperature binary scaling at the unchanged `0.5` operating threshold.

However, it improved all reported probability-quality metrics on the Locked Test.

![Calibration](results/figures/locked_test_calibration_generalization.png)

The result indicates that calibration improved the quality of predictive confidence under the locked temporal distribution even though it could not correct classification errors produced by the frozen decision threshold.

---

## Risk-Aware Routing

Rather than automatically acting on every Stage-1 prediction, the framework introduces an uncertainty-aware routing policy.

Uncertainty is defined as:

```text
uncertainty = 1 - max(P(Benign), P(Attack))
```

The routing threshold was selected on validation before Locked-Test access.

Frozen uncertainty threshold:

**0.114250**

Equivalent minimum confidence for automated handling:

**0.885750**

The resulting policy is:

| Model state | Operational route |
|:---|:---|
| High-confidence Benign | Continue monitoring |
| High-confidence Attack | Generic protective response |
| Low confidence | Additional verification required |

Locked-Test results:

- Verification fraction = **45.07%**
- Error capture = **87.04%**
- Automated fraction = **54.93%**
- Automated accuracy = **92.82%**
- Automated Attack precision = **1.0000**
- Automated Attack recall = **0.8783**
- Automated Attack F1 = **0.9352**

![Risk-aware Routing](results/figures/locked_test_risk_aware_routing.png)

The frozen routing policy captured **94 of 108 Stage-1 errors**, corresponding to an error-capture rate of **87.04%**.

This result should not be interpreted as eliminating model risk. Instead, it demonstrates how calibrated confidence can be used to **reduce the number of low-confidence errors that remain in the automated decision stream**.

The verification route is a research-policy simulation; no human analyst or live autonomous-response experiment was performed.

---

## Physical-Domain Finding

The Physical Schema-A XGBoost model initially achieved apparently perfect validation classification.

Further diagnostics showed that this result was dominated by **absolute barometer level**.

Removing all barometer-derived information reduced validation Macro F1 to:

**0.3032**

Removing only the absolute barometer-level statistics reduced Macro F1 to:

**0.2803**

![Physical Confounding](results/figures/physical_barometer_confounding.png)

The physical branch is therefore reported as a **dataset-confounding case study**, not as evidence of perfect physical-domain UAV attack detection.

This analysis does not prove that barometric behavior is unrelated to attacks. It shows that the available dataset does not adequately disentangle attack effects from recording-session or environmental baseline differences.

---

## Experimental Integrity

Several safeguards were incorporated to reduce optimistic or leakage-driven evaluation.

### Schema leakage control

The raw CSV was reconstructed into ten separate cyber and physical sections before modeling.

A global five-class classifier was deliberately rejected because several attack classes are associated with different feature schemas. Such a classifier could exploit schema identity as a shortcut for the target label.

### Identifier and metadata exclusion

Direct identifiers and collection-related variables that could provide shortcut information were excluded from model inputs where appropriate.

These included variables such as:

- timestamps;
- frame identifiers;
- MAC/IP endpoint identities;
- sequence-like identifiers;
- and raw payload-related fields.

Timestamps were retained only for temporal segmentation and provenance.

### Temporal leakage prevention

Fixed observation windows were generated **within temporal segments only**.

Windows were not allowed to cross:

- timestamp resets;
- or large temporal gaps.

Entire temporal segments were then assigned to one partition, preventing temporal-group overlap across Train, Validation, and Locked Test.

### Feature-selection discipline

Feature reduction was performed using development data only.

The final selected cyber representation was frozen before Locked-Test access.

### Calibration integrity

Temperature scaling was fitted using grouped out-of-fold predictions from the training partition.

The Locked Test was never used to estimate the temperature parameter.

### Routing-policy integrity

The uncertainty threshold was selected using validation predictions only.

The routing policy was frozen before test evaluation.

### Pre-test freeze

Before the Locked Test was opened, hashes of the key frozen artifacts were stored in a pre-test freeze manifest.

These artifacts included:

- the Stage-1 model;
- selected feature configuration;
- calibration configuration;
- routing policy;
- and split configuration.

### Locked-test discipline

The Locked Test was consumed once after the complete development pipeline had been frozen.

No model, feature set, calibration parameter, operating threshold, or routing policy was modified after observing Locked-Test performance.

### Confounding checks

The apparently perfect physical-model result triggered additional feature-importance and ablation analysis.

The analysis identified barometer baseline information as a dominant predictive confound.

Rather than reporting the perfect validation score as evidence of general attack detection, the physical branch was retained as a documented confounding case study.

---

## Main Methodological Contributions

1. Reconstruction of a heterogeneous multi-schema UAV dataset rather than naïve CSV ingestion.
2. Schema-aware experimental design to reduce label/schema leakage.
3. Leakage-resistant temporal segmentation and chronological group-held-out evaluation.
4. Reduction from **161 to 14 aggregated cyber features** while retaining **99.40%** of full-model development Macro F1.
5. Separation of primary attack detection from weak attack attribution.
6. Grouped out-of-fold probability calibration.
7. A validation-frozen uncertainty-aware routing policy.
8. One-time Locked-Test evaluation after pre-test artifact freezing.
9. Explicit reporting of temporal generalization degradation.
10. Identification and documentation of physical sensor/session confounding rather than reporting a misleading perfect score.

---

## Reproducibility / How to Run

The repository preserves the notebook, experiment configurations, audit artifacts, aggregate evaluation results, and final figures required to inspect and reproduce the research workflow.

### 1. Clone the repository

```bash
git clone https://github.com/maram-almomani/reliable-uav-cyber-physical-security.git
cd reliable-uav-cyber-physical-security
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Add the source dataset

The raw dataset is intentionally excluded from Git version control.

Place:

```text
Dataset_T-ITS.csv
```

under:

```text
data/raw/Dataset_T-ITS.csv
```

### 4. Open the main notebook

The primary research notebook is:

```text
notebooks/Reliable_UAV_Cyber_Physical_Security.ipynb
```

The notebook contains the experimental workflow for:

1. raw dataset structural auditing;
2. section-aware parsing;
3. schema analysis;
4. feature governance;
5. temporal segmentation;
6. fixed-observation window construction;
7. chronological group-aware splitting;
8. baseline model comparison;
9. cyber feature reduction;
10. physical confounding analysis;
11. hierarchical detection experiments;
12. grouped out-of-fold temperature calibration;
13. validation-only uncertainty-policy selection;
14. and final locked-test evaluation.

### 5. Google Colab path configuration

The project was developed in Google Colab with Google Drive storage.

Users reproducing the workflow locally or under a different Drive structure must update the project path configuration in the notebook before execution.

### 6. Outputs

Aggregate experimental tables are stored under:

```text
results/tables/
```

Figures are stored under:

```text
results/figures/
```

Reproducibility and frozen experiment configurations are stored under:

```text
data/processed/
```

Large intermediate datasets, fitted models, row-level prediction files, and the raw source dataset are intentionally excluded from the public repository.

---

## Repository Structure

```text
reliable-uav-cyber-physical-security/
├── data/
│   ├── README.md
│   ├── raw/
│   └── processed/
├── models/
├── notebooks/
│   └── Reliable_UAV_Cyber_Physical_Security.ipynb
├── results/
│   ├── figures/
│   └── tables/
├── README.md
├── METHODOLOGY.md
├── RESULTS.md
├── LIMITATIONS.md
├── MODEL_CARD.md
├── requirements.txt
└── .gitignore
```

Additional details are available in:

- [`METHODOLOGY.md`](METHODOLOGY.md)
- [`RESULTS.md`](RESULTS.md)
- [`LIMITATIONS.md`](LIMITATIONS.md)
- [`MODEL_CARD.md`](MODEL_CARD.md)

---

## Limitations

The results should be interpreted within several important constraints.

- The study is based on a single UAV cyber-physical dataset.
- The primary detector evaluates Benign, DoS, and Replay observations sharing a common cyber schema.
- Evil Twin and FDI are not included in the primary detector because their available representations use different schemas.
- The number of independent temporal recording segments is substantially smaller than the number of generated windows.
- The Locked Test revealed a substantial temporal generalization gap, particularly in Attack Recall at the frozen `0.5` operating threshold.
- Results cannot be assumed to transfer directly to unseen UAV platforms, wireless hardware, environments, or communication stacks.
- The physical-domain experiment showed strong barometer/session confounding and is not presented as evidence of general physical attack detection.
- The DoS-vs-Replay attribution model was too weak to justify attack-specific autonomous actions.
- The uncertainty-routing mechanism represents a simulated decision policy rather than a validated human-in-the-loop system.
- No protective response was executed on a real UAV.
- The study does not claim coverage of arbitrary zero-day or previously unseen attack categories.
- No post-test tuning is permitted using the already consumed Locked Test.

A more detailed discussion is available in [`LIMITATIONS.md`](LIMITATIONS.md).

---

## Intended Use

This repository is intended for:

- UAV cybersecurity research;
- machine learning for cyber-physical security;
- intrusion-detection methodology;
- temporal generalization analysis;
- probability calibration research;
- uncertainty-aware security decision making;
- and reproducibility-oriented academic demonstration.

This repository is **not** a certified UAV safety system, production intrusion-detection product, or autonomous flight-control response mechanism.

---

## Author

**Maram M. Momani**

Cybersecurity Researcher  
**Machine Learning for Cybersecurity | Network & IoT Security | Cyber-Physical Systems Security**

GitHub: [maram-almomani](https://github.com/maram-almomani)
